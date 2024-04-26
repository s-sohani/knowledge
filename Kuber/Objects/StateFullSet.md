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
