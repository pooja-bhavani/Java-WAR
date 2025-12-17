```bash
# Kubelet starts successfully with swap enabled
$ sudo swapon /swapfile
$ sudo systemctl start kubelet
$ sudo systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
   Active: active (running)

$ sudo journalctl -u kubelet | grep swap
Jan 15 10:00:00 node-1 kubelet[5678]: I0115 10:00:00.123456    5678 kubelet.go:123] 
"Swap is enabled with LimitedSwap behavior"

# Check swap configuration
$ kubectl get --raw /api/v1/nodes/node-1 | jq '.status.allocatable'
{
  "cpu": "4",
  "memory": "8Gi",
  "swap.alpha.kubernetes.io/memory": "2Gi"    # Swap available!
}

# Pod can use swap for memory bursts
$ kubectl apply -f burstable-workload-v134.yaml
$ kubectl get pods
NAME                READY   STATUS    RESTARTS   AGE
burstable-workload  1/1     Running   0          2m

$ kubectl top pod burstable-workload
NAME                CPU(cores)   MEMORY(bytes)
burstable-workload  100m         2.5Gi          # Using 2.5Gi (1Gi request + 1.5Gi from swap)

$ kubectl describe pod burstable-workload
QoS Class:       Burstable
Resources:
  Limits:        memory: 2Gi
  Requests:      memory: 1Gi

# Check node swap usage
$ kubectl get --raw /api/v1/nodes/node-1 | jq '.status.nodeInfo.swapUsage'
{
  "swapAvailableBytes": 536870912,    # 512Mi available
  "swapTotalBytes": 2147483648        # 2Gi total
}

# Pod survives memory pressure instead of being OOMKilled
Events:
  Normal  Started    2m    kubelet  Started container app
  Normal  SwapUsage  1m    kubelet  Container using swap: 512Mi
```
