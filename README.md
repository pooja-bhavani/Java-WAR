```bash
$ kubectl apply -f config-v133.yaml
configmap/config created

$ kubectl get configmap config -o yaml
apiVersion: v1
data:
  country: "false"        # NO became boolean false!
  enabled: "true"         # yes became boolean true!
  port: "8080"           # 8080something truncated
  version: "1.23"        # 01.23 became 1.23 (lost leading zero)
kind: ConfigMap

# Application reads wrong values
$ kubectl exec test-pod -- env | grep CONFIG
CONFIG_COUNTRY=false     # Should be "NO" (Norway)
CONFIG_ENABLED=true      # Should be "yes" 
CONFIG_VERSION=1.23      # Should be "01.23"

# Debugging nightmare - values silently changed
$ echo "Application fails because country=false instead of 'NO'"
$ echo "Version comparison breaks: '1.23' != '01.23'"
```

```bash
$ KUBECTL_KYAML=true kubectl apply -f config-v134.yaml
configmap/config created

$ kubectl get configmap config -o yaml
apiVersion: v1
data:
  country: "NO"          # Preserved as string
  enabled: "yes"         # Preserved as string  
  port: "8080"          # Preserved as string
  version: "01.23"      # Leading zero preserved
kind: ConfigMap

# Application reads correct values
$ kubectl exec test-pod -- env | grep CONFIG
CONFIG_COUNTRY=NO        # Correct country code
CONFIG_ENABLED=yes       # Correct string value
CONFIG_VERSION=01.23     # Leading zero preserved

# KYAML validation catches problems early
$ cat problematic.yaml
apiVersion: v1
kind: ConfigMap
data:
  country: NO            # Unquoted - KYAML will warn
  
$ KUBECTL_KYAML=true kubectl apply -f problematic.yaml --dry-run=client
Warning: KYAML: Boolean-like strings should be quoted
Warning: KYAML: Value 'NO' may be interpreted as boolean, consider quoting

# Safe application behavior
$ echo "Application works correctly with preserved string values"
$ echo "No silent type coercion surprises"
```
