# Java-WAR

```bash
$ kubectl apply -f gpu-workload-v134.yaml
resourceclaim.resource.k8s.io/gpu-slice-claim created
pod/gpu-workload-v134 created

$ kubectl get resourceclaims
NAME              CLASS       ALLOCATION   AGE
gpu-slice-claim   gpu-slice   allocated    45s

$ kubectl get pods -o wide
NAME                READY   STATUS    RESTARTS   AGE   NODE
gpu-workload-v134   1/1     Running   0          45s   gpu-node-1

$ kubectl describe pod gpu-workload-v134
Resources:
  Claims:
    Name: gpu-slice
    Request: gpu.example.com/memory: 4Gi  # Only using 4GB of 24GB GPU memory

# Second pod can now share the same GPU
$ kubectl apply -f gpu-workload-2-v134.yaml
resourceclaim.resource.k8s.io/gpu-slice-claim-2 created
pod/gpu-workload-2-v134 created

$ kubectl get pods
NAME                  READY   STATUS    RESTARTS   AGE
gpu-workload-v134     1/1     Running   0          2m
gpu-workload-2-v134   1/1     Running   0          30s   # Both running on same GPU!

$ kubectl describe resourceclaim gpu-slice-claim
Status:
  Allocation:
    Available On Nodes:
      - gpu-node-1
    Resource Handles:
    - Data: gpu.example.com/memory=4Gi,gpu.example.com/cores=25%
      Driver Name: gpu.example.com
```
