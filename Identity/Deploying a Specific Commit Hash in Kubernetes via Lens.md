# 🚀 Deploying a Specific Commit Hash in Kubernetes via Lens

To deploy a specific version of your application using a **commit hash** in a Kubernetes cluster managed via **Lens** (formerly called *Freelens*, possibly a typo or rebrand), you need to specify the commit hash in the **image tag** of the Deployment YAML.

---

## ✅ Prerequisites

- Your application images are built and pushed to a container registry (e.g., Docker Hub, ECR, GCR) with the **commit hash as a tag**.
- Example: `myregistry.com/myapp:abc1234` where `abc1234` is the Git commit hash.

---

## 🛠️ Steps to Specify the Commit Hash in Kubernetes Deployment

1. **Open Lens** and connect to your cluster.
2. Navigate to **Workloads > Deployments**.
3. Either edit an existing deployment or create a new one.
4. In the deployment YAML, locate the `containers.image` field.
5. Specify the image with the commit hash as the tag.

---

## 📄 Example Deployment YAML Snippet

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: myregistry.com/myapp:abc1234  # 👈 Commit hash as tag
          ports:
            - containerPort: 80
```

💡 **Tip:** You can use `kubectl set image` as an alternative:

```bash
kubectl set image deployment/my-app my-app=myregistry.com/myapp:abc1234
```

---

## ✅ Best Practices

- Use **CI/CD** to automatically tag images with commit hashes.
- Keep a `latest` or `stable` tag for quick rollbacks.
- Add annotations with the Git commit hash for traceability:

```yaml
metadata:
  annotations:
    git-commit-hash: "abc1234"
```