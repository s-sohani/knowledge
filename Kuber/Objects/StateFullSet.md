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
You could create multiple ReplicaSets.