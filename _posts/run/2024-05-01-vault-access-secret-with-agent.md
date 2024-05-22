---
title: Access a secret with Agent Injector
date: 2024-05-21
categories: [k8s,Configure]
tags: [k8s,configure,vault]     # TAG names should always be lowercase
---

## Vault Agent Injector

You can access a secret from k8s with Vault Agent Injector.

Vault Agent Injector service configured to target an external Vault. The injector service enables the authentication and secret retrieval for the applications, by adding Vault Agent containers as they are written to the pod automatically it includes specific annotations.

### Configure in Vault

Create a role that lets the k8s sa "my-dev-apps-sa" use the policy "my-dev-apps-secret-policy"

One role for the dev namespace

```shell
vault write auth/kubernetes/role/my-dev-apps-role \
   bound_service_account_names=my-dev-apps-sa `: the serviceaccount to use in the pod` \
   bound_service_account_namespaces='dev' `: the namespace that the pod uses` \
   policies=my-dev-apps-secret-policy `: the acl policy for the secret` \
   ttl=168h
```

And one role for the prod namespace

```shell
vault write auth/kubernetes/role/my-prod-apps-role \
   bound_service_account_names=my-prod-apps-sa \
   bound_service_account_namespaces='prod' \
   policies=my-prod-apps-secret-policy \
   ttl=168h
```

The above maps our Kubernetes service account, used by our pod, to a policy.

### Inject secrets into the pod

The Vault Agent Injector only modifies a pod or deployment if it contains a specific set of annotations.

Create two pods. One for dev

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-dev-app
  labels:
    app: my-dev-app
  annotations:
    vault.hashicorp.com/agent-inject: 'true' # enables the Vault Agent Injector service
    vault.hashicorp.com/role: 'my-dev-apps-role' # Vault Kubernetes authentication role
    vault.hashicorp.com/agent-inject-secret-credentials.txt: 'secret/data/my-dev-apps-secrets/mariadb' # the path to the secret
spec:
  serviceAccountName: my-dev-apps-sa
  containers:
    - name: my-dev-app
      image: burtlo/devwebapp-ruby:k8s
```

and one for prod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-prod-app
  labels:
    app: my-prod-app
  annotations:
    vault.hashicorp.com/agent-inject: 'true' # enables the Vault Agent Injector service
    vault.hashicorp.com/role: 'my-prod-apps-role' # Vault Kubernetes authentication role
    vault.hashicorp.com/agent-inject-secret-credentials.txt: 'secret/data/my-prod-apps-secrets/mariadb' # the path to the secret
spec:
  serviceAccountName: my-prod-apps-sa
  containers:
    - name: my-prod-app
      image: burtlo/devwebapp-ruby:k8s
```

### deploy both pods above in different namespaces

First som prereqs

```bash
kubectl create ns dev
kubectl create ns prod

kubectl -n dev create sa my-dev-apps-sa
kubectl -n prod create sa my-prod-apps-sa
```

Apply and run your pods

```bash
kubectl -n dev apply -f my-dev-app.yml
kubectl -n prod apply -f my-prod-app.yml
```

### Verify

```bash
kubectl get all -n dev
kubectl get all -n prod
```

If you get a problem like

```plaintext
NAME             READY   STATUS     RESTARTS   AGE
pod/my-dev-app   0/2     Init:0/1   0          2m51s
```

check out the error with

```bash
kubectl -n dev logs my-dev-app -c vault-agent-init
```

Finaly, read the secret from the pod with

```bash
kubectl -n dev exec my-dev-app -- cat /vault/secrets/credentials.txt
kubectl -n prod exec my-prod-app -- cat /vault/secrets/credentials.txt
```
