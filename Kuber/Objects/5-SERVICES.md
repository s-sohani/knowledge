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


#### CONFIGURING SESSION AFFINITY ON THE SERVICE
If you want all requests made by a certain client to be redirected to the same pod every time.
```
apiVersion: v1
kind: Service
spec:
	sessionAffinity: ClientIP
```

Type of session affinity: 
- None 
- ClientIP

>Services deal with TCP and UDP packets and don’t care about the payload they carry. Because cookies are a construct of the HTTP protocol, services don’t know about them, which explains why session affinity cannot be based on cookies.

#### MULTIPLE PORTS IN THE SAME SERVICE

```
apiVersion: v1
kind: Service
metadata:
	name: kubia
spec:
	ports:
	- name: http
	  port: 80
	  targetPort: 8080
	- name: https
	  port: 443
	  targetPort: 8443
	selector:
	  app: kubia
```

>When creating a service with multiple ports, you must specify a name for each port.

#### USING NAMED PORTS
```
kind: Pod
spec:
	containers:
	- name: kubia
		  ports:
			- name: http
			  containerPort: 8080
			- name: https
			  containerPort: 8443

---

apiVersion: v1
kind: Service
spec:
	ports:
	- name: http
	  port: 80
	  targetPort: http
	- name: https
	  port: 443
	  targetPort: https
```

### Discovering services
How do the client pods know the IP and port of a service?

#### DISCOVERING SERVICES THROUGH ENVIRONMENT VARIABLES
If you create the service before creating the client pods, processes in those pods can get the IP address and port of the service by inspecting their environment variables.
OR
Create service and delete all pods to  be recreated pods with replicaSet and get service's ENV.

>In this way pod can list **all** Services Ip and Ports (services for other pods) in the **NameSpace** not only its own service.

Then in pod you can see:
```
KUBIA_SERVICE_HOST=10.111.249.153
KUBIA_SERVICE_PORT=80
```

#### DISCOVERING SERVICES THROUGH DNS
