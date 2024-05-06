---
title: Quick examples
date: 2024-05-06
categories: [K8S, quick]
tags: [quick,commands]     # TAG names should always be lowercase
---

# Run a quick simple pod

## Create a quick pod

```bash
kubectl run simple-pod --image=nginx --port 80
kubectl get pods/simple-pod -o wide
curl 10.42.0.24
```

## Expose the pod

```bash
kubectl expose pod simple-pod --port 80 --type=LoadBalancer --name=simple-service
kubectl get service simple-service -o wide
curl 192.168.148.22
```
