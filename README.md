# Java-WAR
```bash
$ kubectl apply -f multi-container-app-v133.yaml
pod/multi-container-app created

$ kubectl top pod multi-container-app --containers
POD                  NAME           CPU(cores)   MEMORY(bytes)
multi-container-app  web-server     450m         400Mi        # Under-utilized
multi-container-app  sidecar-proxy  50m          100Mi        # Idle, can't share

$ kubectl describe pod multi-container-app
Containers:
  web-server:
    Limits:      cpu: 1, memory: 1Gi
    Requests:    cpu: 500m, memory: 512Mi
  sidecar-proxy:
    Limits:      cpu: 500m, memory: 512Mi    # Wasted - proxy is idle
    Requests:    cpu: 200m, memory: 256Mi

# Resource waste: sidecar allocated but unused resources can't be shared
$ kubectl get pod multi-container-app -o jsonpath='{.status.containerStatuses[*].name}'
web-server sidecar-proxy

# Web server needs more CPU but can't access sidecar's unused allocation
```



```bash
$ kubectl apply -f multi-container-app-v134.yaml
pod/multi-container-app-v134 created

$ kubectl top pod multi-container-app-v134 --containers
POD                      NAME           CPU(cores)   MEMORY(bytes)
multi-container-app-v134 web-server     800m         600Mi        # Can use more!
multi-container-app-v134 sidecar-proxy  50m          100Mi        # Minimal usage

$ kubectl describe pod multi-container-app-v134
Pod Resources:           # Pod-level allocation
  Limits:      cpu: 1500m, memory: 1.5Gi
  Requests:    cpu: 700m, memory: 768Mi

Containers:
  web-server:            # No individual limits - shares pod pool
    Resources: <none>
  sidecar-proxy:         # No individual limits - shares pod pool
    Resources: <none>

# Dynamic resource sharing in action
$ kubectl exec multi-container-app-v134 -c web-server -- stress --cpu 1 --timeout 30s &
$ kubectl top pod multi-container-app-v134 --containers
POD                      NAME           CPU(cores)   MEMORY(bytes)
multi-container-app-v134 web-server     1200m        800Mi        # Using sidecar's unused CPU!
multi-container-app-v134 sidecar-proxy  50m          100Mi        # Still minimal

# Total pod usage stays within limits, but containers can dynamically share
```
