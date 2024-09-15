```bash
helm install <release-name> <chart-name> --namespace <namespace>

helm upgrade <release-name> <chart-name> --namespace <namespace> --values <values-file.yaml>

helm uninstall <release-name> --namespace <namespace>

# Get manifest
helm template <release-name> <chart-name> --namespace <namespace> --values <values-file.yaml> > manifest.yaml

```

