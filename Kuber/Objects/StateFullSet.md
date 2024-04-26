You now know how to:
- Run single-instance POD
- Replicated stateless PODs
- Stateful PODs utilizing persistent storage
- Run a single database POD instance throgh PVC
But can you employ a ReplicaSet to replicate the database pod?

## Replicating stateful pods
ReplicaSets create multiple pod replicas from a single pod template.
If the pod template includes a volume, which refers to a specific PersistentVolumeClaim, all replicas of the ReplicaSet will use the exact same PersistentVolumeClaim and therefore the same PersistentVolume bound by the claim.
![[Pasted image 20240426152950.png]]

>you can’t make each replica use its own separate PersistentVolumeClaim. You can’t use a ReplicaSet to run a distributed data store, where each instance needs its own separate storage.

### Running multiple replicas with separate storage for each
#### USING ONE R EPLICA S ET PER POD INSTANCE
You could create multiple ReplicaSets.
![[Pasted image 20240426153627.png]]

>BUT if you’d scale the pods in that case. You couldn’t change the desired replica count—you’d have to create additional ReplicaSets instead.

#### USING MULTIPLE DIRECTORIES IN THE SAME VOLUME
A trick you can use is to have all pods use the same PersistentVolume, but then have a separate file directory inside that volume for each pod.

![[Pasted image 20240426153857.png]]

>Because you can’t configure pod replicas differently from a single pod template, you can’t tell each instance what directory it should use, but you can make each instance tomatically select (and possibly also create) a data directory that isn’t being used by any other instance at that time. This solution does require coordination between the instances, and isn’t easy to do correctly.


### Providing a stable identity for each pod
When a ReplicaSet replaces a pod, the new pod is a completely new pod with a new hostname and IP. Starting up with the old instance’s data but with a completely new network identity may cause problems. But Kubernetes, every time a pod is rescheduled, the new pod gets both a new hostname and a new IP address, so the whole application cluster would have to be reconfigured every time one of its members is rescheduled.
#### USING A DEDICATED SERVICE FOR EACH POD INSTANCE
A trick you can use to work around this problem is to provide a stable network address for cluster members by creating a dedicated Kubernetes Service for each individual member.

![[Pasted image 20240426160722.png]]

>The solution is not only ugly, but it still doesn’t solve everything.

Luckily, Kubernetes saves us from resorting to such complex solutions.

## Understanding StatefulSets
Instead of using a ReplicaSet to run these types of pods, you create a StatefulSet resource.

### COMPARING STATEFULSETS WITH REPLICASETS OR REPLICATION CONTROLLERS
A StatefulSet makes sure pods are rescheduled in such a way that they retain their identity and state. New instance needs to get the same name, network identity, and state as the one it’s replacing. Pods created by the StatefulSet aren’t exact replicas of each other. Each can have its own set of volumes—in other words, storage (and thus persistent state) which differentiates it from its peers.

### Providing a stable network identity
Each pod created by a StatefulSet is assigned an ordinal index (zero-based), which is then used to derive the pod’s name and hostname, and to attach stable storage to the pod.

![[Pasted image 20240426163544.png]]

#### INTRODUCING THE GOVERNING SERVICE
Stateful pods sometimes need to be addressable by their hostname.
For example, if the governing Service belongs to the default namespace and is called foo, and one of the pods is called A-0 , you can reach the pod through its fully qualified domain name, which is a-0.foo.default.svc.cluster.local . You can’t do that with pods managed by a
ReplicaSet.
Additionally, you can also use DNS to look up all the StatefulSet’s pods’ names by looking up SRV records for the foo.default.svc.cluster.local domain.

#### REPLACING LOST PETS
When a pod instance managed by a StatefulSet disappears, the replacement pod gets the same name and hostname as the pod that has disappeared.

![[Pasted image 20240426181507.png]]

#### SCALING A STATEFULSET
Scaling the StatefulSet creates a new pod instance with the next unused ordinal index. Scaling down a StatefulSet always removes the instances with the highest ordinal index first.
![[Pasted image 20240426182223.png]]

>Because certain stateful applications don’t handle rapid scale-downs nicely, StatefulSets scale down only one pod instance at a time.

### Providing stable dedicated storage to each stateful instance
Each pod of a StatefulSet needs to reference a different PersistentVolumeClaim to have its own separate PersistentVolume.

![[Pasted image 20240426182958.png]]

#### UNDERSTANDING THE CREATION AND DELETION OF PERSISTENTVOLUMECLAIMS
Scaling down, however, deletes only the pod, leaving the claims alone.
After a claim is deleted, the PersistentVolume it was bound to gets recycled or deleted and its contents are lost.
Because stateful pods are meant to run stateful applications, which implies that the data they store in the volume is **important**.
For this reason, you’re required to delete PersistentVolumeClaims **manually** to release the underlying PersistentVolume.


#### REATTACHING THE PERSISTENT VOLUMECLAIM TO THE NEW INSTANCE OF THE SAME POD
The fact that the PersistentVolumeClaim remains after a scale-down means a subsequent scale-up can reattach the same claim along with the bound PersistentVolume and its contents to the new pod instance.

![[Pasted image 20240426184257.png]]


## Using a StatefulSet
Now building clustered data store.
### Create Three PersistentVolumes
```
kind: List
apiVersion: v1
items:
	- apiVersion: v1
	  kind: PersistentVolume
	  metadata:
		name: pv-a
	  spec:
		capacity:
			storage: 1Mi
		accessModes:
		- ReadWriteOnce
		persistentVolumeReclaimPolicy: Recycle
		gcePersistentDisk:
			pdName: pv-a
			fsType: nfs4
	- apiVersion: v1
	  kind: PersistentVolume
	  metadata:
			name: pv-b
```

### CREATING THE GOVERNING SERVICE
```
apiVersion: v1
kind: Service
metadata:
	name: kubia
spec:
	clusterIP: None
	selector:
		app: kubia
	ports:
	- name: http
	  port: 80
```

### CREATING THE STATEFULSET MANIFEST
```
apiVersion: apps/v1beta1
kind: StatefulSet
metadata:
	name: kubia
spec:
	serviceName: kubia
	replicas: 2
	template:
		metadata:
			labels:
				app: kubia
		spec:
			containers:
			- name: kubia
			  image: luksa/kubia-pet
			  ports:
				- name: http
				containerPort: 8080
			volumeMounts:
			- name: data
			  mountPath: /var/data
	volumeClaimTemplates:
	- metadata:
		name: data
	  spec:
		resources:
		  requests:
			storage: 1Mi
		accessModes:
		- ReadWriteOnce
```

What’s new is the volumeClaimTemplates list. In it, you’re defining one volume claim template called data , which will be used to create a PersistentVolumeClaim for each pod.