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
```
