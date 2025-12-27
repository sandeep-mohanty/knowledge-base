# COMPLETE Python Single Pipeline Pattern (Build → Docker → K8s → Newman → Rollback) - Artifactory Containers

**Everything in one pipeline**: Python build → Docker → Deploy → API tests → **Auto-rollback on failure**. Full templates, mental models, and production-ready code.

---

## 📋 Table of Contents
1. [Mental Model & Architecture](#mental-model)
2. [Prerequisites & Service Connections](#prerequisites)
3. [Complete Templates Directory](#templates)
4. [Single Pipeline: Complete Flow](#single-pipeline)
5. [Workflow & Usage](#workflow)
6. [Debugging & Troubleshooting](#debugging)

***

## 🧠 1. Mental Model & Architecture

### Pipeline Flow Diagram
```text
Push to main → Python Pipeline #123
                    ↓
           Poetry → Docker → Deploy:new123
                    ↓
               Newman Tests ✅ → Success ✅
               Newman Tests ❌ → Auto-Rollback → :456 ✅
```

**5-Stage Safety Pattern**:
```
1. Python Build (Poetry/Pip + Tests)
2. Docker Build & Push (Kaniko)
3. K8s Deploy (Store previous → Deploy new)
4. Newman API Tests (Critical gate)
5. Rollback (Auto if tests fail)
```

***

## 🔧 2. Prerequisites & Service Connections

### Azure DevOps Service Connections
```
1. artifactory-docker → Artifactory Docker registry
2. k8s-prod-cluster → Kubernetes cluster
3. python-prod → Azure DevOps Environment (approvals)
```

### Artifactory Images
```
your-artifactory.com/python/build:3.11-poetry
your-artifactory.com/tools/newman:5.3
your-artifactory.com/apps/python-service:123 (built by pipeline)
```

### Repo Structure (Python Service)
```
services/python-service/
├── Dockerfile
├── pyproject.toml (Poetry) OR requirements.txt (Pip)
├── tests/
│   ├── test_unit.py
│   └── test_api.py
├── postman/
│   ├── sanity.json
│   └── prod.env.json
└── app/
    └── main.py
```

***

## 📂 3. Complete Templates Directory

### templates/python/steps-python-build.yml
```yaml
parameters:
  - name: pythonVersion
    type: string
    default: '3.11'
  - name: packageManager
    type: string
    default: 'poetry'  # 'poetry' or 'pip'
  - name: requirementsPath
    type: string
    default: 'pyproject.toml'
  - name: testCommand
    type: string
    default: 'pytest tests/ -v'
  - name: workingDirectory
    type: string
    default: '.'
  - name: imageTag
    type: string
    default: '3.11-poetry'

steps:
  - task: Docker@2
    displayName: 'Login to Artifactory'
    inputs:
      command: 'login'
      containerRegistry: 'artifactory-docker'

  - script: |
      docker pull your-artifactory.com/python/build:${{ parameters.imageTag }}
    displayName: 'Pull Python build image'

  # Poetry or Pip install
  - ${{ if eq(parameters.packageManager, 'poetry') }}:
    - script: |
        docker run --rm \
          -v $(System.DefaultWorkingDirectory):/workspace \
          -w /workspace/${{ parameters.workingDirectory }} \
          your-artifactory.com/python/build:${{ parameters.imageTag }} \
          poetry install --no-dev
      displayName: 'Poetry install'

  - ${{ if eq(parameters.packageManager, 'pip') }}:
    - script: |
        docker run --rm \
          -v $(System.DefaultWorkingDirectory):/workspace \
          -w /workspace/${{ parameters.workingDirectory }} \
          your-artifactory.com/python/build:${{ parameters.imageTag }} \
          pip install -r ${{ parameters.requirementsPath }}
      displayName: 'Pip install'

  # Run tests
  - script: |
      docker run --rm \
        -v $(System.DefaultWorkingDirectory):/workspace \
        -w /workspace/${{ parameters.workingDirectory }} \
        your-artifactory.com/python/build:${{ parameters.imageTag }} \
        ${{ parameters.testCommand }}
    displayName: 'Run tests: ${{ parameters.testCommand }}'
```

### templates/python/steps-docker-kaniko.yml
```yaml
parameters:
  - name: imageName
    type: string
    required: true
  - name: imageTag
    type: string
    default: '$(Build.BuildId)'
  - name: dockerfilePath
    type: string
    default: 'Dockerfile'
  - name: contextPath
    type: string
    default: '.'

steps:
  - task: Docker@2
    inputs:
      command: 'login'
      containerRegistry: 'artifactory-docker'

  - task: Kaniko@1
    displayName: 'Build & Push Python Docker image'
    inputs:
      dockerfile: '${{ parameters.dockerfilePath }}'
      context: '${{ parameters.contextPath }}'
      imageName: '${{ parameters.imageName }}'
      imageTag: '${{ parameters.imageTag }}'
      registryServiceConnection: 'artifactory-docker'
```

### templates/python/steps-k8s-deploy-rollback.yml
```yaml
parameters:
  - name: kubeConfigServiceConnection
    type: string
    required: true
  - name: namespace
    type: string
    default: 'prod'
  - name: deploymentName
    type: string
    required: true
  - name: imageName
    type: string
    required: true
  - name: imageTag
    type: string
    default: '$(Build.BuildId)'

steps:
  # Store previous image
  - task: Kubernetes@1
    inputs:
      connectionType: 'Kubernetes Service Connection'
      kubernetesServiceEndpoint: '${{ parameters.kubeConfigServiceConnection }}'
      namespace: '${{ parameters.namespace }}'

  - script: |
      kubectl get deployment/${{ parameters.deploymentName }} -n ${{ parameters.namespace }} \
        -o jsonpath='{.spec.template.spec.containers[0].image}' \
        | cut -d':' -f2 > previous-image.txt
      echo "Previous image: $(cat previous-image.txt)"
    displayName: '📝 Store PREVIOUS image'

  # Deploy new image
  - script: |
      kubectl set image deployment/${{ parameters.deploymentName }} \
        ${{ parameters.deploymentName }}=${{ parameters.imageName }}:${{ parameters.imageTag }} \
        -n ${{ parameters.namespace }}
    displayName: '🚀 Deploy NEW: ${{ parameters.imageTag }}'

  - script: |
      kubectl rollout status deployment/${{ parameters.deploymentName }} \
        -n ${{ parameters.namespace }} --timeout=300s
    displayName: '⏳ Wait rollout'
```

### templates/python/steps-newman-rollback.yml
```yaml
parameters:
  - name: serviceUrl
    type: string
    required: true
  - name: kubeConfigServiceConnection
    type: string
    required: true
  - name: namespace
    type: string
    default: 'prod'
  - name: deploymentName
    type: string
    required: true

steps:
  # Newman tests
  - script: |
      docker pull your-artifactory.com/tools/newman:5.3
    displayName: 'Pull Newman image'

  - script: |
      docker run --rm \
        -v $(System.DefaultWorkingDirectory):/workspace \
        -e TARGET_URL="${{ parameters.serviceUrl }}" \
        your-artifactory.com/tools/newman:5.3 \
        newman run postman/sanity.json \
        --env-var TARGET_URL=$TARGET_URL \
        --reporters cli,junit \
        --reporter-junit-export newman-results.xml
    displayName: '🧪 Newman API Tests'

  - task: PublishTestResults@2
    inputs:
      testResultsFormat: 'JUnit'
      testResultsFiles: '**/newman-results.xml'
    condition: always()

  # Auto-rollback on failure
  - task: Kubernetes@1
    inputs:
      connectionType: 'Kubernetes Service Connection'
      kubernetesServiceEndpoint: '${{ parameters.kubeConfigServiceConnection }}'
      namespace: '${{ parameters.namespace }}'
    condition: failed()

  - script: |
      echo "❌ Tests FAILED → ROLLING BACK"
      kubectl rollout undo deployment/${{ parameters.deploymentName }} \
        -n ${{ parameters.namespace }}
    displayName: '🔄 AUTO-ROLLBACK'
    condition: failed()

  - script: |
      kubectl rollout status deployment/${{ parameters.deploymentName }} \
        -n ${{ parameters.namespace }} --timeout=180s
    displayName: '⏳ Wait rollback'
    condition: failed()
```

***

## 🚀 4. Single Pipeline: Complete Flow `<a name="single-pipeline"></a>`

**File**: `Python-Service/azure-pipelines.yml`

```yaml
name: python-k8s-rollback-$(Date:yyyyMMdd)$(Rev:.r)

trigger:
  branches: [ main, develop ]
  paths:
    include: [ 'services/python-service/**' ]

resources:
  repositories:
    - repository: templates
      type: git
      name: Org/azure-pipelines-templates

stages:
  # 1. Python Build & Test
  - stage: 'Python_Build'
    displayName: '1️⃣ Python Build & Test'
    jobs:
      - job: PythonBuild
        pool:
          vmImage: 'ubuntu-latest'
        container: your-artifactory.com/python/build:3.11-poetry
        steps:
          - checkout: self
          - template: python/steps-python-build.yml
            parameters:
              pythonVersion: '3.11'
              packageManager: 'poetry'
              testCommand: 'pytest tests/ -v --junitxml=pytest-results.xml'

  # 2. Docker Build & Push
  - stage: 'Docker_Build'
    displayName: '2️⃣ Docker Build & Push'
    dependsOn: Python_Build
    jobs:
      - job: DockerBuild
        steps:
          - template: python/steps-docker-kaniko.yml
            parameters:
              imageName: 'your-artifactory.com/apps/python-service'
              imageTag: '$(Build.BuildId)'
              dockerfilePath: 'services/python-service/Dockerfile'

  # 3. K8s Deploy (with rollback prep)
  - stage: 'K8s_Deploy'
    displayName: '3️⃣ Deploy to K8s'
    dependsOn: Docker_Build
    jobs:
      - deployment: DeployPython
        environment: 'python-prod'
        strategy:
          runOnce:
            deploy:
              steps:
                - template: python/steps-k8s-deploy-rollback.yml
                  parameters:
                    kubeConfigServiceConnection: 'k8s-prod-cluster'
                    namespace: 'prod'
                    deploymentName: 'python-service'
                    imageName: 'your-artifactory.com/apps/python-service'
                    imageTag: '$(Build.BuildId)'

  # 4. Newman Tests + Auto-Rollback
  - stage: 'Newman_Tests_Rollback'
    displayName: '4️⃣ Newman Tests + Auto-Rollback'
    dependsOn: K8s_Deploy
    condition: always()
    jobs:
      - job: NewmanRollback
        steps:
          - template: python/steps-newman-rollback.yml
            parameters:
              serviceUrl: 'http://python-service.prod.svc.cluster.local:8000/health'
              kubeConfigServiceConnection: 'k8s-prod-cluster'
              namespace: 'prod'
              deploymentName: 'python-service'
```

***

## 🚀 5. Workflow & Usage `<a name="workflow"></a>`

### Execution Flow
```
1. git push main → Pipeline #123 starts
2. Python tests → Docker build → Deploy:new123
3. Newman tests ✅ → Pipeline success ✅
4. Newman tests ❌ → Auto-rollback to :456 → Pipeline failed ✅
```

### Azure DevOps UI Shows
```
1️⃣ Python Build: ✅
2️⃣ Docker Build: ✅  
3️⃣ Deploy to K8s: ✅ (image:123)
4️⃣ Newman Tests: ❌ → 🔄 AUTO-ROLLBACK ✅
```

***

## 🐛 6. Debugging & Troubleshooting `<a name="debugging"></a>`

### Common Issues Table
| Stage | Issue | Fix |
|-------|-------|-----|
| Python | `poetry: command not found` | Use correct `imageTag: 3.11-poetry` |
| Docker | `manifest unknown` | Check image in Artifactory |
| Newman | `Connection refused` | Verify `serviceUrl` + port |
| Rollback | `no revision to undo` | First deploy has no previous version |

### Debug Commands
```bash
# Python service health
kubectl port-forward svc/python-service -n prod 8000:8000
curl http://localhost:8000/health

# Rollout history
kubectl rollout history deployment/python-service -n prod
```

***

## 🎯 Summary

**✅ Single pipeline advantages**:
- **Simple to understand** (one YAML file)
- **Automatic rollback** (no manual intervention)
- **Full audit trail** (one pipeline run = one deployment cycle)
- **Production-ready** (environments, approvals, safety)

**🔄 Ready to scale**:
```
services/user-service/     → Same pipeline, different parameters
services/order-service/    → Same pipeline, different parameters
```

**Everything complete** - copy to `Python-Service/azure-pipelines.yml` and run!

***