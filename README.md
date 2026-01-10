```yaml
# v1.34: Basic Service - no locality control
apiVersion: v1
kind: Service
metadata:
  name: cache-service
spec:
  selector:
    app: redis-cache
  ports:
  - port: 6379
    targetPort: 6379
  # No traffic distribution control available
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-cache
spec:
  replicas: 3
  selector:
    matchLabels:
      app: redis-cache
  template:
    metadata:
      labels:
        app: redis-cache
    spec:
      containers:
      - name: redis
        image: redis:7-alpine
```

```bash
# v1.34: Traffic routing was unpredictable
$ kubectl get pods -o wide
NAME                          READY   STATUS    RESTARTS   AGE   IP           NODE
redis-cache-abc123           1/1     Running   0          5m    10.244.1.10   node-1
redis-cache-def456           1/1     Running   0          5m    10.244.2.15   node-2
redis-cache-ghi789           1/1     Running   0          5m    10.244.3.20   node-3

# Application pod on node-1
$ kubectl get pod app-pod -o wide
NAME      READY   STATUS    RESTARTS   AGE   IP           NODE
app-pod   1/1     Running   0          2m    10.244.1.25   node-1

# Traffic could go to ANY cache pod, even remote ones
# Result:
# High latency (cross-node traffic)
# Poor cache locality
# Increased network overhead
# Unpredictable performance
```
