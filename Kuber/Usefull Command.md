#### Get YAML descriptor for existing pod
What is yaml definition for exist pods look like.
```
kubectl get po kubia-zxzij -o yaml
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

