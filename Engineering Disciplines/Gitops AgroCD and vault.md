# 🔐 GitOps + Argo CD + Vault: Secure Secrets Delivery Tutorial

## 🧭 Table of Contents

1. Overview
2. Why Use Vault with GitOps?
3. Architecture: GitOps + Argo CD + Vault
4. Tools and Technologies
5. Secrets Management Strategies
6. Example: Delivering Secrets via Vault to Kubernetes using GitOps
7. Argo CD and External Secrets Operator Integration
8. Vault Agent Injector (Alternative Approach)
9. Best Practices
10. Resources for Further Learning

---

## 📌 Overview

In GitOps workflows, **everything is stored in Git, including manifests** — but this raises a question:  
**How do you manage secrets securely in GitOps?**

This tutorial walks you through integrating:

- **GitOps (Argo CD)** for deployments
- **HashiCorp Vault** for secret storage
- **External Secrets Operator** or **Vault Agent Injector** for delivery
- Kubernetes for workloads

---

## ❓ Why Use Vault with GitOps?

Storing secrets directly in Git is **insecure**, even if the repo is private.

Using Vault with GitOps allows you to:

- **Keep secrets out of Git**
- **Dynamically inject secrets** into Kubernetes via sidecars or CRDs
- **Rotate and manage secrets lifecycle**
- **Audit** all access and usage
- **Control access** via policies

---

## 🏗️ Architecture: GitOps + Argo CD + Vault

Here's how the flow works:

1. Developer defines an application manifest (Deployment, SecretStore, etc.) in Git
2. Argo CD syncs the manifests into the Kubernetes cluster
3. External Secrets Operator fetches secrets from Vault based on the manifest
4. Secrets are injected into the application as Kubernetes Secrets or environment variables
```
Git (manifests) → Argo CD → Kubernetes CRDs → External Secrets Operator → Vault → Kubernetes Secrets → Application
```
---

## 🧰 Tools and Technologies

| Tool                     | Purpose                                           |
|--------------------------|---------------------------------------------------|
| Argo CD                  | GitOps-based application delivery                |
| HashiCorp Vault          | Centralized secrets management                   |
| External Secrets Operator| Bridge between Kubernetes and external secret stores |
| Vault Agent Injector     | Inject secrets directly into pods as env vars or files |
| Kubernetes               | Application runtime                              |
| Git                      | Source of truth for manifests                    |

---

## 🔐 Secrets Management Strategies

There are two primary ways to deliver secrets securely using Vault in a GitOps workflow:

### 1. External Secrets Operator (ESO)

- Kubernetes CRDs define what secrets to fetch from Vault
- Secrets are created as native Kubernetes Secrets
- Git contains only **metadata**, not the actual secret values

### 2. Vault Agent Injector

- Vault Agent sidecar injects secrets directly into pod memory or filesystem
- No Kubernetes Secret object is created
- Useful for **short-lived or highly sensitive secrets**

---

## 🛠️ Example: Secure Secrets Delivery with Argo CD + Vault + External Secrets Operator

### Goal:
Inject a secret from Vault into a Kubernetes application using GitOps.

### Prerequisites:

- Kubernetes cluster
- Argo CD installed
- Vault installed and reachable from Kubernetes
- External Secrets Operator (ESO) installed

### Step-by-Step:

#### 1. Enable Kubernetes Auth in Vault

```bash
vault auth enable kubernetes

vault write auth/kubernetes/config \
  token_reviewer_jwt="<JWT>" \
  kubernetes_host="<K8S_API_SERVER>" \
  kubernetes_ca_cert="@ca.crt"
```

#### 2. Create a Vault Policy

```bash
vault policy write my-app-policy - <<EOF
path "secret/data/my-app/*" {
  capabilities = ["read"]
}
EOF
```

#### 3. Create a Kubernetes ServiceAccount
```bash
kubectl create sa my-app-sa
```

#### 4. Map Vault Role to Kubernetes SA

```bash
vault write auth/kubernetes/role/my-app \
  bound_service_account_names=my-app-sa \
  bound_service_account_namespaces=default \
  policies=my-app-policy \
  ttl=1h
```

#### 5. Store a Secret in Vault
```bash
vault kv put secret/my-app/db-creds username=admin password=supersecret
```

#### 6. Define SecretStore and ExternalSecret in Git

Store the following YAMLs in your Git repo:

**secretstore.yaml**

```yaml
apiVersion: external-secrets.io/v1beta1  
kind: SecretStore  
metadata:  
  name: vault-backend  
spec:  
  provider:  
    vault:  
      server: https://vault.default.svc:8200  
      path: secret  
      version: v2  
      auth:  
        kubernetes:  
          mountPath: kubernetes  
          role: my-app
```

**externalsecret.yaml**

```yaml
apiVersion: external-secrets.io/v1beta1  
kind: ExternalSecret  
metadata:  
  name: my-app-secret  
spec:  
  refreshInterval: 1h  
  secretStoreRef:  
    name: vault-backend  
    kind: SecretStore  
  target:  
    name: my-app-k8s-secret  
    creationPolicy: Owner  
  data:  
    - secretKey: username  
      remoteRef:  
        key: my-app/db-creds  
        property: username  
    - secretKey: password  
      remoteRef:  
        key: my-app/db-creds  
        property: password
```

#### 7. Argo CD Syncs the Manifests

- Argo CD detects the new `SecretStore` and `ExternalSecret` resources
- ESO fetches the data from Vault and creates a Kubernetes Secret: `my-app-k8s-secret`

#### 8. Use the Secret in Your Deployment

In your `deployment.yaml`, reference the secret like this:

```yaml
envFrom:
  - secretRef:
      name: my-app-k8s-secret
```
---

## 🧪 Vault Agent Injector (Alternative Method)

If you don’t want Kubernetes Secrets to exist at all, use **Vault Agent Injector**.

### Highlights:

- Injects secrets into pods via sidecar container
- Secrets can be written to volume or exposed as env vars
- No secret CRDs or Kubernetes Secrets

### Requirements:

- Vault Agent Injector must be installed in the cluster
- Vault server accessible from pods

### Deployment Annotation Example:

```
metadata:
  annotations:
    vault.hashicorp.com/agent-inject: "true"
    vault.hashicorp.com/role: "my-app"
    vault.hashicorp.com/agent-inject-secret-db: "secret/data/my-app/db-creds"
    vault.hashicorp.com/agent-inject-template-db: |
      {{- with secret "secret/data/my-app/db-creds" -}}
      export DB_USER={{ .Data.data.username }}
      export DB_PASS={{ .Data.data.password }}
      {{- end }}
```

---

## 📋 Best Practices

- Never store secrets in Git, even temporarily
- Use External Secrets Operator for flexibility and GitOps compatibility
- Use Vault Agent Injector for highly sensitive, ephemeral secrets
- Use short-lived tokens and rotate secrets regularly
- Use RBAC and Vault policies to restrict access
- Monitor and audit secret usage with Vault telemetry

---

## 📚 Resources for Further Learning

- https://developer.hashicorp.com/vault/docs
- https://external-secrets.io
- https://argo-cd.readthedocs.io
- https://learn.hashicorp.com/collections/vault/kubernetes
- https://github.com/hashicorp/vault-k8s
- https://github.com/external-secrets/external-secrets

---

## 🧠 Summary

Combining GitOps with Vault provides a secure, auditable, and automated way to manage secrets in Kubernetes. Whether you use the External Secrets Operator or Vault Agent Injector, you can ensure secrets never touch Git and are delivered just-in-time to your workloads.

> GitOps + Argo CD + Vault = Secure, Declarative, Automated Secrets Management

---