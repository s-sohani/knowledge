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

