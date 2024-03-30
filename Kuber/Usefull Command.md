#### Get YAML descriptor for existing pod
What is yaml definition for exist pods look like.
```yaml
kubectl get po kubia-zxzij -o yaml
```

#### To see which attributes are supported by each API object
```
kubectl explain pods
kubectl explain pod.spec
```
