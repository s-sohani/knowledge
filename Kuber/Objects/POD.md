and need to be deleted explicitly.```
apiVersion: v1
kind: Pod
metadata:
	name: kubia-manual
	labels:
		creation_method: manual
		env: prod
spec:
	containers:
	- image: luksa/kubia
	  name: kubia
	  ports:
      - containerPort: 8080
	    protocol: TCP


>Specifying ports in the pod definition is purely informational. Omitting them has no effect on whether clients can connect to the pod through the port or not. If the con-64 CHAPTER 3 Pods: running containers in Kubernetes tainer is accepting connections through a port bound to the 0.0.0.0 address, other pods can always connect to it, even if the port isn’t listed in the pod spec explicitly. But it makes sense to define the ports explicitly so that everyone using your cluster can quickly see what ports each pod exposes. Explicitly defining ports also allows you to assign a name to each port, which can come in handy, as you’ll see later in the book.


#### Create pod
After write pod's yaml file, execute script to create pod:
```
kubectl create -f kubia-manual.yaml
```

