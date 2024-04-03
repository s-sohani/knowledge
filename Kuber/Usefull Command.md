#### Get YAML descriptor for existing pod
What is yaml definition for exist pods look like.
```
kubectl get po kubia-zxzij -o yaml
```

#### Show pods with its labels
```
kubectl get po --show-labels
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
