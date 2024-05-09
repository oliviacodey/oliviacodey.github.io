---
title: Example deployment
date: 2024-05-06
categories: [k8s,Run]
tags: [k8s,run,deployment]     # TAG names should always be lowercase
---

## Deployment

A Deployment provides declarative updates for Pods and ReplicaSets.
You describe a desired state in a Deployment, and the Deployment Controller changes the actual state to the desired state at a controlled rate.
The Deployment creates a ReplicaSet that creates three replicated Pods, indicated by the .spec.replicas field.

## Selectors

Unlike names and UIDs, labels do not provide uniqueness. In general, we expect many objects to carry the same label(s).
The .spec.selector field defines how the created ReplicaSet finds which Pods to manage

## Example

```yaml
---
apiVersion: apps/v1
kind: Deployment # what to create?
metadata:
  name: example-deployment # Name of the Deplyment
spec: # specification for deployment resource
  replicas: 1 # how many replicas of pods we want to create
  selector:
    matchLabels:
      app: mariadb # how the created ReplicaSet finds which Pods to manage (easy to forget)
  template: # blueprint for pods
    metadata:
      labels:
        app: mariadb # service will look for this label
    spec: # specification for Pods
      containers:
      - name: mariadb # Create one container and name it mariadb
        image: mariadb # Get the image from dockerhub
        ports:
        - containerPort: 3306 
        env:
        - name: MARIADB_DATABASE # Setting this value will create a empty database
          value: my_wiki
        - name: MARIADB_ROOT_PASSWORD # Setting this value will set the root password
          # value: secret (you can just specify the secret here)
          valueFrom: # or you can get it from
            secretKeyRef: # a pre-defined reference
              name: mariadb-secret # open the secret named "mariadb-secret"
              key: mariadb-root-password # get the secret stored in the key "mariadb-root-password"
        volumeMounts:
        - name: mariadb-pv # mount the volume named "mariadb-pv"
          mountPath: /var/lib/mysql
      volumes: # create a volume for all the containers in this deployment (normaly just the one)
      - name: mariadb-pv # name it for later referens
        persistentVolumeClaim:
          claimName: mariadb-pvc-1g # Point to the PVC
```
