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
Client pods that know the name of the service can access it through its fully qualified domain name (FQDN) instead of resorting to environment variables.
```
backend-database.default.svc.cluster.local
```

Backend-database corresponds to the service name, default stands for the namespace the service is defined in, and svc.cluster.local is a configurable cluster domain suffix used in all cluster local service names.

>The client must still know the service’s port number. If the service is using a standard port (for example, 80 for HTTP or 5432 for Postgres), that shouldn’t be a problem. If not, the client can get the port number from the environment variable.

>Connecting to a service can be even simpler than that. You can omit the svc.cluster. local suffix and even the namespace.


```
# Show incomming IP and PORT and list of Endpoint's IPs and PORTs
kubectl describe svc kubia

# List of Endpoint's IPs and PORTs
kubectl get endpoints kubia

```
## Connecting to services living outside the cluster
Redirect to external IP(s) and port(s):
- Load balancing
- Service discovery

### Manually configuring service endpoints (External Service)
To create a service with manually managed endpoints, you need to create both a Service and an Endpoints resource to connect External Service. 
```
# CREATING A SERVICE WITHOUT A SELECTOR
apiVersion: v1
kind: Service
metadata:
	name: external-service
spec:
	ports:
	- port: 80

---

# CREATING AN ENDPOINTS RESOURCE FOR A SERVICE WITHOUT A SELECTOR
apiVersion: v1
kind: Endpoints
metadata:
	name: external-service
subsets:
	- addresses:
		- ip: 11.11.11.11
		- ip: 22.22.22.22
	ports:
	- port: 80
```


![[Screenshot from 2024-04-29 07-05-26.png]]

#### Creating an alias for an external service With FQDN
```
apiVersion: v1
kind: Service
metadata:
 name: external-service
spec:
	type: ExternalName  # Service type is set to ExternalName
	externalName: someapi.somecompany.com
	ports:
	- port: 80
```

Now you can connect to external service with
`external-service.default.svc.cluster.local` OR `external-service`

>ExternalName services are implemented solely at the DNS level—a simple CNAME DNS record is created for the service. Therefore, clients connecting to the service will connect to the external service directly, bypassing the service proxy completely.

## Exposing services to external clients
You have a few ways to make a service accessible externally:
- NodePort:
- 
![[Screenshot from 2024-04-29 07-30-18.png]]

