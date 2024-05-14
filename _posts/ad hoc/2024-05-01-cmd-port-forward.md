---
title: Portforward
date: 2024-05-06
categories: [k8s, ad hoc]
tags: [quick,commands,ad hoc,portforward]     # TAG names should always be lowercase
---

## Forward a local port to a port on the Pod

kubectl port-forward allows using resource name, such as a pod name, to select a matching pod to port forward to

```bash
kubectl port-forward pods/mongo-75f59d57f4-4nd6q 28015:27017
kubectl port-forward deployment/mongo 28015:27017
kubectl port-forward service/mongo 28015:27017
```
