---
title: Tokens
date: 2024-05-18
categories: [k8s,Configure]
tags: [k8s,configure,vault]     # TAG names should always be lowercase
---

## What is a token

Batch tokens are encrypted binary large objects (blobs) that carry enough information to perform Vault actions. Therefore, batch tokens are extremely lightweight and scalable; however, they lack most of the flexibility and features of service tokens.

Service tokens:	hvs.
Batch tokens:	hvb.

### Service token lifecycle

Every non-root token has a time-to-live (TTL). When a token expires, Vault automatically revokes it. If you create a new token, the token you used to create the token becomes the parent token. Once the parent token expires, so do all its children regardless of their own TTLs

## Token role

Instead of passing a number of parameters, you can create a role with a set of parameter values set.

### Create a role

```bash
vault write auth/token/roles/my-role \
    allowed_policies="my-apps-secret-policy" \
    orphan=true \
    period=31d
```

### Create a toke for my-role

```bash
vault token create -role=my-role
```

list all tokens

```bash
vault list auth/token/accessors

vault token lookup -accessor 7Y4nF5HnzMXalEaUyNz4TA8y
```

### Create the token

Create a token and point it to a (access) poilcy



```bash
vault token create -policy=my-apps-secret-policy
```

Try out the new token (the GUI done work here)

```bash
curl --header "X-Vault-Token: hvs.laksdflkasj" http://vault:8200/v1/secret/data/my-apps-secrets/mariadb
```
