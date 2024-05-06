# ServiceAccount

"Service accounts are identities that are intended for use by applications instead of people."

command line: no

Monterad under /var/run/secrets/kubernetes.io/serviceaccount/token

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: build-robot
```

```bash
kubectl create token build-robot
```
