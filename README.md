**The Problem:** Resource changes required Pod recreation with downtime

```yaml
# v1.34: Original Pod with baseline resources
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


**Real-World Impact in v1.34:**
```bash
# Monitoring during v1.34 resource change
$ kubectl get pods -w
NAME           READY   STATUS    RESTARTS   AGE
database-pod   1/1     Running   0          5m
database-pod   1/1     Terminating   0      5m
database-pod   0/1     Pending       0      0s
database-pod   0/1     ContainerCreating   0   2s
database-pod   1/1     Running       0      45s

# 45 seconds of downtime! 
```
