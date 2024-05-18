---
title: Access vault secrets
date: 2024-05-17
categories: [k8s,Configure]
tags: [k8s,configure,vault]     # TAG names should always be lowercase
---

## Use a token to access the secret

You can use a token to access the secret directly from anything that has acceess to the vault. In this example we will create a new token and use it from within a container to get the secret.

### Create the token

Create a token and point it to a (access) poilcy

```bash
vault token create -policy=my-apps-secret-policy
```


### Create a simple container that uses the token in clear text

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: devwebapp
  labels:
    app: devwebapp
spec:
  serviceAccountName: internal-app
  containers:
    - name: app
      image: burtlo/devwebapp-ruby:k8s
      env:
      - name: VAULT_ADDR
        value: "http://vault-server:8200"
      - name: VAULT_TOKEN
        value: root
```
