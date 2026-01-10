```yaml
# v1.35: Gang scheduling - all Pods scheduled together or none
apiVersion: workload.x-k8s.io/v1alpha1
kind: Workload
metadata:
  name: ml-training-job
spec:
  podSets:
  - name: workers
    count: 4
    template:
      spec:
        containers:
        - name: worker
          image: tensorflow/tensorflow:latest-gpu
          resources:
            requests:
              nvidia.com/gpu: 1
              cpu: "2"
              memory: "4Gi"
  - name: parameter-server
    count: 1
    template:
      spec:
        containers:
        - name: ps
          image: tensorflow/tensorflow:latest
          resources:
            requests:
              cpu: "1"
              memory: "2Gi"
---
apiVersion: workload.x-k8s.io/v1alpha1
kind: PodGroup
metadata:
  name: ml-training-group
spec:
  policy: gang  # All-or-nothing scheduling
  minMember: 5  # 4 workers + 1 parameter server
```

```bash
# v1.35: scheduling
$ kubectl apply -f ml-training-gang.yaml

# Gang scheduler waits for ALL resources to be available
$ kubectl get pods

NAME                     READY   STATUS    RESTARTS   AGE
# No pods created yet - waiting for full resource availability

# Once resources are available, ALL pods start together:
$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
ml-training-worker-0     1/1     Running   0          30s
ml-training-worker-1     1/1     Running   0          30s
ml-training-worker-2     1/1     Running   0          30s
ml-training-worker-3     1/1     Running   0          30s
ml-training-ps-0         1/1     Running   0          30s

# Result:
# All pods start simultaneously
# No wasted resources
# Training begins immediately
# Predictable job completion
```

---
