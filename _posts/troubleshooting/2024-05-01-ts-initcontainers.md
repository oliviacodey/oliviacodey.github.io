---
title: InitContainers
date: 2024-05-11
categories: [k8s, Troubleshooting]
tags: [troubleshooting,initcontainers]     # TAG names should always be lowercase
---

## Checking the status of Init Containers

For example, a status of Init:1/2 indicates that one of two Init Containers has completed successfully

```bash
kubectl get pods
```

NAME         READY     STATUS     RESTARTS   AGE
pod-name   0/1       Init:1/2   0          7s

```bash
kubectl get pod nginx --template '{{.status.initContainerStatuses}}'
```

## Accessing logs from Init Containers

```bash
kubectl logs pod-name -c <init-container-2>
```
