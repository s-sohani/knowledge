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
each node also has a unique label with the key kubernetes.io/hostname and value set to the actual host-name of the node.
#### Check client permision
```
kubectl auth can-i create sts -n sre-prod
```


#### Discovering other namespaces and their pods
```
kubectl get ns
kubectl get po --namespace kube-system
```
