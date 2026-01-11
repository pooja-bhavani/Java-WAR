
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
