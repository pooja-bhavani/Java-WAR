```bash
# Kubelet needs broad permissions
$ kubectl auth can-i list pods --as=system:node:node-1
yes

# Kubelet can list ALL pods in cluster (security risk)
$ kubectl get pods --as=system:node:node-1 -A
NAMESPACE     NAME                          READY   STATUS    RESTARTS   AGE
kube-system   coredns-558bd4d5db-xyz        1/1     Running   0          1d
kube-system   etcd-master                   1/1     Running   0          1d
default       app-pod-1                     1/1     Running   0          1h
default       app-pod-2                     1/1     Running   0          1h
sensitive     secret-workload               1/1     Running   0          30m   # Kubelet can see this!

# No way to restrict based on node assignment
$ kubectl get pods --field-selector=spec.nodeName=node-2 --as=system:node:node-1
NAME        READY   STATUS    RESTARTS   AGE
other-pod   1/1     Running   0          1h    # Can see pods on other nodes!

# Audit log shows excessive access
$ grep "system:node:node-1" /var/log/audit/audit.log | grep pods
{"verb":"list","objectRef":{"resource":"pods","namespace":"default"}}
{"verb":"list","objectRef":{"resource":"pods","namespace":"kube-system"}}
{"verb":"list","objectRef":{"resource":"pods","namespace":"sensitive"}}   # Unnecessary access
```

```bash
# Fine-grained authorization with field selectors
$ kubectl auth can-i list pods --as=system:node:node-1
yes

# Kubelet can only list pods assigned to its node
$ kubectl get pods --as=system:node:node-1 -A
Error from server (Forbidden): pods is forbidden: User "system:node:node-1" 
cannot list resource "pods" in API group "" at the cluster scope

# Must use field selector for node-specific access
$ kubectl get pods --field-selector=spec.nodeName=node-1 --as=system:node:node-1
NAME              READY   STATUS    RESTARTS   AGE
node-1-pod-1      1/1     Running   0          1h
node-1-pod-2      1/1     Running   0          30m

# Cannot access pods on other nodes
$ kubectl get pods --field-selector=spec.nodeName=node-2 --as=system:node:node-1
Error from server (Forbidden): pods is forbidden: field selector "spec.nodeName=node-2" 
not allowed for user "system:node:node-1"

# Authorization policy enforces node isolation
$ kubectl describe clusterrole system:node
Rules:
  Resources  Non-Resource URLs  Resource Names  Verbs
  pods       []                 []              [get list]
  # But authorization middleware checks field selectors:
  # - spec.nodeName must match requesting node
  # - Cannot list pods without node-specific selector

# Audit log shows restricted access
$ grep "system:node:node-1" /var/log/audit/audit.log | grep pods
{"verb":"list","objectRef":{"resource":"pods"},"requestObject":{"fieldSelector":"spec.nodeName=node-1"}}
# Only shows access to own node's pods

# Multi-tenant security improvement
$ kubectl get pods --as=system:node:node-1 --field-selector=metadata.namespace=tenant-a
Error from server (Forbidden): field selector combination not allowed
# Prevents cross-tenant pod visibility
```

