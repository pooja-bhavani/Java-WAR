# Java-WAR
```bash
$ kubectl apply -f private-image-pod-v134.yaml
serviceaccount/image-puller created
pod/private-image-pod-v134 created

$ kubectl get serviceaccounts
NAME           SECRETS   AGE
image-puller   0         30s    # No static secrets!

$ kubectl describe pod private-image-pod-v134
Service Account:  image-puller
Volumes:
  kube-api-access-xyz:
    Type:                    Projected (a volume that contains injected data)
    TokenExpirationSeconds:  3607    # Token expires in ~1 hour
    ConfigMapName:           kube-root-ca.crt
    DownwardAPI:             true

# Check token rotation
$ kubectl exec private-image-pod-v134 -- cat /var/run/secrets/kubernetes.io/serviceaccount/token | cut -d. -f2 | base64 -d | jq .exp
1705320000   # Expiration timestamp - auto-rotated

$ kubectl get events --field-selector involvedObject.name=private-image-pod-v134
LAST SEEN   TYPE     REASON      OBJECT                    MESSAGE
2m          Normal   Pulling     pod/private-image-pod-v134   Pulling image using ServiceAccount token
2m          Normal   Pulled      pod/private-image-pod-v134   Successfully pulled image with rotated token
```
