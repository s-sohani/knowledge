Pods need a way of finding other pods.
A Kubernetes Service is a resource you create to make a single, constant point of entry to a group of pods providing the same service.

### Creating services
With LabelSelector we can match pods and services.

![[Screenshot from 2024-04-28 06-56-00.png]]

#### CREATING A SERVICE THROUGH KUBECTL EXPOSE
exposing all its pods through a single IP address and port.
This is the cluster IP, it’s only accessible from inside the cluster.
```
apiVersion: v1
kind: Service
metadata:
	name: kubia
spec:
	ports:
	- port: 80
		targetPort: 8080
	selector:
		app: kubia
```

#### List all Service resources
```
kubectl get svc
```


##