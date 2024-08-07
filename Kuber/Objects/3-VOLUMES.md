Pods is logical hosts that use  be shared resources like CPU and RAM, but storage can't be used share. so a pod has its own isolated filesystem. a volume is created when the pod is started and is destroyed when the pod is deleted. Because of this, a volume’s contents will persist across container restarts. After a container is restarted, the new container can see all the files that were written to the volume by the previous container. Also, if a pod contains multiple containers, the volume can be used by all of them at once. A volume is available to all containers in the pod, but it must be mounted in each container that needs to access it.
In Linux and Kubernetes can mounting the same volume into two or more containers in a pod. 

## Volume Types
-  **emptyDir**: A simple empty directory used for storing transient data.
-  **hostPath**: Used for mounting directories from the worker node’s filesystem into the pod.
- **gitRepo**: A volume initialized by checking out the contents of a Git repository.
- **nfs**: An NFS share mounted into the pod.
- **gcePersistentDisk (Google Compute Engine Persistent Disk), awsElastic BlockStore (Amazon Web Services Elastic Block Store Volume), azureDisk (Microsoft Azure Disk Volume)**: Used for mounting cloud provider-specific storage.Using volumes to share data between containers 163.
- **cinder, cephfs, iscsi, flocker, glusterfs, quobyte, rbd, flexVolume, vsphere Volume, photonPersistentDisk, scaleIO**: Used for mounting other types of network storage.
- **configMap, secret, downwardAPI**: Special types of volumes used to expose certain Kubernetes resources and cluster information to the pod.
- **persistentVolumeClaim**: A way to use a pre- or dynamically provisioned persistent storage.

## Using volumes to share data between containers
### Using an emptyDir volume
An emptyDir volume is especially useful for sharing files between containers running in the same pod. But it can also be used by a single container for when a container needs to write data to disk temporarily. Because the volume’s lifetime is tied to that of the pod, the volume’s contents are lost when the pod is deleted.
```
apiVersion: v1
kind: Pod
metadata:
	name: fortune
spec:
	containers:
		- image: luksa/fortune
			name: html-generator
			volumeMounts:
			- name: html
			  mountPath: /var/htdocs
		- image: nginx:alpine
			name: web-server
			volumeMounts:
			- name: html
			  mountPath: /usr/share/nginx/html
			  readOnly: true
		ports:
		- containerPort: 80
		  protocol: TCP
	volumes:
	- name: html
	  emptyDir: {}
```

#### SPECIFYING THE MEDIUM TO USE FOR THE EMPTYDIR
You can tell Kubernetes to create the emptyDir on a tmpfs filesystem (in memory
instead of on disk).
```
volumes:
- name: html
  emptyDir:
    medium: Memory
```

### Using a Git repository as the starting point for a volume
After the gitRepo volume is created, it isn’t kept in sync with the repo it’s referencing. The files in the volume will not be updated when you push additional commits to the Git repository. However, if your pod is managed by a ReplicationController, deleting the pod will result in a new pod being created and this new pod’s volume will then contain the latest commits.
```
apiVersion: v1
kind: Pod
	metadata:
name: gitrepo-volume-pod
spec:
	containers:
	- image: nginx:alpine
		name: web-server
		volumeMounts:
		- name: html
		  mountPath: /usr/share/nginx/html
		  readOnly: true
		ports:
		- containerPort: 80
		  protocol: TCP
	volumes:
	- name: html
	  gitRepo:
		repository: https://github.com/luksa/kubia-website-example.git
		revision: master
		directory: .
```

## Accessing files on the worker node’s filesystem
A hostPath volume points to a specific file or directory on the node’s filesystem.
None of PODs uses the hostPath volume for storing their own data. They all use it to get access to the node’s data.
![[Screenshot from 2024-04-23 09-02-49.png]]

>The volume’s contents are stored on a specific node’s filesystem, when the database pod gets rescheduled to another node, it will no longer see the data.
## Using persistent storage
When an application running in a pod needs to persist data to disk and have that same data available even when the pod is rescheduled to another node, you can’t use any of the volume types we’ve mentioned so far. Because this data needs to be accessible from any cluster node, it must be stored on some type of network-attached storage (NAS).

### Using a GCE Persistent Disk in a pod volume
Google Compute Engine (GCE)
```
apiVersion: v1
kind: Pod
metadata:
	name: mongodb
spec:
	volumes:
	- name: mongodb-data
	  gcePersistentDisk:
		pdName: mongodb
		fsType: ext4
	containers:
		- image: mongo
		  name: mongodb
		  volumeMounts:
			- name: mongodb-data
			  mountPath: /data/db
		  ports:
		  - containerPort: 27017
			protocol: TCP
```

![[Screenshot from 2024-04-23 09-36-01.png]]
### Using other types of volumes with underlying persistent storage
#### USING AN NFS VOLUME

```
volumes:
- name: mongodb-data
	nfs:
		server: 1.2.3.4
		path: /some/path
```

## Decoupling pods from the underlying storage technology
In above config, user must now aboat NFS IP or infrastructure, but this agains Kubernetes idea. To enable apps to request storage in a Kubernetes cluster without having to deal with infrastructure specifics, two new resources were introduced. They are PersistentVolumes and PersistentVolumeClaims.

![[Pasted image 20240424195321.png]]

>Other users cannot use the same PersistentVolume until it has been released by deleting the bound PersistentVolumeClaim.

### Create PV
```
apiVersion: v1
kind: PersistentVolume
metadata:
	name: mongodb-pv
spec:
	capacity:
		storage: 1Gi
	accessModes:  # It can either be mounted by a single client for reading and writing or by multiple clients for reading only.
	- ReadWriteOnce
	- ReadOnlyMany
	persistentVolumeReclaimPolicy: Retain  #After the claim is released, the PersistentVolume should be retained (not erased or deleted).
	gcePersistentDisk:
		pdName: mongodb
		fsType: ext4
```

```
kubectl get pv
```
 > PersistentVolumes don’t belong to any namespace. They’re clusterlevel resources like nodes.
 
 ![[Pasted image 20240424202233.png]]

### Claiming a PersistentVolume by creating a PersistentVolumeClaim
Claiming a PersistentVolume is a completely separate process from creating a pod, because you want the same PersistentVolumeClaim to stay available even if the pod is rescheduled.
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
	name: mongodb-pvc
spec:
	resources:
		requests:
			storage: 1Gi
	accessModes:
	- ReadWriteOnce # support single client performing both reads and writes.
	storageClassName: "" # learn in the section about dynamic provisioning.
```

As soon as you create the claim, Kubernetes finds the appropriate PersistentVolume and binds it to the claim.

```
kubectl get pvc
```

#### Abbreviations used for the access modes:
- RWO — ReadWriteOnce —Only a single node can mount the volume for reading and writing.
-  ROX — ReadOnlyMany —Multiple nodes can mount the volume for reading.
-  RWX — ReadWriteMany —Multiple nodes can mount the volume for both reading and writing.
> PersistentVolume resources are cluster-scoped and thus cannot be created in a specific namespace, but PersistentVolumeClaims can only be created in a specific namespace. They can then only be used by pods in the same namespace.

### Using a PersistentVolumeClaim in a pod
```
apiVersion: v1
kind: Pod
metadata:
	name: mongodb
spec:
	containers:
	- image: mongo
	  name: mongodb
	  volumeMounts:
		- name: mongodb-data
		  mountPath: /data/db
	  ports:
	  - containerPort: 27017
	  protocol: TCP
	volumes:
	- name: mongodb-data
	  persistentVolumeClaim:
	  claimName: mongodb-pvc
```

### Recycling PersistentVolumes
When you delete pod and pvc, The STATUS column shows the PersistentVolume as Released , not Available like before. Because you’ve already used the volume, it may contain data and shouldn’t be bound to a completely new claim without giving the cluster admin a chance to clean it up. Without this, a new pod using the same PersistentVolume could read the data stored there by the previous pod, even if the claim and pod were created in a different namespace.

### persistentVolumeReclaimPolicy
- Retain: Retain the volume and its contents after it’s released from its claim. 
- Recycle/Delete: Deletes the volume’s contents and makes the volume available to be claimed again.

## Dynamic provisioning of PersistentVolumes
Creating PV still requires a cluster administrator to provision the actual storage up front.
The cluster admin, instead of creating PersistentVolumes, can deploy a PersistentVolume provisioner and define one or more StorageClass objects to let users choose what type of PersistentVolume they want.

>Kubernetes includes provisioners for the most popular cloud providers. Instead of the administrator pre-provisioning a bunch of PersistentVolumes, they need to define one or two (or more) StorageClasses.

### Defining the available storage types through StorageClass resources

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
	name: fast
provisioner: kubernetes.io/gce-pd
parameters:  # The parameters passed to the provisioner
	type: pd-ssd
	zone: europe-west1-b
```

### CREATING A PVC DEFINITION REQUESTING A SPECIFIC STORAGE CLASS
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
	name: mongodb-pvc
spec:
	storageClassName: fast
	resources:
		requests:
			storage: 100Mi
			accessModes:
			- ReadWriteOnce
```

### Dynamic provisioning without specifying a storage class
The default storage class is what’s used to dynamically provision a PersistentVolume if the PersistentVolumeClaim doesn’t explicitly say which storage class to use.

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
	name: mongodb-pvc2
spec:
	resources:
		requests:
			storage: 100Mi
			accessModes:
			- ReadWriteOnce
```

This PVC definition includes only the storage size request and the desired access modes, but no storage class.

### THE COMPLETE PICTURE OF DYNAMIC P ERSISTENT VOLUME PROVISIONING

![[Pasted image 20240426105213.png]]

