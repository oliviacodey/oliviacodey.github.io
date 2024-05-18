---
title: Create secret in Vault
date: 2024-05-09
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

## Enter the Vault

Start the shell in vault

```bash
podman logs vault # shows the default root token
podman exec -it vault /bin/sh

```

Create the vault

```bash
vault operator init
```

Unseal the vault

```bash
vault operator unseal # Uses Shamir's secret sharing
```

Login to the vault

```bash
export VAULT_ADDR='http://127.0.0.1:8200'
vault login "token here"
```

## Basic Secret

### Create a secret via the GUI or CLI

```bash
vault kv put secret/my-apps-secrets/mariadb username='giraffe' password='salsa'
```

View the new secret via the browser or:

```bash
vault kv get secret/my-apps-secrets/mariadb
```

### Create a (access) policy to that secret

```plaintext '/home/vault/my-apps-secret-policy.hcl'
path "secret/data/my-apps-secrets/*" {
  capabilities = [“read”]
}
```

```bash
vault policy write my-apps-secret-policy /home/vault/my-apps-secret-policy.hcl
```

or

```bash
vault policy write my-apps-secret-policy - <<EOF
path "secret/data/my-apps-secrets/*" {
  capabilities = [“read”]
}
EOF
```

### Create a token to access the secret

vault token create -policy my-apps-secret-policy

Try out the token

curl --header "X-Vault-Token: 'the toke goes here'" http://vault:8200/v1/secret/data/my-apps-secrets/mariadb

## Access the secret via a ServiceAccount

Create a role that lets the k8s sa "my-apps-sa" use the policy "my-apps-secret-policy"

```shell
vault write auth/kubernetes/role/my-apps-role \
   bound_service_account_names=my-apps-sa \
   bound_service_account_namespaces=my-app \
   policies=my-apps-secret-policy \
   ttl=168h
```

The above maps our Kubernetes service account, used by our pod, to a policy.

## Vault Agent

Vault Agent Injector service configured to target an external Vault. The injector service enables the authentication and secret retrieval for the applications, by adding Vault Agent containers as they are written to the pod automatically it includes specific annotations.

```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update
helm install vault hashicorp/vault \
    --set "global.externalVaultAddr=http://vault:8200"

kubectl get pods
kubectl describe serviceaccount vault
```

Create a token

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: vault-token-g955r
  annotations:
    kubernetes.io/service-account.name: vault
type: kubernetes.io/service-account-token
```

```bash
kubectl apply -f vault-secret.yaml
```

Create a variable named VAULT_HELM_SECRET_NAME that stores the secret name.

```bash
VAULT_HELM_SECRET_NAME=$(kubectl get secrets --output=json | jq -r '.items[].metadata | select(.name|startswith("vault-token-")).name')
```

This command filters the secrets by those that start with vault-token- and returns the name of token

```bash
kubectl describe secret $VAULT_HELM_SECRET_NAME
```

### Configure Kubernetes authentication

Vault provides a Kubernetes authentication method that enables clients to authenticate with a Kubernetes Service Account Token.

```bash
export VAULT_ADDR='http://127.0.0.1:8200'
vault login
vault token lookup
vault auth enable kubernetes
```

On the k8s cluster

```bash
TOKEN_REVIEW_JWT=$(kubectl get secret $VAULT_HELM_SECRET_NAME --output='go-template={{ .data.token }}' | base64 --decode)
KUBE_CA_CERT=$(kubectl config view --raw --minify --flatten --output='jsonpath={.clusters[].cluster.certificate-authority-data}' | base64 --decode)
KUBE_HOST=$(kubectl config view --raw --minify --flatten --output='jsonpath={.clusters[].cluster.server}') # or export KUBE_HOST="https://k8s-server-name:6443"
```

> Get the cluster issuer by running: kubectl get --raw /.well-known/openid-configuration | jq -r .issuer
{: .prompt-tip }

Move these values to the Vault server and run

```bash
```

This way didnt work. The only ways was by uploading the ca-cert in the gui.

```bash
vault write auth/kubernetes/config \
     token_reviewer_jwt="$TOKEN_REVIEW_JWT" \
     kubernetes_host="$KUBE_HOST" \
     kubernetes_ca_cert="/kube_ca.pem" \
     tls_skip_verify="true"
```

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

kubectl apply -f pod-devwebapp-with-annotations.yaml
