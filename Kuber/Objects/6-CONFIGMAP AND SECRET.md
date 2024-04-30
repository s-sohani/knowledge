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



