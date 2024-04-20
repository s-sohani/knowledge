You want your deployments to stay up and running automatically and remain healthy without any manual intervention. To do this, you almost never create pods directly. Instead, you create other types of resources, such as ReplicationControllers or Deployments.
Kubernetes checks if a container is still alive and restarts it if it isn’t.

When Kubernetes restart the container?
- If application has a bug that causes it to crash every once in a while.
- Stop working without their process crashing like memory leak. 
- Check an application’s health from the outside.

## Linveness Probe
Kubernetes can check if a container is still alive through liveness probes.
Kubernetes can probe a container using one of the three mechanisms:
- An HTTP GET probe: the HTTP response code is 2xx or 3xx.
- A TCP Socket probe: tries to open a TCP connection to the specified port of the container.
- An Exec probe: executes an arbitrary command inside the container, If the status code is 0, the probe is successful.

### Creating an HTTP-based Liveness probe
```
apiVersion: v1
kind: pod
metadata:
	name: kubia-liveness
spec:
	containers:
		- image: luksa/kubia-unhealthy
		  name: kubia
		  livenessProbe:
		    httpGet:
		      path: /
		      port: 8080
		    initialDelaySeconds: 15
```

If you don’t set the initial delay, the prober will start probing the container as soon as
it starts.
You can also set additional properties, such as delay, timeout, period.

## Replication Controller
A ReplicationController is a Kubernetes resource that ensures its pods are always
kept running. If the pod disappears for any reason, such as in the event of a node
disappearing from the cluster or because the pod was evicted from the node, the
ReplicationController notices the missing pod and creates a replacement pod and makes sure the actual number of pods of a “type” always matches the desired number.
ReplicationControllers don’t operate on pod types, but on sets of pods that match a certain label selector.

![[Screenshot from 2024-04-15 07-30-35.png]]

A ReplicationController has three essential parts:
- A label selector, which determines what pods are in the ReplicationController’s scope
- A replica count, which specifies the desired number of pods that should be running
- A pod template, which is used when creating new pod replicas
Replication Controllers don’t operate on pod types, but on sets of pods that match a certain label selector. A ReplicationController’s job is to make sure that an exact number of pods always matches its label selector.
Changing the label selector makes the existing pods fall out of the scope of the ReplicationController, so the controller stops caring about them.
## Creating a ReplicationController
When you post the file to the API server, Kubernetes creates a new ReplicationController named kubia , which makes sure three pod instances always match the **label selector** app=kubia (**Label selector** different from **Node selector** in prev section).
```
apiVersion: v1
kind: ReplicationController
metadata:
	name: kubia
spec:
	replicas: 3
	selector:
		app: kubia #The pod selector determining what pods the RC is operating on
template:
	metadata:
		labels:
			app: kubia
	spec:
		containers:
		- name: kubia
		  image: luksa/kubia
		  ports:
		  - containerPort: 8080
```

The pod labels in the template must obviously match the label selector of the ReplicationController

To create ReplicationController:
```
kubectl create -f kubia-rc.yaml
```

Get ReplicationController
```
kubectl get rc
```

Get more information aboad an RC
```
kubectl describe rc kubia
```

If you change a pod’s labels so they no longer match a ReplicationController’s label selector, the pod becomes like any other manually created pod. It’s no longer managed by anything. If the node running the pod fails, the pod is obviously not rescheduled. But keep in mind that when you changed the pod’s labels, the replication controller noticed one pod was missing and spun up a new pod to replace it.

### Edit Exist ReplicationController
```
kubectl edit rc kubia
```

### Horizontally scaling pods
```
# Scale with command
kubectl scale rc kubia --replicas=10
# Scale with edit rc file
kubectl edit rc kubia
```

### Deleting a ReplicationController
When you delete a ReplicationController through kubectl delete , the pods are also deleted. When deleting a ReplicationController with kubectl delete , you can keep its pods running by passing the --cascade=false option to the command.
```
kubectl delete rc kubia --cascade=false
```

## ReplicaSets
It’s a new generation of ReplicationController and replaces it completely (ReplicationControllers will eventually be deprecated).
has more expressive pod selectors:
- A ReplicaSet’s selector also allows matching pods that lack a certain label or pods that include a certain label key, regardless of its value.
- ReplicaSets can match pods based merely on the presence of a label key.
```
apiVersion: apps/v1beta2
kind: ReplicaSet
metadata:
	name: kubia
spec:
	replicas: 3
	selector:
		matchLabels:
			app: kubia
	template:
		metadata:
			labels:
				app: kubia
		spec:
			containers:
			- name: kubia
			  image: luksa/kubia
```
### Use Expressions Label Selector
```
selector:
	matchExpressions:
		- key: app
		operator: In
		values:
		  - kubia
```

### Additional Selectors
- In: Label’s value must match one of the specified values.
- NotIn: Label’s value must not match any of the specified values.
- Exists: Pod must include a label with the specified key (the value isn’t important). When using this operator, you shouldn’t specify the values field.
- DoesNotExist: Pod must not include a label with the specified key. The values property must not be specified.

### Print ReplicaSet
```
kubectl get rs
kubectl describe rs
```


## DaemonSets
Running exactly one pod on each node. Pod to run on each and every node in the cluster.
- A DaemonSet doesn’t have any notion of a desired replica count. It doesn’t need it because its job is to ensure that a pod matching its pod selector is running on each node.
- You’ve already used node selectors to deploy a pod onto specific nodes.
Example
```
kubectl get node
kubectl label node minikube disk=ssd

```

```
apiVersion: apps/v1beta2
kind: DaemonSet
metadata:
	name: ssd-monitor
spec:
	selector:
		matchLabels:
			app: ssd-monitor
	template:
		metadata:
			labels:
				app: ssd-monitor
		spec:
			nodeSelector:
				disk: ssd
			containers:
			- name: main
			  image: luksa/ssd-monitor
```

DaemonSets Commands
```
kubectl create -f ssd-monitor-daemonset.yaml
kubectl get ds
kubectl get po
```

##  JOB - Single Completable Task
Run a task that terminates after completing its work.
In the event of a failure of the process itself (when the process returns an error exit code), the Job can be configured to either restart the container or not.

```
apiVersion: batch/v1
kind: Job
metadata:
	name: batch-job
spec:
	template:
		metadata:
			labels:
				app: batch-job
		spec:
			restartPolicy: OnFailure
			containers:
			- name: main
			  image: luksa/batch-job
```