---
title: ConfigMaps
date: 2024-05-06
categories: [k8s,api]
tags: [k8s,api,ConfigMaps]     # TAG names should always be lowercase
---

## Skapa en configmap från fil

kubectl create configmap mediawiki --from-file /home/micke/LocalSettings.php

## Mounta en configmap i en pod

```yaml
  volumeMounts:
    - name: config-volume
      mountPath: /var/www/html/LocalSettings.php
      subPath: LocalSettings.php #värdet i configmapen
volumes:
- name: config-volume
  configMap:
    name: mediawiki
```
