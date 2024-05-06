---
title: Service accounts
date: 2024-05-06
categories: [k8s, service accounts]
tags: [sa,service accounts]     # TAG names should always be lowercase
---

## Beskrivning

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
