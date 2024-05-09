---
title: ConfigMap
date: 2024-05-06
categories: [k8s,api]
tags: [k8s,api,configmap]
---

## ConfigMaps
A ConfigMap is an API object used to store non-confidential data in key-value pairs. Pods can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume.

## Why
A easyer way to change something in a deployment without having to apply the whole deployment again.
Can be used for mounting a file in a pod.


```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo
data:
  player_initial_lives: "3" # classic key/value pair
```

## Create a configmap from file

```bash
kubectl create configmap mediawiki --from-file ./LocalSettings.php
```

## Mounta en configmap i en pod

```yaml
  volumeMounts:
    - name: config-volume
      mountPath: /var/www/html/LocalSettings.php
      subPath: LocalSettings.php # the key in the configmap
volumes:
- name: config-volume
  configMap:
    name: mediawiki
```
