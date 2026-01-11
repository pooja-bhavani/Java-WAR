```bash
# v1.34: The problematic scenario
$ kubectl apply -f ml-training.yaml

# What often happened - partial scheduling:
$ kubectl get pods
NAME                                 READY   STATUS    RESTARTS   AGE
ml-training-workers-7d4b8c5f-abc12   1/1     Running   0          2m
ml-training-workers-7d4b8c5f-def34   1/1     Running   0          2m
ml-training-workers-7d4b8c5f-ghi56   0/1     Pending   0          2m  # Stuck!
ml-training-workers-7d4b8c5f-jkl78   0/1     Pending   0          2m  # Stuck!
parameter-server                     0/1     Pending   0          2m  # Stuck!

# Result:
# 2 workers running but can't start training (need all 4)
# Wasted GPU resources (2 GPUs doing nothing)
# Job never completes
# Manual intervention required
```
