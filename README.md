# Java-WAR
```bash
$ kubectl apply -f private-image-pod-v133.yaml
secret/registry-secret created
pod/private-image-pod created

$ kubectl get secrets
NAME              TYPE                             DATA   AGE
registry-secret   kubernetes.io/dockerconfigjson   1      30s

$ kubectl describe secret registry-secret
Type:  kubernetes.io/dockerconfigjson
Data
====
.dockerconfigjson:  156 bytes   # Static, never expires

$ kubectl get pods
NAME                READY   STATUS    RESTARTS   AGE
private-image-pod   1/1     Running   0          45s

# Security issue: Secret persists indefinitely
$ kubectl get secret registry-secret -o yaml | grep creationTimestamp
  creationTimestamp: "2024-01-15T10:00:00Z"   # Created weeks ago, still valid

# If compromised, provides long-term access
$ echo "Secret leaked - attacker has permanent registry access until manually rotated"
```
