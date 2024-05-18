---
title: Vault troubleshoot
date: 2024-05-17
categories: [k8s,Configure]
tags: [k8s,configure,vault,troubleshoot]     # TAG names should always be lowercase
---

## Get my token

```bash
vault token lookup
```

## List tokens

The only way to "list tokens" is via the auth/token/accessors command, which actually gives a list of token accessors. While this is still a dangerous endpoint (since listing all of the accessors means that they can then be used to revoke all tokens), it also provides a way to audit and revoke the currently-active set of tokens

```bash
vault list auth/token/accessors

vault token lookup -accessor 7Y4nF5HnzMXalEaUyNz4TA8y
```

## Curl the secret with a token

```bash
curl --header "X-Vault-Token: hvs.uRRJCwxYgZOz7GP0fjTbMnZT" http://vault:8200/v1/secret/data/my-apps-secrets/mariadb
```
