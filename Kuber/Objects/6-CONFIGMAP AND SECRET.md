You can configure your app by 
- Passing command-line arguments to containers
- Setting custom environment variables for each container
- Mounting configuration files into containers through a special type of volume
## Passing command-line arguments to containers
Kubernetes allows overriding the command as part of the pod’s container definition instead of default container command.

```YAML
kind: Pod
spec:
	containers:
	- image: some/image
	  command: ["/bin/command"]
	  args: ["arg1", "arg2", "arg3"]
```

![[Pasted image 20240430200557.png]]

#### Example 
``` Dockerfile
FROM ubuntu:latest
RUN apt-get update ; apt-get -y install fortune
ADD fortuneloop.sh /bin/fortuneloop.sh
ENTRYPOINT ["/bin/fortuneloop.sh"]
CMD ["10"]
```

```YAML
apiVersion: v1
kind: Pod
metadata:
	name: fortune2s
spec:
	containers:
	- image: luksa/fortune:args
	  args: ["2"]  # Override argument
	  name: html-generator
	  volumeMounts:
		- name: html
		  mountPath: /var/htdocs
```


>You don’t need to enclose string values in quotations marks (but you must enclose numbers).
>args:
>- foo
>- bar
>- "15"


## Setting environment variables for a container

```yaml
kind: Pod
spec:
	containers:
	- image: luksa/fortune:env
	env:
	- name: INTERVAL
	  value: "30"
	  name: html-generator
```

### Referring to other environment variables in a variable’s value
```yaml
env:
- name: FIRST_VAR
  value: "foo"
- name: SECOND_VAR
  value: "$(FIRST_VAR)bar"
```

## Decoupling configuration with a ConfigMap
The contents of the map are instead passed to containers as either environment variables or as files in a volume.

![[Pasted image 20240430202927.png]]

>You can keep multiple manifests for ConfigMaps with the same name, each for a different environment.

![[Pasted image 20240430203402.png]]

### Creating a ConfigMap

```
# USING THE KUBECTL CREATE CONFIGMAP COMMAND
kubectl create configmap fortune-config --from-literal=sleep-interval=25

# Create a ConfigMap with multiple literal entries
kubectl create configmap myconfigmap --from-literal=foo=bar --from-literal=bar=baz --from-literal=one=two

# Create YAML ConfigMap
apiVersion: v1
data:
	sleep-interval: "25"
kind: ConfigMap

# CREATING A CONFIGMAP ENTRY FROM THE CONTENTS OF A FILE
kubectl create configmap my-config --from-file=config-file.conf

# You can also specify a key manually for content of file
kubectl create configmap my-config --from-file=customkey=config-file.conf

# CREATING A CONFIG MAP FROM FILES IN A DIRECTORY
kubectl create configmap my-config --from-file=/path/to/dir

# COMBINING DIFFERENT OPTIONS
kubectl create configmap my-config
➥ --from-file=foo.json
➥ --from-file=bar=foobar.conf
➥ --from-file=config-opts/        # A whole directory
➥ --from-literal=some=thing
```

![[Pasted image 20240430210511.png]]


### Passing a ConfigMap entry to a container as an environment variable
```
apiVersion: v1
kind: Pod
metadata:
	name: fortune-env-from-configmap
spec:
	containers:
	- image: luksa/fortune:env
	env:
	- name: INTERVAL
	  valueFrom:
		configMapKeyRef:
			name: fortune-config
			key: sleep-interval
```

![[Pasted image 20240430211556.png]]

>The container referencing the non-existing ConfigMap will fail to start, but the other container will start normally.
>If you then create the missing ConfigMap, the failed container is started without requiring you to recreate the pod.

### If you then create the missing ConfigMap, the failed container is started without requiring you to recreate the pod.

```
spec:
	containers:
	- image: some-image
	  envFrom:
		- prefix: CONFIG_   # All environment variables will be prefixed with                                  CONFIG_.
		configMapRef:
		name: my-config-map
```

>CONFIG_FOO-BAR isn’t a valid environment variable name because it contains a dash. Kubernetes doesn’t convert the keys in any way (it doesn’t convert dashes to underscores, for example). If a ConfigMap key isn’t in the proper format, it skips the entry

### Passing a ConfigMap entry as a command-line argument
You can first initialize an environment variable from the ConfigMap entry and then refer to the variable inside the arguments.

![[Pasted image 20240430212819.png]]

```
apiVersion: v1
kind: Pod
metadata:
	name: fortune-args-from-configmap
spec:
	containers:
	- image: luksa/fortune:args
	  env:
		- name: INTERVAL
		  valueFrom:
			configMapKeyRef:
				name: fortune-config
				key: sleep-interval
		args: ["$(INTERVAL)"]
```

### Using a configMap volume to expose ConfigMap entries as files
Passing configuration options as environment variables or command-line arguments is usually used for short variable values. A ConfigMap, as you’ve seen, can also contain whole config files.
In this case you can use volume namely a configMap volume.
The process running in the container can obtain the entry’s value by reading the contents of the file.

#### Example
Create a new directory called configmap-files like this:
![[Pasted image 20240430223952.png]]

Now create a ConfigMap from all the files in the directory like this:
```
kubectl create configmap fortune-config --from-file=configmap-files
```

```
kubectl get configmap fortune-config -o yaml

apiVersion: v1
data:
	my-nginx-config.conf: |
		server {
			listen 80;
			server_name  www.kubia-example.com;
			gzip on;
			gzip_types text/plain application/xml;
			location / {
				root  /usr/share/nginx/html;
				index index.html index.htm;
			}
		}
	sleep-interval: |
	25
kind: ConfigMap
```

Now you can use both configMaps:
![[Pasted image 20240430224300.png]]

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: fortune-configmap-volume
spec:
	containers:
	- image: nginx:alpine
	  name: web-server
	  volumeMounts:
	      ...
		- name: config
		  mountPath: /etc/nginx/conf.d
		  readOnly: true
	  ...
    volumes:
	...  
	- name: config
	  configMap:
		  name: fortune-config
```

To define which entries should be exposed as files in a configMap volume, use the volume’s items attribute as shown in the following listing.
```yaml
volumes:
- name: config
  configMap:
	name: fortune-config
	items:
	- key: my-nginx-config.conf
	  path: gzip.conf
```

>in Linux when you mount a filesystem into a nonempty directory. The directory then only contains the files from the mounted filesystem, whereas the original files in that directory are inaccessible for as long as the filesystem is mounted.

How to add individual files from a ConfigMap into an existing directory **without** hiding existing files stored in it?
```yaml
spec:
	containers:
	- image: some/image
	  volumeMounts:
		- name: myvolume
		  mountPath: /etc/someconfig.conf
		  subPath: myconfig.conf
```

![[Pasted image 20240430225826.png]]

>This method of mounting individual files has a relatively big deficiency related to updating files.

#### SETTING THE FILE PERMISSIONS FOR FILES IN A CONFIG MAP VOLUME
By default, the permissions on all files in a configMap volume are set to 644 ( -rw-r—r-- ).
You can change this by setting the defaultMode property in the volume spec
```yaml
volumes:
- name: config
  configMap:
	name: fortune-config
	defaultMode: "6600"
```

## Using Secrets to pass sensitive data to containers
Secrets are much like ConfigMaps:
- Pass Secret entries to the container as environment variables
- Expose Secret entries as files in a volume

All secrest stores in master node, `etcd database`, and deploy in node thad run a pod that need access to this secred. so it doesn't write in storage, keep in memory. 

### Introducing the default token Secret
When using kubectl describe on a pod you can see every pod has a secret volume attached to it automatically called (for example) `default-token-cfee9`.
If describe secret you can see ca.crt , namespace , and token which represent everything you need to securely talk to the Kubernetes API.

### Creating a Secret
You’re creating a generic Secret, but you could also have created a tls Secret with the kubectl create secret tls command, This would create the Secret with different entry names

```
kubectl create secret generic fortune-https --from-file=https.key
➥ --from-file=https.cert --from-file=foo
```

### Comparing ConfigMaps and Secrets
The contents of a Secret’s entries are shown as Base64-encoded strings, whereas those of a ConfigMap are shown in clear text.

The reason for using Base64 encoding is simple. A Secret’s entries can contain binary values, not only plain-text. Base64 encoding allows you to include the binary data in YAML or JSON, which are both plain-text formats.

>The maximum size of a Secret is limited to 1MB.

Because not all sensitive data is in binary form, Kubernetes also allows setting a Secret’s values through the stringData field.

```yaml
kind: Secret
apiVersion: v1
stringData:
	foo: plain text
data:
	https.cert: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCekNDQ...
	https.key: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcE...
```
