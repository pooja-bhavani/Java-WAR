```yaml
# v1.35: Same Pod, but now supports in-place resize
apiVersion: v1
kind: Pod
metadata:
  name: database-pod
spec:
  containers:
  - name: postgres
    image: postgres:13
    resources:
      requests:
        cpu: "500m"
        memory: "1Gi"
      limits:
        cpu: "1000m"
        memory: "2Gi"
```

```bash
# v1.35: NO DOWNTIME!
kubectl patch pod database-pod --type='merge' -p='
{
  "spec": {
    "containers": [
      {
        "name": "postgres",
        "resources": {
          "requests": {
            "cpu": "1000m",
            "memory": "2Gi"
          },
          "limits": {
            "cpu": "2000m",
            "memory": "4Gi"
          }
        }
      }
    ]
  }
}'

# Result:
# Zero downtime
# Connections preserved
# State maintained
# Instant resource adjustment
```
