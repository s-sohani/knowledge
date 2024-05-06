### Main parts of pod definition
- **Metadata**: includes the name, namespace, labels, and other information about the pod.
- **Spec**: contains the actual description of the pod’s contents, such as the pod’s containers, volumes, and other data.
- **Status**: contains the current information about the running pod, such as what condition the pod is in, the description and status of each container, and the pod’s internal IP and other basic info.

### Organizing pods with labels
categorizing them into subsets. In pod's `YAML` file write:
```
labels:
		creation_method: manual
		env: prod
```


### Kubernetes Has Declarative Approach
Horizontally scaling pods in Kubernetes is a matter of stating your desire: “I want to have x number of instances running.” You’re not telling Kubernetes what or how to do it. You’re just specifying the desired state.