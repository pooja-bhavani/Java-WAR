```bash
$ kubectl apply -f dynamic-storage-v134.yaml
volumeattributesclass.storage.k8s.io/high-performance created
persistentvolumeclaim/dynamic-storage-v134 created

$ kubectl get pvc
NAME                   STATUS   VOLUME                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
dynamic-storage-v134   Bound    pvc-xyz789-abc123-def456   100Gi      RWO            gp3            30s

$ kubectl get volumeattributesclass
NAME               AGE
high-performance   30s

# Check initial volume attributes
$ aws ec2 describe-volumes --volume-ids vol-xyz789abc123def456
{
    "Volumes": [{
        "VolumeId": "vol-xyz789abc123def456", 
        "Iops": 10000,          # From VolumeAttributesClass
        "Throughput": 500       # From VolumeAttributesClass
    }]
}

# Need more performance? Update VolumeAttributesClass dynamically
$ kubectl patch volumeattributesclass high-performance --type='merge' -p='{"parameters":{"iops":"20000","throughput":"1000"}}'
volumeattributesclass.storage.k8s.io/high-performance patched

# Apply new attributes to existing PVC (NO RECREATION NEEDED)
$ kubectl patch pvc dynamic-storage-v134 --type='merge' -p='{"spec":{"volumeAttributesClassName":"high-performance"}}'
persistentvolumeclaim/dynamic-storage-v134 patched

# Check volume modification in progress
$ kubectl describe pvc dynamic-storage-v134
Events:
  Normal  VolumeAttributesModification  30s  volume-modifier  Modifying volume attributes
  Normal  VolumeAttributesModified      15s  volume-modifier  Volume attributes successfully modified

# Verify new IOPS without downtime
$ aws ec2 describe-volumes --volume-ids vol-xyz789abc123def456
{
    "Volumes": [{
        "VolumeId": "vol-xyz789abc123def456",
        "Iops": 20000,          # Updated dynamically!
        "Throughput": 1000,     # Updated dynamically!
        "ModifyingProgress": 100 # Modification complete
    }]
}

# Pod continues running with improved performance
$ kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
app-pod   1/1     Running   0          5m    # No restart needed!

# Performance improvement visible immediately
$ kubectl exec app-pod -- fio --name=test --rw=randwrite --bs=4k --numjobs=1 --time_based --runtime=30s
write: IOPS=18.5k    # Higher IOPS achieved without downtime
```
