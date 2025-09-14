# 🌍 GitOps with Terraform: A Complete Tutorial

## 🧭 Table of Contents

1. What is GitOps with Terraform?
2. Benefits of Combining GitOps and Terraform
3. Challenges and Considerations
4. GitOps Workflow with Terraform
5. Tools and Architecture
6. Example: GitOps for AWS Infrastructure Using Terraform
7. CI/CD Integration
8. Secrets Management
9. Best Practices
10. Resources for Further Learning

---

## 📌 What is GitOps with Terraform?

**GitOps with Terraform** is the practice of managing cloud infrastructure using **Terraform**, while applying **GitOps principles** such as:

- Git as the single source of truth
- Infrastructure defined declaratively
- Pull-based or automated workflows triggered by Git changes
- Automated reconciliation and drift detection

> It brings the power of GitOps into infrastructure provisioning and cloud resource management using Terraform.

---

## ✅ Benefits of Combining GitOps and Terraform

- **Version-controlled infrastructure** with full audit history
- **Automated provisioning** through Git commits and pull requests
- **Improved collaboration** using Git workflows (PR reviews, approvals)
- **Drift detection** and reconciliation for infrastructure
- **Security**: No need to give developers direct access to cloud consoles
- **Consistency** across environments (dev, staging, prod)

---

## ⚠️ Challenges and Considerations

- Terraform is **not natively pull-based** like Kubernetes and Argo CD
- State management (e.g., remote backends) must be handled securely
- Terraform is **not idempotent** in the same way as Kubernetes manifests
- Long-running apply operations may conflict with GitOps reconciliation cycles

> Solution: Use GitOps **principles** with **CI/CD pipelines** to manage Terraform workflows.

---

## 🔄 GitOps Workflow with Terraform

### High-Level Steps:

1. Developer creates a PR with changes to Terraform code
2. PR is reviewed and approved
3. CI pipeline runs `terraform plan` and posts output to PR
4. On merge to main, CI runs `terraform apply`
5. Terraform provisions or updates infrastructure
6. Git history serves as audit trail

---

## 🧰 Tools and Architecture

| Layer        | Tool Examples                      | Purpose                                         |
|--------------|------------------------------------|-------------------------------------------------|
| VCS          | GitHub, GitLab, Bitbucket          | Source of truth for Terraform code              |
| CI/CD        | GitHub Actions, GitLab CI, CircleCI| Run Terraform plan/apply                        |
| Terraform    | CLI, Terraform Cloud, Atlantis     | Infrastructure as Code                          |
| State Backend| S3 + DynamoDB, Terraform Cloud     | Store Terraform state securely                  |
| Secrets      | Vault, SOPS, AWS Secrets Manager   | Manage sensitive variables                      |
| Notification | Slack, Email, GitHub Checks        | Post plan results and apply status              |

---

## 🛠️ Example: GitOps for AWS Infrastructure Using Terraform

### Goal:
Provision an AWS S3 bucket using GitOps principles and Terraform.

### Prerequisites:

- Git repo to store Terraform code
- AWS credentials (use IAM role or secrets manager)
- CI/CD system (e.g., GitHub Actions)
- Terraform backend configured (e.g., S3 + DynamoDB)

### Step-by-Step:

1. **Define Terraform Code**

Repo structure:

infrastructure/
  └── s3/
      ├── main.tf
      ├── variables.tf
      └── backend.tf

main.tf:

resource "aws_s3_bucket" "my_bucket" {
  bucket = var.bucket_name
  acl    = "private"
}

variables.tf:

variable "bucket_name" {
  type = string
}

backend.tf:

terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "s3/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
  }
}

2. **Create GitHub Actions Workflow**

.github/workflows/terraform.yaml:

name: Terraform S3 GitOps

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  terraform:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Terraform
      uses: hashicorp/setup-terraform@v2
      with:
        terraform_version: 1.5.0

    - name: Terraform Init
      run: terraform init

    - name: Terraform Plan
      run: terraform plan -var="bucket_name=my-gitops-bucket"

    - name: Terraform Apply
      if: github.ref == 'refs/heads/main'
      run: terraform apply -auto-approve -var="bucket_name=my-gitops-bucket"

3. **Create a Pull Request**

- Modify the bucket name or add a new resource
- Open a PR
- CI runs `terraform plan` and shows the expected changes

4. **Merge the PR**

- Trigger `terraform apply` automatically
- Infrastructure is provisioned

---

## 🔐 Secrets Management

Avoid committing secrets (AWS credentials, tokens) directly in Git.

### Options:

- Use **OpenID Connect (OIDC)** for GitHub Actions + AWS
- Store secrets in **GitHub Actions Secrets**, **Vault**, or **AWS Secrets Manager**
- Use **Terraform Cloud** with variable sets

---

## 🔄 CI/CD Integration

Use GitOps-style pipelines:

- `terraform fmt` for formatting checks
- `terraform validate` for syntax checks
- `terraform plan` for previewing changes
- `terraform apply` only on protected branches (e.g., `main`)
- PR workflows for approval and visibility

---

## 📋 Best Practices

- Use **remote state** with locking (S3 + DynamoDB or Terraform Cloud)
- Require **manual approval** for production applies
- Store **Terraform modules** in a separate repo or registry
- Use **feature branches** for changes, merged via pull requests
- Implement **drift detection** via scheduled plan runs
- Avoid storing **plaintext secrets** in Git
- Use **pre-commit hooks** to enforce standards

---

## 📚 Resources for Further Learning

- https://developer.hashicorp.com/terraform
- https://www.terraform.io/cloud
- https://github.com/runatlantis/atlantis
- https://github.com/gruntwork-io/terragrunt
- https://github.com/antonbabenko/pre-commit-terraform
- https://www.hashicorp.com/resources/gitops-pipeline-terraform

---

## 🧠 Summary

GitOps with Terraform brings version control, collaboration, and automation to infrastructure management. While Terraform is not natively pull-based, you can adopt GitOps workflows using CI/CD pipelines, Git PRs, and secure state management to achieve safe, auditable, and scalable infrastructure as code.

> GitOps for Terraform = Git + Terraform + Automation + CI/CD Discipline

---