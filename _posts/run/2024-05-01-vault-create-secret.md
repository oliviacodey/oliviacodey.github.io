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

### Create a secret via the GUI or CLI

```bash
vault kv put secret/my-apps-secrets/mariadb username='giraffe' password='salsa'
vault kv put secret/my-apps-secrets/mysql username='elephant' password='tacos'
```

View the new secret via the browser or:

```bash
vault kv get secret/my-apps-secrets/mariadb
vault kv get secret/my-apps-secrets/mysql
```

### Create a (access) policy to that secret

```plaintext '/vault/config.d/my-apps-secret-policy.hcl'
path "secret/data/my-apps-secrets/*" {
  capabilities = ["read"]
}
```

```bash
vault policy write my-apps-secret-policy /vault/config.d/my-apps-secret-policy.hcl
vault policy read my-apps-secret-policy
```

or

```bash
vault policy write my-apps-secret-policy - <<EOF
path "secret/data/my-apps-secrets/*" {
  capabilities = ["read"]
}
EOF
```

### Create a token to access the secret

vault token create -policy my-apps-secret-policy

Try out the token

curl --header "X-Vault-Token: the-token-goes-here" http://vault:8200/v1/secret/data/my-apps-secrets/mariadb | jq -r .data

## Access the secret via a ServiceAccount

Create a role that lets the k8s sa "my-apps-sa" use the policy "my-apps-secret-policy"

```shell
vault write auth/kubernetes/role/my-apps-role \ `# the role name to use in the pod`
   bound_service_account_names=my-apps-sa \ `# the serviceaccount to use in the pod`
   bound_service_account_namespaces='*' \ `# the namespace that the pod uses`
   policies=my-apps-secret-policy \ `# the acl policy for the secret`
   ttl=168h
```

Use the same policy for a different pod in another namespace

``` bash
vault write auth/kubernetes/role/my-app2s-role \
   bound_service_account_names=my-app2s-sa \
   bound_service_account_namespaces='*' \
   policies=my-apps-secret-policy \
   ttl=168h
```

The above maps our Kubernetes service account, used by our pod, to a policy.

## Vault Agent

Vault Agent Injector service configured to target an external Vault. The injector service enables the authentication and secret retrieval for the applications, by adding Vault Agent containers as they are written to the pod automatically it includes specific annotations.

### Inject secrets into the pod

The Vault Agent Injector only modifies a pod or deployment if it contains a specific set of annotations.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
  labels:
    app: my-app
  annotations:
    vault.hashicorp.com/agent-inject: 'true' # enables the Vault Agent Injector service
    vault.hashicorp.com/role: 'devweb-app' # Vault Kubernetes authentication role
    vault.hashicorp.com/agent-inject-secret-credentials.txt: 'secret/data/devwebapp/config' # the path to the secret
spec:
  serviceAccountName: internal-app
  containers:
    - name: app
      image: burtlo/devwebapp-ruby:k8s
```

``` bash
kubectl apply -f my-pod-with-a-secret-from-vault.yaml
```
