---
title: Install Vault
date: 2024-05-20
categories: [Vault,Install]
tags: [k8s,install,vault]     # TAG names should always be lowercase
---

## Vault by HashiCorp

Vault secures, stores, and tightly controls access to tokens, passwords, certificates, API keys, and other secrets critical in modern computing.

## Install

```bash
podman run -d --cap-add=IPC_LOCK -e 'VAULT_LOCAL_CONFIG={"storage": {"file": {"path": "/vault/file"}}, "listener": [{"tcp": { "address": "0.0.0.0:8200", "tls_disable": true}}], "default_lease_ttl": "168h", "max_lease_ttl": "720h", "ui": true}' -p 8200:8200 hashicorp/vault server

podman run -d --rm --cap-add=IPC_LOCK --label -e 'VAULT_LOCAL_CONFIG={"storage": {"file": {"path": "/vault/file"}}, "listener": [{"tcp": { "address": "0.0.0.0:8200", "tls_disable": true}}], "default_lease_ttl": "168h", "max_lease_ttl": "720h", "ui": true}' \
 "io.containers.autoupdate=registry" \
  --name vault --net=mac-vlan --hostname=vault.domain.com --ip 192.168.0.37 \
  -v vault-config.d:/vault/config.d -v vault-file:/vault/file -v vault-config:/vault/config -p 8200:8200 docker.io/hashicorp/vault:latest server
```

## Enter the Vault for the first time

Start the shell in vault and run the folowing command to initialize a new vault database.

```bash
podman exec -it vault /bin/sh
export VAULT_ADDR='http://127.0.0.1:8200'
vault operator init
```

Write down the five Unseal Keys and the root token

Unseal the vault

```bash
export VAULT_ADDR='http://127.0.0.1:8200'
vault operator unseal # Uses Shamir's secret sharing 3/5
vault operator unseal
vault operator unseal
```

## Enter the Vault the rest of the times

```bash
podman logs vault |grep Token # shows the default root token
podman exec -it vault /bin/sh

```

Login to the vault

```bash
export VAULT_ADDR='http://127.0.0.1:8200'
vault login "token here"
vault token lookup
```

## Configure Kubernetes authentication

Vault provides a Kubernetes authentication method that enables clients to authenticate with a Kubernetes Service Account Token.

### On the k8s cluster

Jump in to the k8s cluster and do the following and install the Vault Agent Injector service

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

Get the token and the ca cert from k8s to use in the vault config

```bash
TOKEN_REVIEW_JWT=$(kubectl get secret vault-token-g955r --output='go-template={{ .data.token }}' | base64 --decode)
KUBE_CA_CERT=$(kubectl config view --raw --minify --flatten --output='jsonpath={.clusters[].cluster.certificate-authority-data}' | base64 --decode)
KUBE_HOST=$(kubectl config view --raw --minify --flatten --output='jsonpath={.clusters[].cluster.server}') # or export KUBE_HOST="https://kube:6443"
```

> Get the cluster issuer by running: kubectl get --raw /.well-known/openid-configuration | jq -r .issuer
{: .prompt-tip }

Move these values to the Vault server and put them in the container volume /vault/config.d

Run the following

```bash
vault auth enable kubernetes
```

```bash
vault write auth/kubernetes/config \
     token_reviewer_jwt=@/vault/config.d/jwt-token \
     kubernetes_host="$KUBE_HOST" \
     kubernetes_ca_cert=@/vault/config.d/kube-ca.pem \
     tls_skip_verify="true" \
     disable_local_ca_jwt="true"
```

Or if you have them in enviroment variables

```bash
vault write auth/kubernetes/config \
     token_reviewer_jwt="$TOKEN_REVIEW_JWT" \
     kubernetes_host="$KUBE_HOST" \
     kubernetes_ca_cert="$KUBE_CA_CERT" \
     tls_skip_verify="true" \
     disable_local_ca_jwt="true" \
     issuer="\"https://kubernetes.default.svc.cluster.local\""
```

Check out your config

```bash
vault read auth/kubernetes/config
```
