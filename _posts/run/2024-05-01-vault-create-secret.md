---
title: Create secret in Vault
date: 2024-05-19
categories: [k8s,Configure]
tags: [k8s,configure,vault]     # TAG names should always be lowercase
---

## How to use

Vault can be accessed via these different methods

* Application aware
  * Application can use JSON Web Token (JWT) to access the vault directly
* Application unaware
  * Vault agent init container
      Deployment change required. Secret rotation not supported. Config templating from secrets. Does not require helm.
  * Vault agent sidecar secret injection
      Deployment change NOT required. The deployment can be patched to use vault sidecar. Secret rotation supported. Config templating from secrets. Requires helm.
  * Mount Vault secrets to CSI Volume
      Deployment change required. Secret rotation not supported. Config templating from secrets NOT SUPPORTED. Requires helm as well as CSI driver and vault CSI provider.

## Basic Secret

### Create a couple of secrets via the GUI or CLI

```bash
vault kv put secret/my-dev-apps-secrets/mariadb username='giraffe' password='salsa'
vault kv put secret/my-prod-apps-secrets/mariadb username='elephant' password='tacos'
```

View the new secrets via the browser or:

```bash
vault kv get secret/my-dev-apps-secrets/mariadb
vault kv get secret/my-prod-apps-secrets/mariadb
```

### Create a (access) policy to those secret

```plaintext title="/vault/config.d/my-dev-apps-secrets-policy.hcl"
path "secret/data/my-dev-apps-secrets/*" {
  capabilities = ["read"]
}
```

```plaintext title="/vault/config.d/my-prod-apps-secrets-policy.hcl"
path "secret/data/my-prod-apps-secrets/*" {
  capabilities = ["read"]
}
```

Write the hcl to a policy

```bash
vault policy write my-dev-apps-secret-policy /vault/config.d/my-dev-apps-secrets-policy.hcl
vault policy write my-prod-apps-secret-policy /vault/config.d/my-prod-apps-secrets-policy.hcl
vault policy read my-dev-apps-secret-policy
vault policy read my-prod-apps-secret-policy
```

or the ad-hoc way

```bash
vault policy write my-apps-secret-policy - <<EOF
path "secret/data/my-apps-secrets/*" {
  capabilities = ["read"]
}
EOF
```

### Create a token to access the secret

```bash
vault token create -policy my-dev-apps-secret-policy
vault token create -policy my-prod-apps-secret-policy
```

Try out the token

```bash
curl --header "X-Vault-Token: the-token-goes-here" http://vault:8200/v1/secret/data/my-dev-apps-secrets/mariadb | jq -r .data
```
