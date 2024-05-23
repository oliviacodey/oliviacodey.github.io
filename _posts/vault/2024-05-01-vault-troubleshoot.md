---
title: Vault troubleshoot
date: 2024-05-17
categories: [Vault,Configure]
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
curl --header "X-Vault-Token: 'the token here'" http://vault:8200/v1/secret/data/my-apps-secrets/mariadb
```

Run the debugger for 5 minutes

```bash
vault debug -interval=1m -duration=5m -compress=0
```

You can try to manually access the Kubernetes API (in your app cluster) from your Vault cluster with the same configuration you used to setup Vault.

```bash
curl -X "POST" "${K8S_HOST}/apis/authentication.k8s.io/v1/tokenreviews" \
     --cacert <(echo $VAULT_SA_CA_CRT)
     -H 'Authorization: Bearer ${TR_ACCOUNT_TOKEN}' \
     -H 'Content-Type: application/json; charset=utf-8' \
     -d $'{
  "kind": "TokenReview",
  "apiVersion": "authentication.k8s.io/v1",
  "spec": {
    "token": "${INTERNAL_APP_TOKEN}"
  }
}'
```
