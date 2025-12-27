# COMPLETE Java Gradle 3-Pipeline Pattern Reference (Build → Test → Rollback)

**Everything in one file**: All templates, pipelines, mental models, diagrams, and usage examples for the **3-pipeline architecture** using **Artifactory containers + K8s + Newman + Rollback**.

***

## 📋 Table of Contents
1. [Mental Model & Architecture](#mental-model)
2. [Prerequisites & Service Connections](#prerequisites)
3. [Complete Templates Directory](#templates)
4. [Pipeline 1: Payment-Build](#pipeline1)
5. [Pipeline 2: Payment-Newman-Tests](#pipeline2)
6. [Pipeline 3: Payment-Rollback](#pipeline3)
7. [Workflow & Usage](#workflow)
8. [Debugging & Troubleshooting](#debugging)

***

## 🧠 1. Mental Model & Architecture

### Pipeline Flow Diagram
```text
Push to main → Payment-Build #123
                    ↓
             Gradle → Docker → Deploy:new123
                    ↓ Artifact: deploy-info.json
Payment-Newman-Tests #123 (auto-triggered)
                    ↓
                Newman Tests ✅ → Success
                Newman Tests ❌ → Manual: Payment-Rollback #123
                                     ↓
                               Rollback to :456 ✅
```

### Key Artifacts Between Pipelines
```
deploy-info.json:
{
  "buildId": "123",
  "imageTag": "123", 
  "imageName": "your-artifactory.com/apps/payment-service",
  "namespace": "prod",
  "deploymentName": "payment-service",
  "serviceUrl": "http://payment-service.prod.svc.cluster.local:8080/api/health"
}

previous-image.txt: "456"
```

***

## 🔧 2. Prerequisites & Service Connections

### Azure DevOps Service Connections
```
1. artifactory-docker → Artifactory Docker registry
2. k8s-prod-cluster → Kubernetes cluster (Kubeconfig)
3. payment-prod → Azure DevOps Environment (approvals)
```

### Artifactory Images
```
your-artifactory.com/java/gradle-build:17-3.9
your-artifactory.com/tools/newman:5.3
your-artifactory.com/apps/payment-service:123 (built by pipeline)
```

### Repo Structure (Payment Service)
```
services/payment-service/
├── Dockerfile
├── gradlew
├── build.gradle
├── postman/
│   ├── sanity.json
│   └── prod.env.json
└── k8s/
    └── deployment.yaml
```

***

## 📂 3. Complete Templates Directory

### templates/java/steps-gradle-artifactory.yml
```yaml
parameters:
  - name: imageTag
    type: string
    default: '17-3.9'
  - name: gradleTasks
    type: string
    default: 'clean build bootJar'
  - name: workingDirectory
    type: string
    default: '.'
  - name: publishArtifacts
    type: boolean
    default: true
  - name: artifactName
    type: string
    default: 'java-app'
  - name: gradleOpts
    type: string
    default: '-Xmx2g'
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
      docker pull your-artifactory.com/java/gradle-build:${{ parameters.imageTag }}
    displayName: 'Pull Artifactory Gradle image'

  - script: |
      docker run --rm \
        -v $(System.DefaultWorkingDirectory):/workspace \
        -w /workspace/${{ parameters.workingDirectory }} \
        -e GRADLE_OPTS="${{ parameters.gradleOpts }}" \
        -e GRADLE_USER_HOME="/workspace/.gradle" \
        your-artifactory.com/java/gradle-build:${{ parameters.imageTag }} \
        ./gradlew ${{ parameters.gradleTasks }} \
        --stacktrace --info
    displayName: 'Gradle: ${{ parameters.gradleTasks }}'

  - ${{ if eq(parameters.publishArtifacts, true) }}:
    - task: PublishPipelineArtifact@1
      inputs:
        targetPath: '$(System.DefaultWorkingDirectory)/${{ parameters.workingDirectory }}/build/libs'
        artifact: ${{ parameters.artifactName }}
```

### templates/java/steps-docker-kaniko.yml
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
    displayName: 'Build & Push Docker image'
    inputs:
      dockerfile: '${{ parameters.dockerfilePath }}'
      context: '${{ parameters.contextPath }}'
      imageName: '${{ parameters.imageName }}'
      imageTag: '${{ parameters.imageTag }}'
      registryServiceConnection: '${{ parameters.registryServiceConnection }}'
```

### templates/java/stage-gradle-build-only.yml
```yaml
parameters:
  - name: serviceName
    type: string
    required: true
  - name: imageTag
    type: string
    default: '17-3.9'
  - name: gradleTasks
    type: string
    default: 'clean build bootJar'
  - name: workingDirectory
    type: string
    default: '.'
  - name: registryServiceConnection
    type: string
    default: 'artifactory-docker'
  - name: vmImage
    type: string
    default: 'ubuntu-latest'

stages:
  - stage: 'Gradle_Build'
    displayName: '1️⃣ Gradle Build'
    jobs:
      - job: GradleBuild
        pool:
          vmImage: ${{ parameters.vmImage }}
        container: your-artifactory.com/java/gradle-build:${{ parameters.imageTag }}
        steps:
          - checkout: self
            clean: true
          - template: java/steps-gradle-artifactory.yml
            parameters:
              imageTag: ${{ parameters.imageTag }}
              gradleTasks: ${{ parameters.gradleTasks }}
              workingDirectory: ${{ parameters.workingDirectory }}
              publishArtifacts: true
              artifactName: '${{ parameters.serviceName }}-build'
              registryServiceConnection: ${{ parameters.registryServiceConnection }}

  - stage: 'Docker_Build'
    displayName: '2️⃣ Docker Build & Push'
    dependsOn: Gradle_Build
    jobs:
      - job: DockerBuild
        pool:
          vmImage: ${{ parameters.vmImage }}
        steps:
          - template: java/steps-docker-kaniko.yml
            parameters:
              imageName: 'your-artifactory.com/apps/${{ parameters.serviceName }}'
              imageTag: '$(Build.BuildId)'
              dockerfilePath: '${{ parameters.workingDirectory }}/Dockerfile'
              contextPath: '${{ parameters.workingDirectory }}'
              registryServiceConnection: '${{ parameters.registryServiceConnection }}'
```

***

## 🛠️ 4. Pipeline 1: Payment-Build `<a name="pipeline1"></a>`

**File**: `Payment-Build/azure-pipelines.yml`

```yaml
name: payment-build-$(Date:yyyyMMdd)$(Rev:.r)

trigger:
  branches: [ main, develop ]
  paths:
    include: [ 'services/payment-service/**' ]

resources:
  repositories:
    - repository: templates
      type: git
      name: Org/azure-pipelines-templates

  pipelines:
    - pipeline: newman-tests
      source: 'Payment-Newman-Tests'
      trigger:
        stages:
          - of type: 'deployment'
      triggerResource: auto

stages:
  - template: templates/java/stage-gradle-build-only.yml@templates
    parameters:
      serviceName: 'payment-service'
      imageTag: '17-3.9'
      gradleTasks: 'clean build bootJar'
      workingDirectory: 'services/payment-service'
      registryServiceConnection: 'artifactory-docker'

  - stage: 'Deploy_New_Version'
    displayName: '🚀 Deploy NEW version'
    dependsOn: Docker_Build
    jobs:
      - deployment: DeployNewVersion
        environment: 'payment-prod'
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
                    kubectl get deployment/payment-service -n prod \
                      -o jsonpath='{.spec.template.spec.containers[0].image}' \
                      | cut -d':' -f2 > $(Build.ArtifactStagingDirectory)/previous-image.txt
                    echo "Previous image: $(cat $(Build.ArtifactStagingDirectory)/previous-image.txt)"
                  displayName: '📝 Store PREVIOUS image tag'

                # Deploy NEW image
                - script: |
                    kubectl set image deployment/payment-service \
                      payment-service=your-artifactory.com/apps/payment-service:$(Build.BuildId) \
                      -n prod
                  displayName: '🚀 Deploy NEW: $(Build.BuildId)'

                - script: |
                    kubectl rollout status deployment/payment-service -n prod --timeout=300s
                  displayName: '⏳ Wait rollout'

                # Create deploy-info artifact
                - script: |
                    cat > $(Build.ArtifactStagingDirectory)/deploy-info.json << EOF
                    {
                      "buildId": "$(Build.BuildId)",
                      "imageTag": "$(Build.BuildId)",
                      "imageName": "your-artifactory.com/apps/payment-service",
                      "namespace": "prod",
                      "deploymentName": "payment-service",
                      "serviceUrl": "http://payment-service.prod.svc.cluster.local:8080/api/health"
                    }
                    EOF

                # Publish artifacts
                - task: PublishPipelineArtifact@1
                  inputs:
                    targetPath: '$(Build.ArtifactStagingDirectory)'
                    artifact: 'deploy-info'

                - task: PublishPipelineArtifact@1
                  inputs:
                    targetPath: '$(Build.ArtifactStagingDirectory)/previous-image.txt'
                    artifact: 'rollback-info'
```

***

## 🧪 5. Pipeline 2: Payment-Newman-Tests `<a name="pipeline2"></a>`

**File**: `Payment-Newman-Tests/azure-pipelines.yml`

```yaml
name: payment-newman-tests-$(Date:yyyyMMdd)$(Rev:.r)

trigger: none

resources:
  pipelines:
    - pipeline: payment-build
      source: 'Payment-Build'
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
          - download: payment-build
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
            inputs:
              testResultsFormat: 'JUnit'
              testResultsFiles: '**/newman-results.xml'
            condition: always()

          - script: |
              echo "✅ Deployment healthy: $(ServiceUrl)"
            condition: succeeded()

          - script: |
              echo "❌ Deployment unhealthy!"
              echo "🚨 Manual: Queue Payment-Rollback pipeline"
            condition: failed()
```

***

## 🔄 6. Pipeline 3: Payment-Rollback `<a name="pipeline3"></a>`

**File**: `Payment-Rollback/azure-pipelines.yml`

```yaml
name: payment-rollback-$(Date:yyyyMMdd)$(Rev:.r)

parameters:
  - name: targetNamespace
    type: string
    default: 'prod'
  - name: deploymentName
    type: string
    default: 'payment-service'
  - name: useRolloutUndo
    type: boolean
    default: true

trigger: none

stages:
  - stage: 'Rollback_Deployment'
    displayName: '🔄 Rollback Deployment'
    jobs:
      - job: PerformRollback
        pool:
          vmImage: ubuntu-latest
        steps:
          - ${{ if eq(parameters.useRolloutUndo, true) }}:
            - task: Kubernetes@1
              inputs:
                connectionType: 'Kubernetes Service Connection'
                kubernetesServiceEndpoint: 'k8s-prod-cluster'
                namespace: '${{ parameters.targetNamespace }}'

            - script: |
                echo "🔄 Rolling back to previous revision"
                kubectl rollout undo deployment/${{ parameters.deploymentName }} \
                  -n ${{ parameters.targetNamespace }}
              displayName: '🔄 Rollout Undo'

            - script: |
                kubectl rollout status deployment/${{ parameters.deploymentName }} \
                  -n ${{ parameters.targetNamespace }} --timeout=180s
              displayName: '⏳ Wait rollback'

          - ${{ if eq(parameters.useRolloutUndo, false) }}:
            - download: payment-build
              artifact: rollback-info

            - script: |
                PREV_TAG=$(cat $(Pipeline.Workspace)/rollback-info/previous-image.txt)
                echo "Previous tag: $PREV_TAG"
                kubectl set image deployment/${{ parameters.deploymentName }} \
                  ${{ parameters.deploymentName }}=your-artifactory.com/apps/${{ parameters.deploymentName }}:$PREV_TAG \
                  -n ${{ parameters.targetNamespace }}
              displayName: '🔄 Rollback to specific tag'
```

***

## 🚀 7. Workflow & Usage `<a name="workflow"></a>`

### Complete Execution Flow
```
1. git push main → Payment-Build #123 starts
2. Payment-Build #123 ✅ → Auto-triggers Payment-Newman-Tests #123
3. Payment-Newman-Tests #123 ✅ → Pipeline success ✅
   OR Payment-Newman-Tests #123 ❌ → Manual trigger Payment-Rollback
4. Payment-Rollback #123 → Deployment back to previous version ✅
```

### Manual Queue Commands

**Payment-Newman-Tests** (re-test):
```
Queue → Payment-Newman-Tests (auto-downloads latest build artifacts)
```

**Payment-Rollback** (after test failure):
```
Queue → Payment-Rollback
Parameters:
- targetNamespace: prod
- deploymentName: payment-service
- useRolloutUndo: true (recommended)
```

***

## 🐛 8. Debugging & Troubleshooting `<a name="debugging"></a>`

### Common Issues Table

| Stage | Issue | Symptoms | Fix |
|-------|-------|----------|-----|
| Build | Docker login fail | `unauthorized: authentication required` | Check `artifactory-docker` service connection |
| Build | Gradle OOM | `OutOfMemoryError` | Increase `gradleOpts: '-Xmx4g'` |
| Deploy | K8s connection | `Unable to connect to cluster` | Verify `k8s-prod-cluster` service connection |
| Newman | Service unreachable | `Connection refused` | Check `serviceUrl` in `deploy-info.json` |
| Rollback | No previous image | `previous-image.txt` empty | First deploy has no previous version |

### Debug Checklist
```
1. Payment-Build: Check Docker image in Artifactory
   → your-artifactory.com/apps/payment-service:123

2. Payment-Newman-Tests: Verify serviceUrl
   → curl http://payment-service.prod.svc.cluster.local:8080/api/health

3. Payment-Rollback: Check deployment history
   → kubectl rollout history deployment/payment-service -n prod
```

### K8s Verification Commands
```bash
# Check current deployment
kubectl get deployment payment-service -n prod -o yaml

# View rollout history  
kubectl rollout history deployment/payment-service -n prod

# Test service health
kubectl port-forward svc/payment-service -n prod 8080:8080
curl http://localhost:8080/api/health
```

***

## 🎯 Summary

**✅ What you get**:
- **3 independent pipelines** with clear ownership
- **Automatic triggering** between Build → Tests
- **Manual rollback** with safety (`rollout undo`)
- **Full audit trail** across all pipelines
- **Reusable templates** for all Java Gradle services
- **Artifactory-first** (no public registries)
- **Production-ready** with environments & approvals

**🔄 Scale to other services**:
```
Order-Build → Order-Newman-Tests → Order-Rollback
Inventory-Build → Inventory-Newman-Tests → Inventory-Rollback
```
