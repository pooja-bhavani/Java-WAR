```bash
$ kubectl apply -f ml-training-job-v133.yaml
job.batch/ml-training-job created

# Simulate pod failure
$ kubectl delete pod ml-training-job-xyz --force
pod "ml-training-job-xyz" force deleted

$ kubectl get pods -w
NAME                    READY   STATUS        RESTARTS   AGE
ml-training-job-xyz     1/1     Terminating   0          2m
ml-training-job-abc     0/1     Pending       0          5s    # New pod created immediately!

# Both pods exist simultaneously - resource conflict
$ kubectl describe pod ml-training-job-abc
Events:
  Warning  FailedScheduling  30s  default-scheduler  0/3 nodes are available: 
  1 Insufficient nvidia.com/gpu (old pod still holds GPU while terminating)

# Resource waste and autoscaler confusion
$ kubectl get nodes
NAME     STATUS   ROLES    AGE   VERSION
node-1   Ready    <none>   1d    v1.33.0
node-2   Ready    <none>   1d    v1.33.0   # Autoscaler may add unnecessary nodes

# Old pod finally terminates, new pod can start
$ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
ml-training-job-abc     1/1     Running   0          2m
```

```bash
$ kubectl apply -f ml-training-job-v134.yaml
job.batch/ml-training-job-v134 created

# Simulate pod failure
$ kubectl delete pod ml-training-job-v134-xyz --force
pod "ml-training-job-v134-xyz" force deleted

$ kubectl get pods -w
NAME                        READY   STATUS        RESTARTS   AGE
ml-training-job-v134-xyz    1/1     Terminating   0          2m
# No new pod created yet - waiting for clean termination

$ kubectl describe job ml-training-job-v134
Spec:
  Pod Replacement Policy: Failed    # Wait for Failed status

# Pod reaches Failed status, resources fully released
$ kubectl get pods
NAME                        READY   STATUS   RESTARTS   AGE
ml-training-job-v134-xyz    0/1     Failed   0          3m

# Now new pod is created with clean resource handover
$ kubectl get pods -w
NAME                        READY   STATUS    RESTARTS   AGE
ml-training-job-v134-xyz    0/1     Failed    0          3m
ml-training-job-v134-def    0/1     Pending   0          1s    # Created after clean termination
ml-training-job-v134-def    1/1     Running   0          15s   # Starts immediately - no conflicts

$ kubectl describe pod ml-training-job-v134-def
Events:
  Normal  Scheduled  15s  default-scheduler  Successfully assigned to node-1
  Normal  Pulling    14s  kubelet           Pulling image "ml-training:latest"
  Normal  Pulled     10s  kubelet           Successfully pulled image
  Normal  Started    10s  kubelet           Started container trainer

# No resource conflicts, no unnecessary autoscaling
$ kubectl get nodes
NAME     STATUS   ROLES    AGE   VERSION
node-1   Ready    <none>   1d    v1.34.0   # No additional nodes needed
```
