# 🐳 Azure DevOps Pipeline: Postman Tests Against a Deployed Public API

This tutorial describes how to configure an Azure DevOps pipeline that runs **Postman collections inside a container** (Newman image) against your already-deployed **public API**.
No deployment stage, no local spin-up — just clean API regression testing with every run.

---

## 1. Prerequisites

- **Azure DevOps project** and repo ready (contains the YAML + Postman collections).
- Exported **Postman Collections** (`.json`).
- Optionally exported **Environment JSON** (for variable injection like `baseUrl`).
- A **public base URL** for your API (e.g., `https://api.myapp.com`).

---

## 2. Example Repository Layout

```plaintext
api-tests/
├── tests/
│   ├── collections/
│   │    └── MyApiCollection.json
│   ├── environments/
│   │    └── public-environment.json
└── azure-pipelines.yml
```

---

## 3. Creating the Pipeline in Azure DevOps (Point to YAML)

1. Go to your DevOps project → **Pipelines → New Pipeline**.
2. Select your repository type (Azure Repos, GitHub, etc.).
3. Pick your repo.
4. Choose **Existing Azure Pipelines YAML file**.
5. Select the branch and path → `azure-pipelines.yml`.
6. Save & run. ✅ This registers the pipeline and executes the initial run.

From now on, the YAML controls *everything*.

---

## 4. The azure-pipelines.yml File (Containerized Newman Job)

Here’s a clean YAML pipeline running entirely inside a container that already has Newman installed:

```yaml
# azure-pipelines.yml

trigger:
  branches:
    include:
      - main   # Run tests on commits to main branch

pool:
  vmImage: 'ubuntu-latest'   # Azure host needed to run containers

container: postman/newman:alpine   # all steps below run in this Newman container

steps:
# Step 1: Checkout repo (to access collections/environments)
- checkout: self

# Step 2: Run Postman collection against deployed public API
- script: |
    newman run tests/collections/MyApiCollection.json \
      -e tests/environments/public-environment.json \
      --reporters cli,junit \
      --reporter-junit-export Results/postman-results.xml
  displayName: 'Run API Regression Tests'

# Step 3: Publish results to Azure DevOps
- task: PublishTestResults@2
  displayName: 'Publish Postman Test Results'
  inputs:
    testResultsFiles: 'Results/postman-results.xml'
    testRunTitle: 'Public API Test Harness'
    mergeTestResults: true
```

---

## 5. Public Environment Config

Your `public-environment.json` can simply point at your public API URL:

```json
{
  "id": "1234-5678",
  "name": "Public Environment",
  "values": [
    {
      "key": "baseUrl",
      "value": "https://api.myapp.com",
      "enabled": true
    }
  ]
}
```

Your Postman collection should then call `{{baseUrl}}/endpoint` everywhere.

---

## 6. Secure Variables in Pipeline

If sensitive values like tokens are needed, avoid hardcoding them in JSON. Use pipeline variables:

```yaml
- script: |
    newman run tests/collections/MyApiCollection.json \
      --env-var "baseUrl=https://api.myapp.com" \
      --env-var "authToken=$(API_TOKEN)" \
      --reporters cli,junit \
      --reporter-junit-export Results/postman-results.xml
  displayName: 'Run Secure Postman Tests'
```

Here `$(API_TOKEN)` comes from Azure DevOps **Pipeline Variables** (marked secret).

---

## 7. Scheduling Public Monitoring Runs

Because your API is already deployed and public, you can also use schedules to make this pipeline into a **monitoring harness**:

```yaml
schedules:
- cron: "0 3 * * *"   # Run daily at 3 AM UTC
  displayName: Daily API Health Check
  branches:
    include:
      - main
```

This lets you continuously validate your public API without waiting for code pushes.

---

## 8. Benefits of This Containerized Setup

- ✅ **Simplicity** — no deploy stage, API assumed ready
- ✅ **Portability** — same Newman container can be run locally (`docker run ...`)
- ✅ **Security** — secrets injected as pipeline variables, not committed to repo
- ✅ **Consistency** — uses official `postman/newman:alpine` container for all test runs

---

# ✅ Wrap-Up

This pipeline design does one thing and does it well:
- Runs your Postman harness against **already deployed, public APIs**
- Executes **entirely inside a container** for reproducibility
- Publishes rich results into Azure DevOps **Tests tab**
- Can run **on-demand, per commit, or on nightly schedule** for continuous API validation

You’ve now got a **containerized API test pipeline** that’s lightweight, self-contained, and production-safe 🐳✨