# 🎯 Argo CD In-Depth Tutorial

## 🧭 Table of Contents

1. What is Argo CD?
2. Why Use Argo CD?
3. Core Concepts and Architecture
4. Argo CD Installation
5. Managing Applications with Argo CD
6. Sync Strategies
7. Application Health and Status
8. Access Control and Security
9. Secrets Management in Argo CD
10. Notifications and Integrations
11. Best Practices
12. Resources for Further Learning

---

## 📌 What is Argo CD?

**Argo CD** is a declarative, GitOps continuous delivery tool for Kubernetes. It enables you to **deploy applications** to Kubernetes clusters by **syncing manifests stored in Git repositories**.

Think of Argo CD as a **GitOps controller** that automates and manages the deployment of Kubernetes resources based on the desired state stored in Git.

---

## ❓ Why Use Argo CD?

- Declarative deployment management
- Git as the source of truth
- Visual UI and CLI for managing applications
- Automatic or manual sync of apps
- Drift detection and reconciliation
- Multi-cluster support
- RBAC and SSO integrations

> Argo CD is purpose-built for Kubernetes and GitOps workflows.

---

## 🏗️ Core Concepts and Architecture

### 1. Application

An Argo CD **Application** is a Kubernetes custom resource that defines:
- The Git repository
- The path to the manifests
- The destination cluster and namespace

### 2. Repositories

Argo CD can connect to:
- Git repositories (public/private)
- Helm chart repositories
- Kustomize directories

### 3. Sync

Argo CD **syncs** the live cluster state with the desired state in Git.

### 4. Argo CD Components

- **argocd-server**: API server and web UI
- **argocd-repo-server**: Clones and renders manifests from Git
- **argocd-application-controller**: Reconciles app state
- **argocd-dex-server**: Handles SSO (optional)

---

## ⚙️ Argo CD Installation

### Install Argo CD using kubectl

```bash
kubectl create namespace argocd

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Port-forward the Argo CD UI (for local access)

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### Login with CLI

```bash
argocd login localhost:8080

Default username: admin  
Initial password: generated from argocd-server pod

kubectl get pods -n argocd -l app.kubernetes.io/name=argocd-server -o name

kubectl -n argocd exec <pod-name> -- argo admin initial-password
```

---

## 🚀 Managing Applications with Argo CD

You can create applications using:

- The **Web UI**
- The **CLI**
- Kubernetes manifests (declarative)

### CLI Example

```bash
argocd app create my-app \
  --repo https://github.com/your-org/your-repo \
  --path k8s/app \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default
```

### Sync the application
```bash
argocd app sync my-app
```

### View app status

```bash
argocd app get my-app
```

---

## 🔁 Sync Strategies

Argo CD supports multiple sync strategies:

### 1. Manual Sync

You must run `argocd app sync` manually or click sync in the UI.

### 2. Automatic Sync

Argo CD will automatically apply changes when detected in Git.

Enable auto-sync with:

argocd app set my-app --sync-policy automated

### 3. Prune

Remove Kubernetes resources that no longer exist in Git.

argocd app set my-app --sync-policy automated --self-heal --auto-prune

---

## 📊 Application Health and Status

Argo CD continuously monitors application health:

- **Healthy**: All resources are running as expected
- **Progressing**: Waiting for resources to become healthy
- **Degraded**: Some resources failed
- **Unknown**: Argo CD can't determine status

### View health status

argocd app health my-app

---

## 🔐 Access Control and Security

Argo CD supports:

- Role-Based Access Control (RBAC)
- Single Sign-On (SSO) integrations:
  - GitHub
  - GitLab
  - LDAP
  - Okta
- API tokens for automation

### Example: Create a read-only role

Edit the `argocd-rbac-cm` ConfigMap:

kind: ConfigMap  
metadata:  
  name: argocd-rbac-cm  
data:  
  policy.csv: |  
    g, read-only, role:readonly  
    p, role:readonly, applications, get, */*, allow

---

## 🔐 Secrets Management in Argo CD

Since GitOps workflows put manifests in Git, secrets must be encrypted or managed securely.

### Approaches:

- **Sealed Secrets** by Bitnami
- **SOPS** (Secrets OPerationS)
- **External Secrets Operator** integrated with Vault or AWS Secrets Manager

> Avoid committing plain-text secrets in Git!

---

## 🔔 Notifications and Integrations

Argo CD supports notifications via the **argocd-notifications** controller.

You can send alerts to:

- Slack
- Microsoft Teams
- Email
- Webhooks

### Example use cases:

- Send Slack alert when sync fails
- Notify via email on application health degradation
- Trigger external workflows using webhooks

---

## 📋 Best Practices

- Use declarative Application manifests in Git
- Enable auto-sync with pruning and self-heal for production apps
- Use RBAC and SSO for access control
- Monitor application health and sync status
- Avoid storing unencrypted secrets in Git
- Use separate repos or folders for different environments
- Keep Argo CD versions up to date

---

## 📚 Resources for Further Learning

- https://argo-cd.readthedocs.io/
- https://github.com/argoproj/argo-cd
- https://argo-cd.readthedocs.io/en/stable/operator-manual/rbac/
- https://github.com/argoproj-labs/argocd-notifications
- https://www.youtube.com/watch?v=Z6ZqZ3zY2yY (CNCF Argo CD demo)
- https://argo-cd.readthedocs.io/en/stable/user-guide/auto_sync/

---

## 🧠 Summary

Argo CD is a powerful GitOps controller that helps teams manage Kubernetes applications declaratively and securely. With features like automatic sync, health monitoring, and SSO integrations, Argo CD is a core tool in any modern Kubernetes platform.

> Argo CD = GitOps + Kubernetes + Automation + Security

---