```bash
# v1.35: Predictable traffic routing
$ kubectl get pods -o wide
NAME                          READY   STATUS    RESTARTS   AGE   IP           NODE
redis-cache-abc123           1/1     Running   0          5m    10.244.1.10   node-1
redis-cache-def456           1/1     Running   0          5m    10.244.2.15   node-2
redis-cache-ghi789           1/1     Running   0          5m    10.244.3.20   node-3

# Application pod on node-1
$ kubectl get pod app-pod -o wide
NAME      READY   STATUS    RESTARTS   AGE   IP           NODE
app-pod   1/1     Running   0          2m    10.244.1.25   node-1

# Traffic preferentially goes to local cache pod (redis-cache-abc123)
# Result:
# Low latency (same-node traffic)
# Better cache locality
# Reduced network overhead
# Predictable performance
```
