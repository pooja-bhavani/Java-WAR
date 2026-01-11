```bash
# v1.34: Complex setup required
# 1. Install cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.0/cert-manager.yaml

# 2. Create CA issuer
kubectl apply -f ca-issuer.yaml

# 3. Create certificate resource
kubectl apply -f certificate.yaml

# 4. Wait for certificate to be issued
kubectl wait --for=condition=Ready certificate/workload-cert

# 5. Finally deploy the Pod
kubectl apply -f secure-workload.yaml

# Result:
# Complex external dependencies
# Multiple components to manage
# Manual certificate rotation
# Potential security gaps
```
