Update your app with zero-downtime update process.

## Updating applications running in pods
A normal service looks like 
![[Pasted image 20240502185509.png]]

When we create new version of application and push it to docker repository, how can we update the pod version?

We can do that with two ways:
- Delete all existing pods first and then start the new ones.
- Start new ones and, once they’re up, delete the old ones. You can do this either by adding all the new pods and then deleting all the old ones at once, or sequentially, by adding new pods and removing old ones gradually.

### Deleting old pods and replacing them with new ones
If you have a ReplicationController managing a set of v1 pods, you can easily replace them by modifying the pod template so it refers to version v2 of the image and then deleting the old pod instances. The ReplicationController will notice that no pods match its label selector and it will spin up new instances.

![[Pasted image 20240502190402.png]]

>You should accept the short downtime between the time the old pods are deleted and new ones are started.

### Spinning up new pods and then deleting the old ones
>This prcess will require more hardware resources, because you’ll have double the number of pods running at the same time for a short while.

#### SWITCHING FROM THE OLD TO THE NEW VERSION AT ONCE
You can change the Service’s label selector and have the Service switch over to the new pods.

![[Pasted image 20240502191007.png]]

#### PERFORMING A ROLLING UPDATE
Instead of bringing up all the new pods and deleting the old pods at once, you can also perform a rolling update, which replaces pods step by step. In this case, you’ll want the Service’s pod selector to include both the old and the new pods.

![[Pasted image 20240502191724.png]]

## Performing an automatic rolling update with a ReplicationController
All you need to do is tell it which ReplicationController you’re replacing, give a name for the new ReplicationController, and specify the new image you’d like to replace the original one with.
```
kubectl rolling-update kubia-v1 kubia-v2 --image=luksa/kubia:v2
```
kubectl created this ReplicationController by copying the kubia-v1 controller and changing the image in its pod template.

![[Pasted image 20240502193650.png]]

If you look closely at the controller’s label selector, It includes not only a simple app=kubia label, but also an additional deployment label, This is necessary to avoid having both the new and the old ReplicationControllers operating on the same set of pods. 
The rolling-update process has modified the selector of the first ReplicationController, as well, kubectl had also modified the labels of the live pods just before modifying the ReplicationController’s selector.

![[Pasted image 20240502194727.png]]

In progress :
- Scaling kubia-v2 up to 1
- Scaling kubia-v1 down to 2

![[Pasted image 20240502194945.png]]

- Scaling kubia-v2 up to 2
- Scaling kubia-v1 down to 1
.
.
.
- Scaling kubia-v2 up to 3
- Scaling kubia-v1 down to 0
.
.
.
- Update succeeded. Deleting kubia-v1
- Replicationcontroller "kubia-v1" rolling updated to "kubia-v2"

### Drawback of this scenario
Kubernetes modifying the labels of my pods and the label selectors of my ReplicationController s is something that I don’t expect and could cause me to go around the office yelling at my colleagues, “Who’s been messing with my controllers!?!?”

These requests are the ones scaling down your ReplicationController, which shows that the **kubectl** client is the one doing the scaling, instead of it being **performed by the Kubernetes master**.
What if you lost network connectivity while kubectl was performing the update.

## Using Deployments for updating apps declaratively
RC and RS are low-level but Deployment is Higher-level for deploy and update application decleratively, for example When you create a Deployment, a ReplicaSet resource is created and pods are created and managed by the Deployment’s ReplicaSets.
Using a Deployment instead of the lower-level constructs makes updating an app much easier, because you’re defining the desired state through the single Deployment resource and letting Kubernetes take care of the rest ( for example update application throgh create new RC).

### Creating a Deployment
```yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
	name: kubia
spec:
	replicas: 3
	template:
		metadata:
			name: kubia
			labels:
				app: kubia
		spec:
			containers:
			- image: luksa/kubia:v1
			  name: nodejs
```

the Deployment can have multiple pod versions running under its wing, so its name shouldn’t reference the app version So it is above that version stuff.
```
# Use --record for revision history
kubectl create -f kubia-deployment-v1.yaml --record

kubectl get deployment
kubectl describe deployment

# Specifically for checking a Deployment’s status
kubectl rollout status deployment kubia

```

#### Recognize Deployment through POD name
their names were composed of the name of the controller plus a randomly generated string (for example, kubia-v1-m33mv). The three pods created by the Deployment include an additional numeric value in the middle of their names. The number corresponds to the hashed value of the pod template in the Deployment.

![[Screenshot from 2024-05-05 07-16-08.png]]

The ReplicaSet’s name also contains the hash value of its pod template.

![[Screenshot from 2024-05-05 07-16-38.png]]

As you’ll see later, a Deployment creates multiple ReplicaSets—one for each version of the pod template. Using the hash value of the pod template like this allows the Deployment to always use the same (possibly existing) ReplicaSet for a given version of the pod template.

### Updating a deployment
Now compare this to how you’re about to update a Deployment. The only thing you need to do is modify the pod template defined in the Deployment resource and Kubernetes will take all the steps necessary to get the actual system state to what’s defined in the resource.

#### UNDERSTANDING THE AVAILABLE DEPLOYMENT STRATEGIES
- RollingUpdate (default): removes old pods one by one, while adding new ones at the same time
- Recreate: deletes all the old pods at once and then creates new ones. Use this strategy when your application doesn’t support running multiple versions in parallel and requires the old version to be stopped completely before the new one is started.

![[Pasted image 20240505200117.png]]

### Rolling back a deployment
This rolls the Deployment back to the previous revision.
```
kubectl rollout undo deployment kubia
```

#### DISPLAYING A DEPLOYMENT ’ S ROLLOUT HISTORY
When a rollout completes, the old ReplicaSet isn’t deleted, and this enables rolling back to any revision, not only the previous one. The revision history can be displayed with the kubectl rollout history command:
```
kubectl rollout history deployment kubia
```

![[Pasted image 20240505203958.png]]

Remember the `--record` command-line option you used when creating the Deployment? Without it, the CHANGE-CAUSE column in the revision history would be empty, making it much harder to figure out what’s behind each revision.

For example, if you want to roll back to the first version:
```
kubectl rollout undo deployment kubia --to-revision=1
```

Each ReplicaSet stores the complete information of the Deployment at that specific revision, so you shouldn’t delete it manually. If you do, you’ll lose that specific revision from the Deployment’s history, preventing you from rolling back to it.

![[Pasted image 20240505204301.png]]

The length of the revision history is limited by the revisionHistoryLimit property on the Deployment resource. It defaults to two, Older ReplicaSets are deleted automatically.

### Controlling the rate of the rollout

**maxSurge** allowed the number of all pods to reach.
**maxUnavailable** disallowed having any unavailable pods.

```yaml
spec:
	strategy:
		rollingUpdate:
			maxSurge: 1
			maxUnavailable: 0
		type: RollingUpdate
```

![[Pasted image 20240505213338.png]]

Another example
![[Pasted image 20240505213354.png]]

### Pausing the rollout process
A Deployment can also be paused during\ the rollout process. This allows you to verify that everything is fine with the new version before proceeding with the rest of the rollout.

I’ve prepared the v4 image, so go ahead and trigger the rollout by changing the image to luksa/kubia:v4 , but then immediately (within a few seconds) pause the rollout:
```
kubectl set image deployment kubia nodejs=luksa/kubia:v4
kubectl rollout pause deployment kubia

kubectl rollout resume deployment kubia
```

A **canary** release is a technique for minimizing the risk of rolling out a bad version of an application and it affecting all your users. Instead of rolling out the new version to everyone, you replace only one or a small number of old pods with new ones. This way only a small number of users will initially hit the new version.

>If a Deployment is paused, the undo command won’t undo it until you resume the Deployment.

### Blocking rollouts of bad versions
To automate previose section (rollout puase):
The **minReadySeconds** property specifies how long a newly created pod should be ready before the pod is treated as available. Until the pod is available, the rollout process will not continue. 
You used this property to slow down your rollout process by having Kubernetes wait 10 seconds after a pod was ready before continuing with the rollout.

The Result YAML file:
```yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
	name: kubia
spec:
	replicas: 3
	minReadySeconds: 10
	strategy:
		rollingUpdate:
			maxSurge: 1
			maxUnavailable: 0
		type: RollingUpdate
	template:
	    metadata:
		    name: kubia
		    labels:
			    app: kubia
	    spec:
		    containers:
		    - image: luksa/kubia:v3
		      name: nodejs
		      readinessProbe:
			    periodSeconds: 1
			    httpGet:
				    path: /
				    port: 8080
```
