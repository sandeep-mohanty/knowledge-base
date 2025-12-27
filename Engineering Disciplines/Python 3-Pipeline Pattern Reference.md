# COMPLETE Python 3-Pipeline Pattern Reference (Build → Test → Rollback) - Artifactory Containers

**Three independent pipelines** for maximum flexibility, auditability, and team ownership:

1. **`Python-Build`** → Poetry/Pip → Docker → Deploy new version
2. **`Python-Newman-Tests`** → API sanity tests (auto/manual trigger)  
3. **`Python-Rollback`** → Rollback to previous version (manual trigger)

***

## 📋 Table of Contents
1. [Mental Model & Architecture](#mental-model)
2. [Prerequisites & Service Connections](#prerequisites)
3. [Complete Templates Directory](#templates)
4. [Pipeline 1: Python-Build](#pipeline1)
5. [Pipeline 2: Python-Newman-Tests](#pipeline2)
6. [Pipeline 3: Python-Rollback](#pipeline3)
7. [Workflow & Usage](#workflow)
8. [Debugging & Troubleshooting](#debugging)

***

## 🧠 1. Mental Model & Architecture

### Pipeline Flow Diagram
```text
Push to main → Python-Build #123
                    ↓
         Poetry → Docker → Deploy:new123
                    ↓ Artifact: deploy-info.json
Python-Newman-Tests #123 (auto-triggered)
                    ↓
                Newman Tests ✅ → Success
                Newman Tests ❌ → Manual: Python-Rollback #123
                                     ↓
                               Rollback to :456 ✅
```

### Key Artifacts Between Pipelines
```
deploy-info.json:
{
  "buildId": "123",
  "imageTag": "123", 
  "imageName": "your-artifactory.com/apps/python-service",
  "namespace": "prod",
  "deploymentName": "python-service",
  "serviceUrl": "http://python-service.prod.svc.cluster.local:8000/health"
}

previous-image.txt: "456"
```

***

## 🔧 2. Prerequisites & Service Connections

### Azure DevOps Service Connections
```
1. artifactory-docker → Artifactory Docker registry
2. k8s-prod-cluster → Kubernetes cluster (Kubeconfig)
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
├── pyproject.toml (Poetry) OR requirements.txt
├── tests/
│   ├── test_unit.py
│   └── conftest.py
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
    default: 'pytest tests/ -v --junitxml=pytest-results.xml'
  - name: workingDirectory
    type: string
    default: '.'
  - name: imageTag
    type: string
    default: '3.11-poetry'
  - name: registryServiceConnection
    type: string
    default: 'artifactory-docker'

steps:
  - task: Docker@2
    displayName: 'Login to Artifactory'
    inputs:
      command: 'login'
      containerRegistry: ${{ parameters.registryServiceConnection }}

  - script: |
      docker pull your-artifactory.com/python/build:${{ parameters.imageTag }}
    displayName: 'Pull Python build image'

  # Install dependencies
  - ${{ if eq(parameters.packageManager, 'poetry') }}:
    - script: |
        docker run --rm \
          -v $(System.DefaultWorkingDirectory):/workspace \
          -w /workspace/${{ parameters.workingDirectory }} \
          your-artifactory.com/python/build:${{ parameters.imageTag }} \
          poetry install --no-dev
      displayName: 'Poetry install (no-dev)'

  - ${{ if eq(parameters.packageManager, 'pip') }}:
    - script: |
        docker run --rm \
          -v $(System.DefaultWorkingDirectory):/workspace \
          -w /workspace/${{ parameters.workingDirectory }} \
          your-artifactory.com/python/build:${{ parameters.imageTag }} \
          pip install -r ${{ parameters.requirementsPath }}
      displayName: 'Pip install'

  # Run unit tests
  - script: |
      docker run --rm \
        -v $(System.DefaultWorkingDirectory):/workspace \
        -w /workspace/${{ parameters.workingDirectory }} \
        your-artifactory.com/python/build:${{ parameters.imageTag }} \
        ${{ parameters.testCommand }}
    displayName: 'pytest: ${{ parameters.testCommand }}'

  - task: PublishTestResults@2
    displayName: 'Publish pytest results'
    inputs:
      testResultsFormat: 'JUnit'
      testResultsFiles: '**/pytest-results.xml'
    condition: always()
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
  - name: registryServiceConnection
    type: string
    default: 'artifactory-docker'

steps:
  - task: Docker@2
    inputs:
      command: 'login'
      containerRegistry: ${{ parameters.registryServiceConnection }}

  - task: Kaniko@1
    displayName: 'Build & Push Python Docker image'
    inputs:
      dockerfile: '${{ parameters.dockerfilePath }}'
      context: '${{ parameters.contextPath }}'
      imageName: '${{ parameters.imageName }}'
      imageTag: '${{ parameters.imageTag }}'
      registryServiceConnection: '${{ parameters.registryServiceConnection }}'
```

### templates/python/stage-python-build-only.yml
```yaml
parameters:
  - name: serviceName
    type: string
    required: true
  - name: pythonVersion
    type: string
    default: '3.11'
  - name: packageManager
    type: string
    default: 'poetry'
  - name: testCommand
    type: string
    default: 'pytest tests/ -v --junitxml=pytest-results.xml'
  - name: workingDirectory
    type: string
    default: '.'
  - name: imageTag
    type: string
    default: '3.11-poetry'
  - name: registryServiceConnection
    type: string
    default: 'artifactory-docker'
  - name: vmImage
    type: string
    default: 'ubuntu-latest'

stages:
  - stage: 'Python_Build'
    displayName: '1️⃣ Python Build & Test'
    jobs:
      - job: PythonBuild
        pool:
          vmImage: ${{ parameters.vmImage }}
        container: your-artifactory.com/python/build:${{ parameters.imageTag }}
        steps:
          - checkout: self
            clean: true
          - template: python/steps-python-build.yml
            parameters:
              pythonVersion: ${{ parameters.pythonVersion }}
              packageManager: ${{ parameters.packageManager }}
              testCommand: ${{ parameters.testCommand }}
              workingDirectory: ${{ parameters.workingDirectory }}
              imageTag: ${{ parameters.imageTag }}
              registryServiceConnection: ${{ parameters.registryServiceConnection }}

  - stage: 'Docker_Build'
    displayName: '2️⃣ Docker Build & Push'
    dependsOn: Python_Build
    jobs:
      - job: DockerBuild
        pool:
          vmImage: ${{ parameters.vmImage }}
        steps:
          - template: python/steps-docker-kaniko.yml
            parameters:
              imageName: 'your-artifactory.com/apps/${{ parameters.serviceName }}'
              imageTag: '$(Build.BuildId)'
              dockerfilePath: '${{ parameters.workingDirectory }}/Dockerfile'
              contextPath: '${{ parameters.workingDirectory }}'
              registryServiceConnection: '${{ parameters.registryServiceConnection }}'
```

***

## 🛠️ 4. Pipeline 1: Python-Build `<a name="pipeline1"></a>`

**File**: `Python-Build/azure-pipelines.yml`

```yaml
name: python-build-$(Date:yyyyMMdd)$(Rev:.r)

trigger:
  branches: [ main, develop ]
  paths:
    include: [ 'services/python-service/**' ]

resources:
  repositories:
    - repository: templates
      type: git
      name: Org/azure-pipelines-templates

  pipelines:
    - pipeline: newman-tests
      source: 'Python-Newman-Tests'
      trigger:
        stages:
          - of type: 'deployment'
      triggerResource: auto

stages:
  - template: templates/python/stage-python-build-only.yml@templates
    parameters:
      serviceName: 'python-service'
      pythonVersion: '3.11'
      packageManager: 'poetry'
      testCommand: 'pytest tests/ -v --junitxml=pytest-results.xml'
      workingDirectory: 'services/python-service'
      imageTag: '3.11-poetry'
      registryServiceConnection: 'artifactory-docker'

  - stage: 'Deploy_New_Version'
    displayName: '🚀 Deploy NEW version'
    dependsOn: Docker_Build
    jobs:
      - deployment: DeployNewVersion
        environment: 'python-prod'
        strategy:
          runOnce:
            deploy:
              steps:
                # Store previous image BEFORE deploy
                - task: Kubernetes@1
                  inputs:
                    connectionType: 'Kubernetes Service Connection'
                    kubernetesServiceEndpoint: 'k8s-prod-cluster'
                    namespace: 'prod'

                - script: |
                    kubectl get deployment/python-service -n prod \
                      -o jsonpath='{.spec.template.spec.containers[0].image}' \
                      | cut -d':' -f2 > $(Build.ArtifactStagingDirectory)/previous-image.txt
                    echo "Previous image: $(cat $(Build.ArtifactStagingDirectory)/previous-image.txt)"
                  displayName: '📝 Store PREVIOUS image tag'

                # Deploy NEW image
                - script: |
                    kubectl set image deployment/python-service \
                      python-service=your-artifactory.com/apps/python-service:$(Build.BuildId) \
                      -n prod
                  displayName: '🚀 Deploy NEW: $(Build.BuildId)'

                - script: |
                    kubectl rollout status deployment/python-service -n prod --timeout=300s
                  displayName: '⏳ Wait rollout'

                # Create deploy-info artifact
                - script: |
                    cat > $(Build.ArtifactStagingDirectory)/deploy-info.json << EOF
                    {
                      "buildId": "$(Build.BuildId)",
                      "imageTag": "$(Build.BuildId)",
                      "imageName": "your-artifactory.com/apps/python-service",
                      "namespace": "prod",
                      "deploymentName": "python-service",
                      "serviceUrl": "http://python-service.prod.svc.cluster.local:8000/health"
                    }
                    EOF
                  displayName: '📦 Create deploy-info.json'

                # Publish artifacts
                - task: PublishPipelineArtifact@1
                  displayName: 'Publish deploy-info artifact'
                  inputs:
                    targetPath: '$(Build.ArtifactStagingDirectory)'
                    artifact: 'deploy-info'

                - task: PublishPipelineArtifact@1
                  displayName: 'Publish rollback-info artifact'
                  inputs:
                    targetPath: '$(Build.ArtifactStagingDirectory)/previous-image.txt'
                    artifact: 'rollback-info'
```

***

## 🧪 5. Pipeline 2: Python-Newman-Tests `<a name="pipeline2"></a>`

**File**: `Python-Newman-Tests/azure-pipelines.yml`

```yaml
name: python-newman-tests-$(Date:yyyyMMdd)$(Rev:.r)

trigger: none

resources:
  pipelines:
    - pipeline: python-build
      source: 'Python-Build'
      trigger:
        stages:
          - of type: 'deployment'
      versions:
        - latest
      triggerResource: auto

stages:
  - stage: 'Download_Deploy_Info'
    displayName: '📥 Download deploy info'
    jobs:
      - job: DownloadArtifacts
        steps:
          - download: python-build
            artifact: deploy-info

          - script: |
              cat $(Pipeline.Workspace)/deploy-info/deploy-info.json
              SERVICE_URL=$(jq -r '.serviceUrl' $(Pipeline.Workspace)/deploy-info/deploy-info.json)
              DEPLOYMENT_NAME=$(jq -r '.deploymentName' $(Pipeline.Workspace)/deploy-info/deploy-info.json)
              echo "##vso[task.setvariable variable=ServiceUrl]$SERVICE_URL"
              echo "##vso[task.setvariable variable=DeploymentName]$DEPLOYMENT_NAME"
            displayName: 'Parse deploy-info'

  - stage: 'Newman_API_Tests'
    displayName: '🧪 Newman Sanity Tests'
    dependsOn: Download_Deploy_Info
    jobs:
      - job: NewmanTests
        pool:
          vmImage: ubuntu-latest
        container: your-artifactory.com/tools/newman:5.3
        steps:
          - script: |
              newman run postman/sanity.json \
                --env-var TARGET_URL="$(ServiceUrl)" \
                --reporters cli,junit \
                --reporter-junit-export newman-results.xml
            displayName: '🔍 API Tests: $(ServiceUrl)'

          - task: PublishTestResults@2
            displayName: '📊 Publish Newman results'
            inputs:
              testResultsFormat: 'JUnit'
              testResultsFiles: '**/newman-results.xml'
            condition: always()

          - script: |
              echo "✅ Tests PASSED - Deployment $(ServiceUrl) is healthy"
            displayName: '✅ Tests PASSED'
            condition: succeeded()

          - script: |
              echo "❌ Tests FAILED - Deployment $(ServiceUrl) is unhealthy"
              echo "Manual intervention required: Queue Python-Rollback pipeline"
            displayName: '❌ Tests FAILED - Trigger Rollback manually'
            condition: failed()
```

***

## 🔄 6. Pipeline 3: Python-Rollback `<a name="pipeline3"></a>`

**File**: `Python-Rollback/azure-pipelines.yml`

```yaml
name: python-rollback-$(Date:yyyyMMdd)$(Rev:.r)

parameters:
  - name: targetNamespace
    type: string
    default: 'prod'
  - name: deploymentName
    type: string
    default: 'python-service'
  - name: previousImageTag
    type: string
    default: ''
  - name: useRolloutUndo
    type: boolean
    default: true

trigger: none  # Manual only

stages:
  - stage: 'Rollback_Deployment'
    displayName: '🔄 Rollback Deployment'
    jobs:
      - job: PerformRollback
        pool:
          vmImage: ubuntu-latest
        steps:
          # Option A: Rollout undo (recommended)
          - ${{ if eq(parameters.useRolloutUndo, true) }}:
            - task: Kubernetes@1
              inputs:
                connectionType: 'Kubernetes Service Connection'
                kubernetesServiceEndpoint: 'k8s-prod-cluster'
                namespace: '${{ parameters.targetNamespace }}'

            - script: |
                echo "🔄 Rolling back deployment/${{ parameters.deploymentName }} to previous revision"
                kubectl rollout undo deployment/${{ parameters.deploymentName }} \
                  -n ${{ parameters.targetNamespace }}
              displayName: '🔄 Rollout Undo (safest)'

            - script: |
                kubectl rollout status deployment/${{ parameters.deploymentName }} \
                  -n ${{ parameters.targetNamespace }} --timeout=180s
              displayName: '⏳ Wait for rollback'

          # Option B: Specific image tag
          - ${{ if and(ne(parameters.previousImageTag, ''), eq(parameters.useRolloutUndo, false)) }}:
            - task: Kubernetes@1
              inputs:
                connectionType: 'Kubernetes Service Connection'
                kubernetesServiceEndpoint: 'k8s-prod-cluster'
                namespace: '${{ parameters.targetNamespace }}'

            - script: |
                echo "🔄 Setting image to previous tag: ${{ parameters.previousImageTag }}"
                kubectl set image deployment/${{ parameters.deploymentName }} \
                  ${{ parameters.deploymentName }}=your-artifactory.com/apps/${{ parameters.deploymentName }}:${{ parameters.previousImageTag }} \
                  -n ${{ parameters.targetNamespace }}
              displayName: '🔄 Rollback to specific image tag'

          # Option C: From artifact
          - ${{ if eq(parameters.previousImageTag, '') }}:
            - download: python-build
              artifact: rollback-info

            - script: |
                PREV_TAG=$(cat $(Pipeline.Workspace)/rollback-info/previous-image.txt)
                echo "Previous tag from artifact: $PREV_TAG"
                kubectl set image deployment/${{ parameters.deploymentName }} \
                  ${{ parameters.deploymentName }}=your-artifactory.com/apps/${{ parameters.deploymentName }}:$PREV_TAG \
                  -n ${{ parameters.targetNamespace }}
              displayName: '🔄 Rollback using artifact tag'
```

***

## 🚀 7. Workflow & Usage `<a name="workflow"></a>`

### Complete Execution Flow
```
1. git push main → Python-Build #123 starts
2. Python-Build #123 ✅ → Auto-triggers Python-Newman-Tests #123
3. Python-Newman-Tests #123 ✅ → Pipeline success ✅
   OR Python-Newman-Tests #123 ❌ → Manual: Queue Python-Rollback
4. Python-Rollback #123 → Back to previous version ✅
```

### Manual Queue Commands

**Python-Newman-Tests** (re-test):
```
Queue → Python-Newman-Tests (auto-downloads latest build artifacts)
```

**Python-Rollback** (after test failure):
```
Queue → Python-Rollback
Parameters:
- targetNamespace: prod
- deploymentName: python-service
- useRolloutUndo: true (recommended)
```

***

## 🐛 8. Debugging & Troubleshooting `<a name="debugging"></a>`

### Common Issues Table

| Stage | Issue | Symptoms | Fix |
|-------|-------|----------|-----|
| Build | `poetry not found` | `command not found` | Use `imageTag: 3.11-poetry` |
| Docker | `manifest unknown` | `no such image` | Check Artifactory image exists |
| Newman | `Connection refused` | `ECONNREFUSED` | Verify `serviceUrl:8000/health` |
| Rollback | `no revision to undo` | First deploy | Use specific `previousImageTag` |

### Debug Checklist
```
1. Python-Build: Check Docker image in Artifactory
   → your-artifactory.com/apps/python-service:123

2. Python-Newman-Tests: Verify serviceUrl
   → curl http://python-service.prod.svc.cluster.local:8000/health

3. Python-Rollback: Check deployment history
   → kubectl rollout history deployment/python-service -n prod
```

### K8s Verification Commands
```bash
# Check current deployment
kubectl get deployment python-service -n prod -o yaml

# View rollout history
kubectl rollout history deployment/python-service -n prod

# Test service health
kubectl port-forward svc/python-service -n prod 8000:8000
curl http://localhost:8000/health
```

***

## 🎯 Summary

**✅ What you get**:
- **3 independent pipelines** with clear ownership
- **Automatic triggering** (Build → Tests)
- **Manual rollback** with multiple safety options
- **Full audit trail** across pipelines
- **Poetry/Pip support** via parameters
- **Reusable templates** for all Python services
- **Artifactory-first** container control

**🔄 Scale to other services**:
```
User-Build → User-Newman-Tests → User-Rollback
Order-Build → Order-Newman-Tests → Order-Rollback
```
