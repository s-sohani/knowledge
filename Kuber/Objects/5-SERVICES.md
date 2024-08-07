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
- **NodePort**: Each cluster node opens a port on the node itself.
- **LoadBalancer**: An extension of the NodePort, 
- **Ingress**: For exposing multiple services through a single IP address.

![[Screenshot from 2024-04-29 07-30-18.png]]

### Using a NodePort service
By creating a NodePort service, reserve a port on all its nodes. Pod can accessible through any node’s IP and the reserved node port.
```
apiVersion: v1
kind: Service
metadata:
	name: kubia-nodeport
spec:
	type: NodePort
	ports:
	- port: 80
	  targetPort: 8080
	  nodePort: 30123
	selector:
		app: kubia
```

> Kubernetes will choose a random port if you omit nodePort.

```
kubectl get svc kubia-nodeport
```

The service is accessible at the following addresses:
- 10.11.254.223:80
- <1st node’s IP>:30123
- <2nd node’s IP>:30123

![[Screenshot from 2024-04-29 07-45-14.png]]

>If you only point your clients to the first node, when that node fails, your clients can’t access the service anymore.

### Exposing a service through an external load balancer

```
apiVersion: v1
kind: Service
metadata:
	name: kubia-loadbalancer
spec:
	type: LoadBalancer
	ports:
	- port: 80
	  targetPort: 8080
	selector:
		app: kubia
```

```
kubectl get svc kubia-loadbalancer
```

Once it does that, the IP address will be listed as the external IP address of your service, In this case, the loadbalancer is available at IP 130.211.53.173, so you can now access the service at that IP address.

>The browser is using keep-alive connections and sends all its requests through a single connection, whereas curl opens a new connection every time. Services work at the connection level, so when a connection to a service is first opened, a random pod is selected and then all network packets belonging to that connection are all sent to that single pod. Even if session affinity is set to None, users will always hit the same pod.


![[Screenshot from 2024-04-29 08-00-41.png]]

### Understanding the peculiarities of external connections
When an external client connects to a service through the node port (this also includes cases when it goes through the load balancer first), the randomly chosen pod may or may not be running on the same node that received the connection. An additional network hop is required to reach the pod, but this may not always be desirable.
You can prevent this additional hop by configuring the service to redirect external traffic only to pods running on the node that received the connection. This is done by setting the externalTrafficPolicy field in the service’s spec section:
```
spec:
	externalTrafficPolicy: Local
```

If a service definition includes this setting and an external connection is opened through the service’s node port, the service proxy will choose a locally running pod. If no local pods exist, the connection will hang (it won’t be forwarded to a random global pod, the way connections are when not using the annotation). You therefore need to ensure the load balancer forwards connections only to nodes that have at least one such pod.
Using this annotation also has other drawbacks. Normally, connections are spread evenly across all the pods, but when using this annotation, that’s no longer the case. Imagine having two nodes and three pods. Let’s say node A runs one pod and node B runs the other two. If the load balancer spreads connections evenly across the two nodes, the pod on node A will receive 50% of all connections, but the two pods on node B will only receive 25% each.

![[Screenshot from 2024-04-29 08-10-49.png]]

### Exposing services externally through an Ingress resource

![[Pasted image 20240429184735.png]]
#### Enabling the Ingress add-on in Minikube
```
minikube addons list
minikube addons enable ingress
```

#### Creating an Ingress resource
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
	name: kubia
spec:
	rules:
	- host: kubia.example.com  # This Ingress maps this to your service.
	  http:
		paths:
		- path: /
		  backend:
			serviceName: kubia-nodeport # All requests will be sent to port 80 of the kubia-nodeport service.
			servicePort: 80

---

# Define different paths
- host: kubia.example.com
	http:
		paths:
			- path: /kubia
			  backend:
				serviceName: kubia
				servicePort: 80
			- path: /foo
			  backend:
				serviceName: bar
				servicePort: 80

---

# Define different services to different hosts
spec:
	rules:
	- host: foo.example.com
		http:
			paths:
			- path: /
			  backend:
				serviceName: foo
				servicePort: 80
	- host: bar.example.com
		http:
			paths:
			- path: /
			  backend:
				serviceName: bar
				servicePort: 80
```

```
kubectl get ingresses
```

#### Configuring Ingress to handle TLS traffic
```
openssl genrsa -out tls.key 2048
openssl req -new -x509 -key tls.key -out tls.cert -days 360 -subj /CN=kubia.example.com

# Secret from the two files like this:
kubectl create secret tls tls-secret --cert=tls.cert --key=tls.key

# Ingress handling TLS traffic:
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
	name: kubia
spec:
	tls:
	- hosts:
		- kubia.example.com
		  secretName: tls-secret
	rules:
		- host: kubia.example.com
			http:
			paths:
			- path: /
			  backend:
			  serviceName: kubia-nodeport
			  servicePort: 80

```

### Signaling when a pod is ready to accept connections
When create a pod, if you don’t want the pod to start receiving requests immediately (when pod required to load some data or config to became ready), you can use readiness:

#### TYPES OF READINESS PROBES
- An Exec probe, where a process is executed. The container’s status is determined by the process’ exit status code.
- An HTTP GET probe, which sends an HTTP GET request to the container and the HTTP status code of the response determines whether the container is ready or not.
- A TCP Socket probe, which opens a TCP connection to a specified port of the container. If the connection is established, the container is considered ready.

When a container is started, Kubernetes can be configured to wait for a configurable amount of time to pass before performing the first readiness check. After that, it invokes the probe periodically and acts based on the result of the readiness probe. If a pod reports that it’s not ready, it’s removed from the service. If the pod then becomes ready again, it’s re-added.

>Unlike liveness probes, if a container fails the readiness check, it won’t be killed or restarted.

#### ADDING A READINESS PROBE TO THE POD TEMPLATE

```
kubectl edit rc kubia

---

spec:
	containers:
	- name: kubia
	  image: luksa/kubia
	  readinessProbe:
		exec:
			command:
			- ls
			- /var/ready
```

### Using a headless service for discovering individual pods
- the client needs to connect to all of those pods
- the backing pods themselves need to each connect to all the other backing pods.

Kubernetes allows clients to discover pod IPs through DNS lookups. Usually, when you perform a DNS lookup for a service, the DNS server returns a single IP, the service’s cluster IP. But if you tell Kubernetes you don’t need a cluster IP for your service (you do this by setting the clusterIP field to None in the service specification ) , the DNS server will return the pod IPs instead of the single service IP. Instead of returning a single DNS A record, the DNS server will return multiple A records for the service, each pointing to the IP of an individual pod backing the service at that moment. Clients can therefore do a simple DNS A record lookup and get the IPs of all the pods that are part of the service. The client can then use that information to connect to one, many, or all of them.

So with headless service, because DNS returns the pods’ IPs, clients connect directly to the pods, instead of through the service proxy.

```
apiVersion: v1
kind: Service
metadata:
	name: kubia-headless
spec:
	clusterIP: None
	ports:
	- port: 80
	  targetPort: 8080
	selector:
	  app: kubia
```


