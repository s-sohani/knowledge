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

## Using persistent storage
