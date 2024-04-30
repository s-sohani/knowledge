You can configure your app by 
- Passing command-line arguments to containers
- Setting custom environment variables for each container
- Mounting configuration files into containers through a special type of volume
## Passing command-line arguments to containers
Kubernetes allows overriding the command as part of the pod’s container definition instead of default container command.

```
kind: Pod
spec:
	containers:
	- image: some/image
	  command: ["/bin/command"]
	  args: ["arg1", "arg2", "arg3"]
```

![[Pasted image 20240430200557.png]]

