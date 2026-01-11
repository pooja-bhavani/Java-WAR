```yaml
# v1.34: Traditional approach - Pods scheduled independently
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ml-training-workers
spec:
  replicas: 4  # Need all 4 to work together
  selector:
    matchLabels:
      app: ml-worker
  template:
    metadata:
      labels:
        app: ml-worker
    spec:
      containers:
      - name: worker
        image: tensorflow/tensorflow:latest-gpu
        resources:
          requests:
            nvidia.com/gpu: 1
            cpu: "2"
            memory: "4Gi"
---
# Separate parameter server
apiVersion: v1
kind: Pod
metadata:
  name: parameter-server
spec:
  containers:
  - name: ps
    image: tensorflow/tensorflow:latest
    resources:
      requests:
        cpu: "1"
        memory: "2Gi"
```
