# YAML pipeline container jobs - Azure Pipelines

**Azure DevOps Services | Azure DevOps Server | Azure DevOps Server 2022**

This article explains container jobs in Azure Pipelines. Containers are lightweight abstractions from the host operating system that provide all the necessary elements to run a job in a specific environment.

By default, Azure Pipelines [jobs](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/phases?view=azure-devops) run directly on [agents](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/agents?view=azure-devops) installed on host machines. Hosted agent jobs are convenient, require little initial setup or infrastructure maintenance, and are well-suited for basic projects. For more control over task context, you can define and run pipeline jobs in containers to get the exact versions of operating systems, tools, and dependencies you want.

For a container job, the agent first fetches and starts the container, and then runs each step of the job inside the container. If you need finer-grained control of individual build steps, you can use [step targets](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/tasks?view=azure-devops#step-target) to choose a container or host for each step.

## Requirements for container jobs

-   A YAML-based pipeline. Classic pipelines don't support container jobs.
-   A Windows or Ubuntu hosted agent. MacOS agents don't support containers. To use non-Ubuntu Linux agents, see [Nonglibc-based containers](#nonglibc-based-containers).
-   Docker installed on the agent, with permission to access the Docker daemon.
-   Agent running directly on the host, not already inside a container. Nested containers aren't supported.

-   [Linux](#tabpanel_1_linux)
-   [Windows](#tabpanel_1_windows)

Linux-based containers also have the following requirements:

-   Bash installed.
-   GNU C Library (`glibc`)-based. Nonglibc containers require added setup. For more information, see [Nonglibc-based containers](#nonglibc-based-containers).
-   No `ENTRYPOINT`. Containers with an `ENTRYPOINT` might not work, because [docker exec](https://docs.docker.com/reference/cli/docker/container/exec) expects the container to always be running.
-   `USER` provided with access to `groupadd` and other privileged commands without using `sudo`.
-   Ability to run Node.js, which the agent provides.
    
    Note
    
    Node.js must be preinstalled for Linux containers on Windows hosts.
    

Some stripped-down containers available on Docker Hub, especially containers based on Alpine Linux, don't satisfy these requirements. For more information, see [Nonglibc-based containers](#nonglibc-based-containers).

## Single job

The following example defines a Windows or Linux single-job container.

-   [Linux](#tabpanel_2_linux)
-   [Windows](#tabpanel_2_windows)

This example tells the system to fetch the `ubuntu` image tagged `18.04` from [Docker Hub](https://hub.docker.com) and then start the container. The `printenv` command runs inside the `ubuntu:18.04` container.

```yaml
pool:
  vmImage: 'ubuntu-latest'

container: ubuntu:18.04

steps:
- script: printenv
```

## Multiple jobs

You can use containers to run the same step in multiple jobs. The following example runs the same step in multiple versions of Ubuntu Linux. You don't have to use the `jobs` keyword, because only a single job is defined.

```yaml
pool:
  vmImage: 'ubuntu-latest'

strategy:
  matrix:
    ubuntu16:
      containerImage: ubuntu:16.04
    ubuntu18:
      containerImage: ubuntu:18.04
    ubuntu20:
      containerImage: ubuntu:20.04

container: $[ variables['containerImage'] ]

steps:
- script: printenv
```

### Multiple jobs on a single agent host

A container job uses the underlying host agent's Docker configuration file for image registry authorization. This file signs out at the end of the Docker registry container initialization.

Registry image pulls for container jobs could be denied for unauthorized authentication if another job running in parallel on the agent already signed out the Docker configuration file. The solution is to set a Docker environment variable called `DOCKER_CONFIG` for each agent pool running on the hosted agent.

Export the `DOCKER_CONFIG` in each agent pool's _runsvc.sh_ script as follows:

```bash
export DOCKER_CONFIG=./.docker
```

## Startup options

You can use the `options` property to specify options for container startup.

```yaml
container:
  image: ubuntu:18.04
  options: --hostname container-test --ip 192.168.0.1

steps:
- script: echo hello
```

Run `docker create --help` to get the list of options you can pass to Docker invocation. Not all these options are guaranteed to work with Azure Pipelines. Check first to see if you can use a `container` property for the same purpose.

For more information, see the [docker container create](https://docs.docker.com/reference/cli/docker/container/create/) command reference and the [resources.containers.container](https://learn.microsoft.com/en-us/azure/devops/pipelines/yaml-schema/resources-containers-container) definition in the [YAML schema reference for Azure Pipelines](https://learn.microsoft.com/en-us/azure/devops/pipelines/yaml-schema).

## Reusable container definition

The following YAML example defines the containers in the `resources` section, and then references them by their assigned aliases. The `jobs` keyword is used for clarity.

```yaml
resources:
  containers:
  - container: u16
    image: ubuntu:16.04

  - container: u18
    image: ubuntu:18.04

  - container: u20
    image: ubuntu:20.04

jobs:
- job: RunInContainer
  pool:
    vmImage: 'ubuntu-latest'

  strategy:
    matrix:
      ubuntu16:
        containerResource: u16
      ubuntu18:
        containerResource: u18
      ubuntu20:
        containerResource: u20

  container: $[ variables['containerResource'] ]

  steps:
  - script: printenv
```

## Service endpoints

You can host containers on registries other than public Docker Hub. To host an image on [Azure Container Registry](https://learn.microsoft.com/en-us/azure/container-registry/) or another private container registry, including a private Docker Hub registry, add a [service connection](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints?view=azure-devops#docker-registry-service-connection) to access the registry. Then you can reference the endpoint in the container definition.

Private Docker Hub connection:

```yaml
container:
  image: registry:ubuntu1804
  endpoint: private_dockerhub_connection
```

Azure Container Registry connection:

```yaml
container:
  image: myprivate.azurecr.io/windowsservercore:1803
  endpoint: my_acr_connection
```

Note

Azure Pipelines can't set up a service connection for Amazon Elastic Container Registry (ECR), because Amazon ECR requires other client tools to convert Amazon Web Services (AWS) credentials to be usable for Docker authentication.

## Nonglibc-based containers

The hosted Azure Pipelines agents supply Node.js, which is required to run tasks and scripts. The Node.js version compiles against the C runtime used in the hosted cloud, typically `glibc`. Some Linux variants use other C runtimes. For instance, Alpine Linux uses `musl`. For more information, see [Microsoft-hosted agents](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/hosted?view=azure-devops#software).

If you want to use a nonglibc-based container in a pipeline, you must:

-   Supply your own copy of Node.js.
-   Add a label to your image pointing to the location of the Node.js binary.
-   Provide the `bash`, `sudo`, `which`, and `groupadd` Azure Pipelines dependencies.

### Supply your own Node.js

If you use a nonglibc-based container, you must add a Node binary to your container. Node.js 18 is a safe choice. Start from the `node:18-alpine` image.

### Direct the agent to Node.js

The agent reads the container label `"com.azure.dev.pipelines.handler.node.path"`. If this label exists, it must be the path to the Node.js binary.

For example, in an image based on `node:18-alpine`, add the following line to your Dockerfile:

```dockerfile
LABEL "com.azure.dev.pipelines.agent.handler.node.path"="/usr/local/bin/node"
```

### Add required packages

Azure Pipelines requires a Bash-based system to have common administrative packages installed. Alpine Linux doesn't have several of the needed packages. Install `bash`, `sudo`, and `shadow` to cover basic needs.

```dockerfile
RUN apk add bash sudo shadow
```

If you depend on any built-in or Marketplace tasks, also supply the binaries they require.

### Full Dockerfile example

```dockerfile
FROM node:18-alpine

RUN apk add --no-cache --virtual .pipeline-deps readline linux-pam \
  && apk add bash sudo shadow \
  && apk del .pipeline-deps

LABEL "com.azure.dev.pipelines.agent.handler.node.path"="/usr/local/bin/node"

CMD [ "node" ]
```
