# Mastering Reusable Azure Pipelines Templates for Java (Gradle + K8s + Newman + Rollback) - Artifactory Containers

**Complete CI/CD with safety net**: Gradle → Docker → Deploy → Newman tests → **Rollback if tests fail**.

***

## 1. Updated mental model: Deploy + Test + Rollback

```text
Pipeline with Rollback Safety
 ├─ 1. Gradle Build
 ├─ 2. Docker Build & Push  
 ├─ 3. Deploy to K8s (NEW VERSION)
 │
 ├─ 4. Newman Sanity Tests
 │    ↓
 ├─ 5A. SUCCESS → Promote (✅)
 │    ↓
 └─ 5B. FAILURE → ROLLBACK to previous version (🔄)
```

**Rollback strategy**:
- Store **previous image tag** before deploy
- Deploy **new image**
- Run **Newman tests**
- If tests fail → `kubectl rollout undo` to previous image

***

## 2. Enhanced K8s deploy template (with rollback prep)

```yaml
# templates/java/steps-k8s-deploy-rollback.yml
parameters:
  - name: kubeConfigServiceConnection
    type: string
    required: true
  - name: namespace
    type: string
    default: 'dev'
  - name: deploymentName
    type: string
    required: true
  - name: imageName
    type: string
    required: true
  - name: imageTag
    type: string
    default: '$(Build.BuildId)'
  - name: replicas
    type: number
    default: 1
  - name: previousImageTagVar
    type: string
    default: 'PREVIOUS_IMAGE_TAG'  # Pipeline variable to store old tag

steps:
  # STEP 1: Get current deployment image (before deploy)
  - script: |
      kubectl get deployment/${{ parameters.deploymentName }} \
        -n ${{ parameters.namespace }} \
        -o jsonpath='{.spec.template.spec.containers[0].image}' \
        | cut -d':' -f2 \
        > previous_image_tag.txt
    displayName: 'Get previous image tag'
    env:
      KUBECONFIG: $(kubeconfig)

  - script: |
      PREV_TAG=$(cat previous_image_tag.txt)
      echo "##vso[task.setvariable variable=${{ parameters.previousImageTagVar }}]$PREV_TAG"
    displayName: 'Store previous image tag as pipeline variable'
    env:
      KUBECONFIG: $(kubeconfig)

  # STEP 2: Deploy new image
  - task: Kubernetes@1
    displayName: 'Set K8s context'
    inputs:
      connectionType: 'Kubernetes Service Connection'
      kubernetesServiceEndpoint: '${{ parameters.kubeConfigServiceConnection }}'
      namespace: '${{ parameters.namespace }}'

  - script: |
      kubectl set image deployment/${{ parameters.deploymentName }} \
        ${{ parameters.deploymentName }}=${{ parameters.imageName }}:${{ parameters.imageTag }} \
        --namespace=${{ parameters.namespace }}
    displayName: 'Deploy NEW image: ${{ parameters.imageTag }}'

  - script: |
      kubectl rollout status deployment/${{ parameters.deploymentName }} \
        --namespace=${{ parameters.namespace }} --timeout=300s
    displayName: 'Wait for rollout complete'
```

***

## 3. Enhanced Newman tests template (with rollback trigger)

```yaml
# templates/java/steps-newman-rollback.yml
parameters:
  - name: serviceUrl
    type: string
    required: true
  - name: postmanCollection
    type: string
    default: 'postman/sanity.json'
  - name: environment
    type: string
    default: 'postman/dev.env.json'
  - name: newmanImage
    type: string
    default: 'your-artifactory.com/tools/newman:5.3'
  - name: previousImageTagVar
    type: string
    default: 'PREVIOUS_IMAGE_TAG'
  - name: kubeConfigServiceConnection
    type: string
    required: true
  - name: namespace
    type: string
    default: 'dev'
  - name: deploymentName
    type: string
    required: true
  - name: imageName
    type: string
    required: true

steps:
  # Run Newman tests
  - script: |
      docker pull ${{ parameters.newmanImage }}
    displayName: 'Pull Newman image'

  - script: |
      docker run --rm \
        -v $(System.DefaultWorkingDirectory):/workspace \
        -e POSTMAN_COLLECTION="/workspace/${{ parameters.postmanCollection }}" \
        -e POSTMAN_ENVIRONMENT="/workspace/${{ parameters.environment }}" \
        -e TARGET_URL="${{ parameters.serviceUrl }}" \
        ${{ parameters.newmanImage }} \
        newman run $POSTMAN_COLLECTION \
        -e $POSTMAN_ENVIRONMENT \
        --env-var TARGET_URL=$TARGET_URL \
        --reporters cli,junit \
        --reporter-junit-export newman-results.xml
    displayName: 'Newman API sanity tests (CRITICAL)'

  # PUBLISH RESULTS (always)
  - task: PublishTestResults@2
    displayName: 'Publish Newman results'
    inputs:
      testResultsFormat: 'JUnit'
      testResultsFiles: '**/newman-results.xml'
    condition: always()

  # ROLLBACK IF TESTS FAIL
  - task: Kubernetes@1
    displayName: 'Set K8s context for rollback'
    inputs:
      connectionType: 'Kubernetes Service Connection'
      kubernetesServiceEndpoint: '${{ parameters.kubeConfigServiceConnection }}'
      namespace: '${{ parameters.namespace }}'
    condition: failed()

  - script: |
      echo "🚨 Newman tests FAILED - Initiating ROLLBACK"
      echo "Rolling back to previous image: $(${{ parameters.previousImageTagVar }})"
      
      kubectl rollout undo deployment/${{ parameters.deploymentName }} \
        --to-revision=1 \
        --namespace=${{ parameters.namespace }}
    displayName: '🔄 ROLLBACK to previous version'
    condition: failed()

  - script: |
      kubectl rollout status deployment/${{ parameters.deploymentName }} \
        --namespace=${{ parameters.namespace }} --timeout=180s
    displayName: 'Wait for rollback complete'
    condition: failed()
```

***

## 4. Complete 4-stage pipeline with rollback

```yaml
# templates/java/stage-gradle-k8s-rollback.yml
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
  - name: namespace
    type: string
    default: 'dev'
  - name: replicas
    type: number
    default: 1
  - name: serviceUrl
    type: string
    required: true
  - name: kubeConfigServiceConnection
    type: string
    required: true
  - name: registryServiceConnection
    type: string
    default: 'artifactory-docker'
  - name: previousImageTagVar
    type: string
    default: 'PREVIOUS_IMAGE_TAG'

stages:
  - stage: 'Gradle_Build'
    displayName: '1️⃣ Gradle Build'
    jobs:
      - job: GradleBuild
        container: your-artifactory.com/java/gradle-build:${{ parameters.imageTag }}
        steps:
          - template: java/steps-gradle-artifactory.yml
            parameters:
              imageTag: ${{ parameters.imageTag }}
              gradleTasks: ${{ parameters.gradleTasks }}
              workingDirectory: ${{ parameters.workingDirectory }}

  - stage: 'Docker_Build'
    displayName: '2️⃣ Docker Build & Push'
    dependsOn: Gradle_Build
    jobs:
      - job: DockerBuild
        steps:
          - template: java/steps-docker-kaniko.yml
            parameters:
              imageName: 'your-artifactory.com/apps/${{ parameters.serviceName }}'
              imageTag: '$(Build.BuildId)'
              dockerfilePath: '${{ parameters.workingDirectory }}/Dockerfile'

  - stage: 'K8s_Deploy'
    displayName: '3️⃣ Deploy to K8s'
    dependsOn: Docker_Build
    jobs:
      - deployment: DeployK8s
        environment: '${{ parameters.namespace }}'
        strategy:
          runOnce:
            deploy:
              steps:
                - template: java/steps-k8s-deploy-rollback.yml
                  parameters:
                    kubeConfigServiceConnection: '${{ parameters.kubeConfigServiceConnection }}'
                    namespace: '${{ parameters.namespace }}'
                    deploymentName: '${{ parameters.serviceName }}'
                    imageName: 'your-artifactory.com/apps/${{ parameters.serviceName }}'
                    imageTag: '$(Build.BuildId)'
                    previousImageTagVar: '${{ parameters.previousImageTagVar }}'

  - stage: 'Newman_Tests_Rollback'
    displayName: '4️⃣ Newman Tests + Auto-Rollback'
    dependsOn: K8s_Deploy
    condition: always()  # Always run tests
    jobs:
      - job: NewmanRollback
        steps:
          - template: java/steps-newman-rollback.yml
            parameters:
              serviceUrl: '${{ parameters.serviceUrl }}'
              postmanCollection: 'postman/sanity.json'
              newmanImage: 'your-artifactory.com/tools/newman:5.3'
              previousImageTagVar: '${{ parameters.previousImageTagVar }}'
              kubeConfigServiceConnection: '${{ parameters.kubeConfigServiceConnection }}'
              namespace: '${{ parameters.namespace }}'
              deploymentName: '${{ parameters.serviceName }}'
              imageName: 'your-artifactory.com/apps/${{ parameters.serviceName }}'
```

***

## 5. Per-project pipeline (with rollback safety)

```yaml
# azure-pipelines.yml (Production-ready with rollback)
name: payment-k8s-rollback-$(Date:yyyyMMdd)$(Rev:.r)

trigger:
  branches: [ main ]
  paths:
    include: [ 'services/payment-service/**' ]

resources:
  repositories:
    - repository: templates
      type: git
      name: Org/azure-pipelines-templates

stages:
  - template: templates/java/stage-gradle-k8s-rollback.yml@templates
    parameters:
      serviceName: 'payment-service'
      imageTag: '17-3.9'
      gradleTasks: 'clean build bootJar'
      workingDirectory: 'services/payment-service'
      namespace: 'prod'                    # Production!
      replicas: 3
      serviceUrl: 'http://payment-service.prod.svc.cluster.local:8080/api/health'
      kubeConfigServiceConnection: 'k8s-prod-cluster'
      registryServiceConnection: 'artifactory-docker'
      previousImageTagVar: 'PAYMENT_PREV_TAG'
```

***

## 6. Rollback behavior visualization

```text
✅ SUCCESS path:
Gradle → Docker → Deploy:new123 → Newman:✅ → Complete

❌ FAILURE path:
Gradle → Docker → Deploy:new123 → Newman:❌ 
                           ↓
                    Store prev:456
                           ↓
                    kubectl rollout undo → back to :456
                           ↓
                    Pipeline marks as FAILED
```

**Key safety features**:
- **Previous image stored** before any changes
- **Rollback only on Newman failure** (`condition: failed()`)
- **Always publish test results** for investigation
- **Environment protection** via Azure DevOps environments

***

## 7. Advanced: Blue/Green rollback (alternative)

For zero-downtime rollback, use **two deployments**:

```yaml
# Alternative: Blue/Green pattern
stages:
  - stage: BlueGreenDeploy
    jobs:
      - deployment: Blue
        environment: 'prod-blue'
      - deployment: Green  
        environment: 'prod-green'
        strategy:
          blueGreen: true
```

But `rollout undo` is simpler and works with existing K8s deployments.

***

## 8. Monitoring rollback in Azure DevOps UI

**Pipeline run will show**:
```
3️⃣ Deploy to K8s: ✅ (new image:123 deployed)
4️⃣ Newman Tests: ❌ (2/5 tests failed)
   🔄 ROLLBACK to previous version: ✅ (back to image:456)
```

**K8s side**:
```
kubectl get deployment payment-service -n prod -w
# new123 → failing → rollback to 456
```

***

This gives you **production-grade safety**: automatic rollback on API test failure, full audit trail, and zero manual intervention.
