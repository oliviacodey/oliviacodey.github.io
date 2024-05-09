---
title: Service accounts
date: 2024-05-06
categories: [k8s,api]
tags: [sa,service accounts,api]     # TAG names should always be lowercase
---

## What

Service accounts are identities that are intended for use by applications instead of people.
Kubernetes offers two distinct ways for clients that run within your cluster to authenticate to the API server

|command line|no|

## Where

Mounted under /var/run/secrets/kubernetes.io/serviceaccount/token

## Use

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: build-robot
```

```bash
kubectl create token build-robot
```
