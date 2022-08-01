## kubernetes provider

# Install & configure kubernetes provider

```
kubectl crossplane install provider crossplane/provider-kubernetes:main
```

```
kubectl apply -f examples/k8s-provider/provider-kubernetes.yaml 
```