#### CONFIGURING TAB COMPLETION FOR KUBECTL
To enable tab completion in bash, you’ll first need to install a package called bash completion and then run the following command (you’ll probably also want to add it to ~/.bashrc or equivalent):
```
$ source <(kubectl completion bash)
```

#### Get YAML descriptor for existing POD
What is yaml definition for exist pods look like.
```
kubectl get po kubia-zxzij -o yaml
```

#### To see which attributes are supported by each API object
```
kubectl explain pods
kubectl explain pod.spec
```

#### Print log
```
kubectl logs kubia-manual
# Show previos log
kubectl logs -p kubia-manual
```

If your pod includes multiple **containers** and you want see specific container's log:
```
kubectl logs kubia-manual -c <container name>
kubectl logs kubia-manual -c kubia
```

#### Port forward
```
kubectl port-forward kubia-manual 8888:8080
```

#### Create POD
```
kubectl create -f kubia-manual.yaml
```

#### Show pods with its labels
```
kubectl get po --show-labels
# Show spicific labels
kubectl get po -L creation_method,env
# Show PODs contain label
kubectl get po -l creation_method=manual
# Show PODs contain key 
kubectl get po -l env
# Show PODs NOT contain key
kubectl get po -l '!env'
# Similarly, you could also match pods with the following label selectors
creation_method!=manual
env in (prod,devel)
env notin (prod,devel)
# Using multiple conditions in a label selector through comma seperate
app=pc,rel=beta
```
#### Edit/Create POD's label
```
kubectl label po kubia-manual-v2 env=debug
# To edit label
kubectl label po kubia-manual-v2 env=debug --overwrite
# Or in YAML file 
labels:
		creation_method: manual
		env: prod
```
#### Using labels and selectors to constrain pod scheduling
```
# 1- Lebeled node 
kubectl label node gke-kubia-85f6-node-0rrx gpu=true
# 2- Create YAML file and define node selector
apiVersion: v1
kind: Pod
metadata:
	name: kubia-gpu
spec:
	nodeSelector:
	  gpu: "true"
	containers:
	- image: luksa/kubia
	  name: kubia
```
#### Scheduling to one specific node
each node also has a unique label with the key `kubernetes.io/hostname` and value set to the actual host-name of the node.
#### Check client permision
```
kubectl auth can-i create sts -n sre-prod
```

#### Annotation
Annotations can hold much larger pieces of information and are primarily meant to be used by tools.
Certain annotations are automatically added to objects by Kubernetes, but others are added by users manually.
```
apiVersion: v1
kind: pod
metadata:
	annotations:
		kubernetes.io/created-by: |
		{"kind":"SerializedReference", "apiVersion":"v1",
		"reference":{"kind":"ReplicationController", "namespace":"default", ...

 ---

kubectl annotate pod kubia-manual mycompany.com/someannotation="foo bar"
```

#### Namespaces 
```
kubectl get ns
kubectl get po --namespace kube-system

# Create Namespace From Cli
kubectl create namespace custom-namespace

# From YAML file
apiVersion: v1
kind: Namespace
metadata:
	name: custom-namespace
# Then execute this command
kubectl create -f custom-namespace.yaml
```

> Most objects’ names must conform to the naming conventions specified in RFC 1035 (Domain names)

To create object or pod in specific Namespace, use -n in cli:
```
kubectl create -f kubia-manual.yaml -n custom-namespace
```
Or in YAML file, define Namespace in metadata.

> Namespace isolate all resources except they don’t provide any kind of isolation of running objects. For example network on pods that has public IP over kubernates.

#### Stopping and removing pods
```
# Sends a SIGTERM signal, wait 30s, if is running yet sends SIGKILL
kubectl delete po kubia-gpu

# Deleting pods using label selectors
kubectl delete po -l creation_method=manual
kubectl delete po -l rel=canary

# Deleting pods by deleting the whole namespace
kubectl delete ns custom-namespace

# Deleting all pods in current namespace, while keeping the namespace
# Note: If pod is created with ReplicationControll, pod recreated as soon as you # delete them, so you must delete ReplicationControll first.
kubectl delete po --all

# Deleting (almost) all resources in a namespace
# Note: Secrets are preserved and need to be deleted explicitly.
kubectl delete all --all
```

#### Why the container had to be restarted
See Last State section.
```
kubectl describe po kubia-liveness
```

