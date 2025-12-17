# Java-WAR

```bash
$ kubectl apply -f gpu-workload-v133.yaml
pod/gpu-workload created

$ kubectl get pods -o wide
NAME           READY   STATUS    RESTARTS   AGE   NODE
gpu-workload   1/1     Running   0          30s   gpu-node-1

$ kubectl describe pod gpu-workload
Resources:
  Limits:
    nvidia.com/gpu: 1    # Entire GPU allocated, no sharing possible
  Requests:
    nvidia.com/gpu: 1

# Second pod trying to use same GPU
$ kubectl apply -f gpu-workload-2.yaml
pod/gpu-workload-2 created

$ kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
gpu-workload     1/1     Running   0          2m
gpu-workload-2   0/1     Pending   0          30s   # Stuck - no GPU available

$ kubectl describe pod gpu-workload-2
Events:
  Warning  FailedScheduling  30s  default-scheduler  0/3 nodes are available: 1 Insufficient nvidia.com/gpu
```
