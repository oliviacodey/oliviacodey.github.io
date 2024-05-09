---
title: Expose the pod
date: 2024-05-06
categories: [k8s, ad hoc]
tags: [quick,commands,ad hoc]     # TAG names should always be lowercase
---

```bash
kubectl expose pod simple-pod --port 80 --type=LoadBalancer --name=simple-service
kubectl get service simple-service -o wide
curl 192.168.0.22
```
