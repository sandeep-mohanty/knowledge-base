# Mastering Azure Pipelines YAML: A Mental-Model-First Deep Dive

## 1. Big-picture mental model

Think of an Azure YAML pipeline as a **declarative program** that the pipeline engine _compiles_ and then _executes_ in a distributed environment.

### 1.1 Conceptual architecture diagram

```text
                 +-----------------------------+
                 |      Azure DevOps UI       |
                 +--------------+--------------+
                                |
                                v
                       (1) Pipeline run request
                                |
                                v
+----------------+     +--------+--------+     +----------------------+
| Git Repository | --> | Pipeline Engine | --> |    Agent Pool(s)     |
| azure-pipelines.yml| | (Compiler+Orchestrator)| | self-hosted / MS-hosted |
+----------------+     +--------+--------+     +-----------+----------+
                                |                          |
                                v                          v
                      +---------+---------+       +--------+--------+
                      | Template Expansion|       |  Agents (VMs)   |
                      +---------+---------+       +-----------------+
                                |
                                v
                       +--------+--------+
                       |  Execution Graph|
                       | stages/jobs/steps|
                       +------------------+
```

Mentally separate three phases:

1. **Definition** phase  
   - YAML, templates, parameters, schema.
2. **Compilation** phase  
   - Template expansion, `${{ }}` evaluation, graph building.
3. **Execution** phase  
   - Conditions, variables, jobs/steps running on agents, logs.

Each phase has different “levers” and debugging strategies.

***

## 2. Lifecycle and control flow mental model

### 2.1 Lifecycle of a pipeline run

```text
[Trigger fires / Manual run]
              |
              v
   [Definition loaded from repo]
              |
              v
   [Template expansion + parameters (compile-time)]
              |
              v
   [Execution graph built: stages -> jobs -> steps]
              |
              v
   [Stage 1] -----> [Stage 2] -----> [Stage 3]
      |                |               |
      | jobs           | jobs          | jobs
      v                v               v
    Steps            Steps           Steps
```

At a high level:

- **Triggers** decide *whether to start* a run.
- **Templates + parameters** decide *what exists* in the run.
- **Conditions + dependsOn** decide *what actually executes* within that run.

***

## 3. Triggers, PRs, and schedules — richer examples

### 3.1 Multi-branch + path-filter mental model

Use a **matrix** in your head: branch vs path.

```text
Branches: main, develop, feature/*
Paths:    src/, infra/, docs/
```

Example YAML:

```yaml
trigger:
  branches:
    include: [ main, develop ]
    exclude: [ feature/* ]
  paths:
    include: [ src/, infra/ ]
    exclude: [ docs/ ]

pr:
  branches:
    include: [ main, develop, release/* ]
  paths:
    include: [ src/ ]
```

Mental model for a push:
- If branch is `feature/foo`, no CI run.
- If branch is `main` but changes only in `docs/`, no CI run.
- If PR targets `release/1.0` and touches `src/`, PR validation runs.

### 3.2 Multi-schedule scenario

```yaml
schedules:
  - cron: "0 2 * * 1-5"   # weekdays 2am UTC
    displayName: Nightly CI
    branches:
      include: [ develop ]

  - cron: "0 3 * * 0"     # Sundays 3am UTC
    displayName: Weekly security scan
    branches:
      include: [ main ]
    always: true
```

Mental model:  
- Nightly job ensures dev branch health.  
- Weekly job ensures main’s security posture regardless of changes.

You can explain: “One pipeline, but two different schedule roles.”

---

## 4. Stages, jobs, steps — hierarchical mental model with variants

### 4.1 Vertical view: pipeline “stack”

```text
Pipeline
 ├─ Stages (Build, Test, Deploy, etc.)
 │   ├─ Jobs (Linux, Windows, IntegrationTests, etc.)
 │   │   └─ Steps (tasks/scripts)
 │   └─ Deployment jobs (environments, approvals)
 └─ Shared: variables, templates, resources
```

### 4.2 Parallel vs sequential jobs — matrix example

```yaml
stages:
  - stage: Build
    displayName: 'Build per OS'
    jobs:
      - job: Build_Linux
        pool:
          vmImage: ubuntu-latest
        steps:
          - script: ./build.sh

      - job: Build_Windows
        pool:
          vmImage: windows-latest
        steps:
          - script: build.cmd

  - stage: Test
    dependsOn: Build
    jobs:
      - job: Test_All
        dependsOn:
          - Build_Linux
          - Build_Windows
        condition: succeeded()
        steps:
          - script: ./run-tests.sh
```

Mental model:
- In **Build** stage, Linux and Windows build jobs run in parallel.
- **Test_All** waits for both builds; if either fails, default `succeeded()` condition causes Test to skip.

You can reason: “If we want tests to run even if one build fails, we must adjust the condition or dependency structure.”

***

## 5. Variables, parameters, and expressions — mental model as “two evaluation layers”

### 5.1 Two namespaces mental diagram

```text
Compile-time (Template/Parameter Layer)
---------------------------------------
- parameters: name, type, default, values
- expressions: ${{ ... }}
- decides: what exists (jobs, stages, steps, templates)

Runtime (Variable Layer)
------------------------
- variables: pipeline vars, secret vars, variable groups
- notation: $(varName), variables['varName'] in conditions
- decides: how existing items behave (conditions, scripts)
```

### 5.2 Diverse examples

#### Example A: parameter-driven environment layout

```yaml
parameters:
  - name: targetEnv
    type: string
    default: dev
    values: [ dev, qa, prod ]

stages:
  - ${{ if eq(parameters.targetEnv, 'dev') }}:
    - stage: Dev
      jobs:
        - job: DevDeploy
          steps:
            - script: ./deploy-dev.sh

  - ${{ if in(parameters.targetEnv, 'qa', 'prod') }}:
    - stage: SharedPreChecks
      jobs:
        - job: Sanity
          steps:
            - script: ./precheck.sh

  - ${{ if eq(parameters.targetEnv, 'qa') }}:
    - stage: QA
      jobs:
        - job: QaDeploy
          steps:
            - script: ./deploy-qa.sh
```

Mental model: the **graph shape** itself depends on `targetEnv`.  
- For `dev`, you get Dev stage only.  
- For `qa`, you get PreChecks + QA.

#### Example B: runtime variables steering behavior

```yaml
variables:
  - name: runLongTests
    value: $[eq(variables['Build.SourceBranch'], 'refs/heads/main')]

stages:
  - stage: Tests
    jobs:
      - job: UnitTests
        condition: succeeded()
        steps:
          - script: ./unit-tests.sh

      - job: LongRunningTests
        condition: and(succeeded(), eq(variables['runLongTests'], 'True'))
        steps:
          - script: ./long-tests.sh
```

Mental model:  
- Graph is fixed: `UnitTests` and `LongRunningTests` always exist.  
- Whether **LongRunningTests executes** depends on branch.

---

## 6. Conditions and dependency patterns — diagrams + variants

### 6.1 Classic pipeline graph

```text
[Build] ---> [Test] ---> [Deploy]
   |            |            |
   v            v            v
 Steps        Steps        Steps
```

Default `condition: succeeded()` at each arrow.

### 6.2 Add monitoring job that always runs

```yaml
stages:
  - stage: BuildAndTest
    jobs:
      - job: Build
        steps: ...

      - job: Test
        dependsOn: Build
        condition: succeeded()
        steps: ...

      - job: ReportStatus
        dependsOn:
          - Build
          - Test
        condition: always()
        steps:
          - script: ./report-status.sh
```

Mental model:  
- `ReportStatus` is a “finally” block; it runs no matter what happened in upstream jobs.

### 6.3 Dev/test/prod deploy with different safety levels

```yaml
stages:
  - stage: Deploy_Dev
    dependsOn: BuildAndTest
    condition: succeeded()
    jobs:
      - deployment: Deploy_Dev
        environment: dev

  - stage: Deploy_Test
    dependsOn: Deploy_Dev
    condition: succeeded()
    jobs:
      - deployment: Deploy_Test
        environment: test

  - stage: Deploy_Prod
    dependsOn: Deploy_Test
    # extra guard: only from main branch
    condition: and(
      succeeded(),
      eq(variables['Build.SourceBranch'], 'refs/heads/main')
    )
    jobs:
      - deployment: Deploy_Prod
        environment: prod
```

Mental model:  
- Linear chain with increasing strictness.  
- You can add approvals on `test` and `prod` environments to make the lifecycle explicit and auditable.

***

## 7. Templates and reuse — “platform pipeline” mental model

### 7.1 Template architecture diagram

```text
Service repo A        Service repo B         Infra repo
-------------         -------------          ----------
azure-pipelines.yml   azure-pipelines.yml    templates/
    |                      |                 ├── steps/
    |                      |                 │     dotnet-build.yml
    v                      v                 ├── stages/
  uses templates @infra  uses templates @infra      deploy-stage.yml
```

In your head, treat templates as **library modules** with well-defined API:

- Inputs: `parameters`.
- Outputs: “shape” of stages/jobs/steps.

### 7.2 Steps template examples with variants

**Dotnet build template**

```yaml
# /templates/steps/dotnet-build.yml
parameters:
  - name: configuration
    type: string
    default: Release
  - name: projects
    type: string
    default: '**/*.csproj'

steps:
  - task: UseDotNet@2
    displayName: 'Use .NET 8 SDK'
    inputs:
      packageType: 'sdk'
      version: '8.0.x'

  - script: dotnet restore ${{ parameters.projects }}
    displayName: 'Restore'

  - script: dotnet build ${{ parameters.projects }} --configuration ${{ parameters.configuration }} --no-restore
    displayName: 'Build'
```

Usage in service A:

```yaml
steps:
  - template: templates/steps/dotnet-build.yml@infra
    parameters:
      configuration: Debug
      projects: 'src/ServiceA/*.csproj'
```

Usage in service B:

```yaml
steps:
  - template: templates/steps/dotnet-build.yml@infra
    parameters:
      configuration: Release
      projects: 'src/ServiceB/*.csproj'
```

Mental model:  
- One central definition, behavior specialization via parameters.  
- You can govern versions, flags centrally.

***

## 8. Environments and deployment jobs — mental model as “release tracks”

### 8.1 Environment-centric diagram

```text
[Build & Test] ---> [Deploy_Dev] ---> [Deploy_Test] ---> [Deploy_Prod]
                         |              |                 |
                  (Env: dev)    (Env: test)        (Env: prod)
                    approvals?     approvals?         approvals?
                    checks?        checks?            checks?
```

### 8.2 Canary vs direct deploy example

Single pipeline supporting both strategies via parameters.

```yaml
parameters:
  - name: deploymentStrategy
    type: string
    default: direct
    values: [ direct, canary ]

stages:
  - stage: Deploy
    dependsOn: Build
    jobs:
      - deployment: DeployToProd
        environment: prod
        strategy:
          ${{ if eq(parameters.deploymentStrategy, 'direct') }}:
            runOnce:
              deploy:
                steps:
                  - script: ./deploy-prod.sh

          ${{ if eq(parameters.deploymentStrategy, 'canary') }}:
            canary:
              increments: [ 25, 50, 100 ]
              preDeploy:
                steps:
                  - script: ./pre-canary-checks.sh
              deploy:
                steps:
                  - script: ./deploy-canary.sh
              routeTraffic:
                steps:
                  - script: ./shift-traffic.sh
              postRouteTraffic:
                steps:
                  - script: ./post-canary-verify.sh
```

Mental model:  
- Same pipeline, two **release tracks** selectable at queue time.  
- Shape of deployment strategy defined by template-like conditions.

***

## 9. Cross-cutting mental models for debugging

### 9.1 “Why didn’t X run?” checklist

In your head, scan in this order:

```text
1. Trigger & branch/path:
   - Did the run start when it should?

2. Template & parameters:
   - Is the stage/job even present in this run's graph?

3. dependsOn:
   - Are its dependencies present?
   - Did any dependency fail/skip?

4. condition:
   - Evaluate it manually with real values.

5. Agent context:
   - Pool, vmImage, container – is the environment as expected?
```

Use mental “if/else” reasoning instead of jumping straight into logs.

### 9.2 “How can I fix it?” multi-solution framing

When asked for a fix, structure answers like:

1. **Minimal change**  
   - Tweak `condition` or `dependsOn` only.
2. **Structural improvement**  
   - Introduce a new stage/job (e.g., dedicated cleanup or post-processing).
3. **Design refactor**  
   - Extract to template, add parameter, make behavior explicit.

Then compare:

- Cognitive load for future readers.
- Blast radius (number of pipelines impacted).
- Fit with existing patterns (organization conventions).

***

## 10. More diverse example: microservice monorepo

### 10.1 Scenario mental diagram

```text
Monorepo: services/api, services/web, services/worker
Goal: 
- Build only changed services.
- Run tests per service.
- Deploy impacted services only.
```

### 10.2 YAML skeleton

```yaml
trigger:
  branches:
    include: [ main, develop ]

variables:
  - name: changedServices
    value: ''  # set later via script

stages:
  - stage: DetectChanges
    jobs:
      - job: Diff
        steps:
          - script: |
              # pseudo: compute changed services and set variable
              echo "##vso[task.setvariable variable=changedServices]api,web"
            displayName: 'Detect changed services'

  - stage: BuildAndTest
    dependsOn: DetectChanges
    jobs:
      - job: BuildApi
        condition: contains(variables['changedServices'], 'api')
        steps:
          - script: ./build-api.sh
          - script: ./test-api.sh

      - job: BuildWeb
        condition: contains(variables['changedServices'], 'web')
        steps:
          - script: ./build-web.sh
          - script: ./test-web.sh

      - job: BuildWorker
        condition: contains(variables['changedServices'], 'worker')
        steps:
          - script: ./build-worker.sh
          - script: ./test-worker.sh

  - stage: Deploy
    dependsOn: BuildAndTest
    jobs:
      - deployment: DeployApi
        condition: contains(variables['changedServices'], 'api')
        environment: api-prod
        strategy:
          runOnce:
            deploy:
              steps:
                - script: ./deploy-api.sh

      - deployment: DeployWeb
        condition: contains(variables['changedServices'], 'web')
        environment: web-prod
        strategy:
          runOnce:
            deploy:
              steps:
                - script: ./deploy-web.sh
```

Mental model:
- **Diff stage** sets a variable storing changed services.
- Subsequent jobs use `condition` to decide whether to run.
- Structure is fixed; behavior changes based on repo changes.

This is a pattern you can adapt to complex multi-tenant or multi-service systems.

***

## 11. How to strengthen your own mental model

Given your background, these practices will solidify understanding:

- **Treat YAML as a DSL with compilation phases**  
  Internally think: “This part is compile-time; this part is runtime.”
- **Visualize graphs when reading YAML**  
  Sketch stages and jobs as nodes with arrows based on `dependsOn` and conditions.
- **Write alternate designs intentionally**
  - For any non-trivial need, draft:
    1. A simple runtime-condition solution.
    2. A parameterized template solution.
    3. A stage/job structural refactor.
- **Explain in “domain” terms, not YAML terms**
  - “We have three release lanes with approvals at test/prod and a canary route in prod”  
    instead of  
  - “We have three stages with a deployment job using canary strategy.”

This keeps explanations aligned with how stakeholders think, while your mental model remains grounded in YAML structure and pipeline execution behavior.
