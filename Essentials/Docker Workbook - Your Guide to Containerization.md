# Docker Workbook - Your Guide to Containerization

### Introduction

-   Containers technology can bring portability, maintainability, scalability and enhance the security of your application ecosystem if properly used
    
-   Containers bundle softwares and all their dependencies into a single process that runs on a Linux host machine
    
-   Containerisation is possible thanks to Linux functionalities like [chroot](https://www.geeksforgeeks.org/linux-unix/chroot-command-in-linux-with-examples/?ref=hackerstack.org), [namespaces](https://www.hackerstack.org/understanding-linux-namespaces) and [control groups](https://www.hackerstack.org/understanding-linux-cgroups)
    
-   Namespaces define resources containers can see and cgroups resources they can use and in which quantity
    
-   The containers share the Linux host machine Kernel, use the host hardware directly and are isolated from each other
    
-   Container runtimes are software that run containers, like [Docker](https://www.docker.com/?ref=hackerstack.org), [Containerd](https://containerd.io/?ref=hackerstack.org) and [CRI-O](https://cri-o.io/?ref=hackerstack.org)
    
-   Container technologies existed many years before Docker, but the massive adoption of the technology has began with Docker, thanks to a new approach and a very user friendly interface
    

### How Docker works

![Docker Architecture](https://docs.docker.com/get-started/images/docker-architecture.webp "Docker Architecture")

[Docker Architecture](https://docs.docker.com/get-started/docker-overview/?ref=hackerstack.org#docker-architecture)

When you [install Docker](https://docs.docker.com/get-started/get-docker/?ref=hackerstack.org), you get a client (the 'docker' command) and a daemon, most of the time running through a systemd service. The Docker Daemon runs in the background and get instructions from the Docker client, in order to download container images from [registries](#container-registries), [create container images](#creating-container-images-from-a-dockerfile) and [run containers](#managing-containers-with-docker). The client and daemon are not necessarily on the same machine. The machine on which the daemon runs is called a Docker Host.

Here are the CLI references for the Docker client and daemon:

-   [Docker client CLI reference](https://docs.docker.com/reference/cli/docker/?ref=hackerstack.org)
    
-   [Docker daemon CLI reference](https://docs.docker.com/reference/cli/dockerd/?ref=hackerstack.org)
    

Also, here are the reference documentations containing all the configuration directives for the Docker client and daemon configuration files:

-   [Docker client configuration file reference](https://docs.docker.com/reference/cli/docker/?ref=hackerstack.org#configuration-files)
    
-   [Docker daemon configuration file reference](https://docs.docker.com/reference/cli/dockerd/?ref=hackerstack.org#daemon-configuration-file)
    

### Container images

#### The Dockerfile

[Dockerfile reference](https://docs.docker.com/reference/dockerfile/?ref=hackerstack.org) | [Dockerfile instructions best practices](https://docs.docker.com/build/building/best-practices/?ref=hackerstack.org#dockerfile-instructions)

-   A container image is mandatory for running containers.
    
-   The standard file used for creating container images with Docker is called a [Dockerfile](https://docs.docker.com/reference/dockerfile/?ref=hackerstack.org). Inside that file, you write a set of [instructions](https://docs.docker.com/reference/dockerfile/?ref=hackerstack.org#overview) that describe the content of the image.
    
-   At the [Common Dockerfile instructions](#common-dockerfile-instructions) section, you will learn about commonly used Dockerfile instructions and their specificities. You can also [initialize a sample Dockerfile](https://docs.docker.com/reference/cli/docker/init/?ref=hackerstack.org) for your project.
    
-   Once your Dockerfile is ready, you create/build a container image from that file using tools like [Docker](https://www.docker.com/?ref=hackerstack.org), [Kaniko](https://github.com/GoogleContainerTools/kaniko?ref=hackerstack.org) or [Buildah](https://buildah.io/?ref=hackerstack.org). To exclude a specific file from the build context, you can use a [.dockerignore](https://docs.docker.com/engine/reference/builder/?ref=hackerstack.org#dockerignore-file) file.
    
-   Also, here are [best practices](https://docs.docker.com/build/building/best-practices/?ref=hackerstack.org#dockerfile-instructions) when writing Dockerfile instructions.
    
-   For language specific guides, have a look at [Docker language specific guides](https://docs.docker.com/guides/?ref=hackerstack.org):
    
    -   [PHP](https://docs.docker.com/guides/?languages=php&ref=hackerstack.org), [Python](https://docs.docker.com/guides/?languages=python&ref=hackerstack.org), [Java](https://docs.docker.com/guides/?languages=java&ref=hackerstack.org), [Ruby](https://docs.docker.com/guides/?languages=ruby&ref=hackerstack.org), [Go](https://docs.docker.com/guides/?languages=go&ref=hackerstack.org), [JavaScript](https://docs.docker.com/guides/?languages=js&ref=hackerstack.org), [Rust](https://docs.docker.com/guides/?languages=rust&ref=hackerstack.org), [R](https://docs.docker.com/guides/?languages=r&ref=hackerstack.org)
-   Lines you write inside the Dockerfile are handled by Docker parsers. You can customize the parser behavior by using [Parser directives](https://docs.docker.com/reference/dockerfile/?ref=hackerstack.org#parser-directives) which are written as a special type of comment in the form '# directive=value' inside the Dockerfile. They can be used for instance to choose the Dockerfile syntax, define espace characters or configure how [build checks](https://docs.docker.com/build/checks/?ref=hackerstack.org) are evaluated.
    

#### Container Registries

[Docker Hub](https://hub.docker.com/?ref=hackerstack.org)

-   Container Registries services can be used to store container image. They are most of the time accessible remotely and used to share container images.
    
-   [Docker Container Registry](https://hub.docker.com/?ref=hackerstack.org) is among the well-known publicly available container registries. When you run a container with Docker for instance, without specifying a specific container registry to use for downloading/pulling the image, this is the one used by default.
    

#### Creating container images from a Dockerfile

[Image building best practices](https://docs.docker.com/build/building/best-practices/?ref=hackerstack.org)

-   Go inside the folder where your Dockerfile lives and run the following command. The image tag is optional. If nothing is specified, 'latest' will be used:

```
# Syntax
docker build -t <image_name>[:<image_tag>] .

# Example
docker build -t myapp:0.1.0 .
```

-   In case you need to later push the image you have built into a remote container registry, the '<image\_name>' should correspond to the full URL of the image inside the container registry. Here is an example:

```
docker build -t myregistry.example.com/myapp:0.1.0 .
```

-   Use 'docker build --help' for more. Also, here are [best practices](https://docs.docker.com/build/building/best-practices/?ref=hackerstack.org) to be aware of when creating container images.

#### Pushing container images into registries

-   Before pushing your locally built images into remote container registries, ensure you have properly tagged the images during build time. The full image URL should be in the form:

```
# Registry image URL
<registry_url>/<image_name>:<image_tag>

# Examples
myregistry.example.com/tools/myapp:0.1.0
myregistry.example.com/superapp:0.1.0
```

-   Once the image is properly tagged, authenticate with the container registry (if required) by using the following command:

```
# Syntax
docker login <registry_domain_name>

# Example
docker login myregistry.example.com
```

-   Then, use the following command to push the image into the registry:

```
# Syntax
docker push <registry_image_url>

# Examples
docker push myregistry.example.com/tools/myapp:0.1.0
docker push myregistry.example.com/superapp:0.1.0
```

#### Builders

[Docker builders](https://docs.docker.com/build/builders/?ref=hackerstack.org) | [BuildKit](https://docs.docker.com/build/buildkit/?ref=hackerstack.org)

-   Docker uses [builders](https://docs.docker.com/build/builders/?ref=hackerstack.org) under the hood when you build container images. The builder in Docker is called [BuildKit](https://docs.docker.com/build/buildkit/?ref=hackerstack.org). It comes bundled with the Docker Engine and is used by default to build images.
    
-   Buildx can be used to extend build capabilities with BuildKit. There are other [build drivers](https://docs.docker.com/build/builders/drivers/?ref=hackerstack.org) you can configure through buildx when creating new builders.
    

Here are sample commands for managing Docker builders with buildx:

-   List builders:

```
$ docker buildx ls
NAME/NODE     DRIVER/ENDPOINT   STATUS    BUILDKIT   PLATFORMS
default*      docker
 \_ default    \_ default       running   v0.21.0    linux/amd64 (+4), linux/386
```

-   Create a new builder that uses the 'docker-container' build driver. This will run a BuildKit container for each build instead of using the BuildKit instance bundled inside the Docker Engine:

```
# Create the new builder
$ docker buildx create \
  --name container-builder \
  --driver docker-container

# Verify
$ docker buildx ls
NAME/NODE                DRIVER/ENDPOINT                   STATUS     BUILDKIT   PLATFORMS
container-builder        docker-container
 \_ container-builder0    \_ unix:///var/run/docker.sock   inactive
default*                 docker
 \_ default               \_ default                       running    v0.21.0    linux/amd64 (+4), linux/386
```

-   Set the newly created builder called 'container-builder' as the default builder. The builder to use can also be selected by setting its name into the [BUILDX\_BUILDER](https://docs.docker.com/build/building/variables/?ref=hackerstack.org#buildx_builder) environment variable or by using the '--builder' flag when running the 'docker build' or 'docker buildx build' command.

```
# Set the newly created builder as the default one
$ docker buildx use container-builder

# Verify
$ docker buildx ls
NAME/NODE                DRIVER/ENDPOINT                   STATUS     BUILDKIT   PLATFORMS
container-builder*       docker-container
 \_ container-builder0    \_ unix:///var/run/docker.sock   inactive
default                  docker
 \_ default               \_ default                       running    v0.21.0    linux/amd64 (+4), linux/386
 
# The asterix (*) indicates
# the default builder
```

#### Image layering / caching

Each instruction inside a Dockerfile will create a new layer on top of  
the current image. Understanding how Docker build caches work can be helpful to faster container image builds.

Here is the link to learn about [Docker Build Cache](https://docs.docker.com/build/cache/?ref=hackerstack.org).

### Common Dockerfile instructions

[Dockerfile reference](https://docs.docker.com/reference/dockerfile/?ref=hackerstack.org) | [Dockerfile instructions best practices](https://docs.docker.com/build/building/best-practices/?ref=hackerstack.org#dockerfile-instructions)

#### FROM

[Instruction documentation](https://docs.docker.com/reference/dockerfile/?ref=hackerstack.org#from)

The FROM instruction can be used to initialize a new build stage and set the base image for subsequent instructions.

If another FROM is used inside the same Dockerfile, that will create another image and a [multi-stage build](https://docs.docker.com/build/building/multi-stage/?ref=hackerstack.org). Image assets from previous build stages could then be reused inside following build stages.

```
# Syntax
FROM [--platform=<platform>] <image>[:<tag> | @digest] [AS <name>]
```

-   The platform is in the form 'os/arch', for instance:
    -   'linux/amd64' or 'windows/arm64'
-   The name given to the stage could then be used as follows:

```
# Create a new build stage using the previous stage image as the base image
FROM <name>

# Copy a file from a specific named build stage into this current stage image
COPY --from=<name> <src> ... <dest>

# Inject files or directories from a specific named 
# build stage into this current stage image
RUN --mount=type=bind,from=<name>,target=<target_path>
```

#### COPY

[Instruction documentation](https://docs.docker.com/reference/dockerfile/?ref=hackerstack.org#copy)

COPY can be used to copy files from the build context or from a [multi-stage build](https://docs.docker.com/build/building/multi-stage/?ref=hackerstack.org) stage to the filesystem of the image. The copied files will remain in the final image. To add files from the build context only temporarily for a [RUN](#run) command for instance, use bind mounts as follows:

```
RUN --mount=type=bind,source=requirements.txt,target=/tmp/requirements.txt \
    pip install --requirement /tmp/requirements.txt
```

Bind mounts are more efficient than COPY. Here are examples using COPY:

```
# Copy from the buid context
COPY --chown=www-data:www-data --chmod=755 conf/php-config.ini "$PHP_INI_DIR/conf.d/"

# Copy from a build stage (in multi-stage build)
COPY --chown=www-data:www-data --chmod=755 --from=replace conf/nginx.conf $NGINX_CONF_PATH
```

#### ADD

[Instruction documentation](https://docs.docker.com/reference/dockerfile/?ref=hackerstack.org#add)

In addition to copying files from the build context to the filesystem of the image, ADD is also capable of downloading files from remote HTTPS and Git URLs, very files checksums, and more. It is also capable of extracting tar files automatically when adding them into image filesystems from the build context.

Here are examples of using ADD:

```
# Add a file from a remote HTTPS URL into the image
ADD https://example.com/myfile.zip /files/

# Add files from a remote Git repository into the image
ADD git@my.repo.example:web/app.git /app
```

#### ARG

[Instruction documentation](https://docs.docker.com/reference/dockerfile/?ref=hackerstack.org#arg)

ARG can be used for build time variables. Build time variables are used only during image build process and not persisted into the final image. Also, the ARG instruction is not suitable for storing sensitive informations (API keys, passwords, etc). One reason for that is because they will be visible when using the 'docker history' command.

Here are examples of using ARG:

```
# Declare one or many build time variables
ARG myvar1=myvalue1 MYVAR2=myvalue2 MYVAR3
```

During the build process, you can also set or override previously declared build time variables as follows:

```
$ docker build -t myapp:1.0.1 \
   --build-arg myvar1=myvalue2 \
   --build-arg MYVAR3=myvalue3
```

If you need to inject sensitive data into the build context for running a specific command, use [RUN --mount=type=secret](https://docs.docker.com/reference/dockerfile/?ref=hackerstack.org#run---mounttypesecret).

Here are examples of injecting secret data into the build context:

-   [inject a file containing secret data](https://docs.docker.com/reference/dockerfile/?ref=hackerstack.org#example-access-to-s3)
-   [inject an environment variable containing secret data](https://docs.docker.com/reference/dockerfile/?ref=hackerstack.org#example-mount-as-environment-variable)

Here are examples to help you understand the scopes of build time variables:

```
# These variables have a global scope. They will be available for all 
# the FROM instructions but not inside the build stages themselves.
ARG IMAGE_STAGE1=ubuntu
ARG IMAGE_STAGE2=debian

FROM ${IMAGE_STAGE1} AS stage1 # variable accessible here
# Should be redeclared inside this stage in order to use it
ARG IMAGE_STAGE1
RUN echo "Hello from ${IMAGE_STAGE1}"

FROM ${IMAGE_STAGE2} AS stage2 # variable accessible here
# Should be redeclared inside this stage in order to use it
ARG IMAGE_STAGE2
RUN echo "Hello from ${IMAGE_STAGE2}"
```

#### ENV

[Instruction documentation](https://docs.docker.com/reference/dockerfile/?ref=hackerstack.org#env)

The ENV instruction can be used to set environment variables that are available for subsequent intructions inside the Dockerfile (like ARG), but, unlike ARG, the variables will also be available inside the environment of the containers created using the resulting image.

```
# Syntax
ENV <varname>=<value> [<key>=<value>...]
```

Such variables can be overriden when launching containers with the 'docker run' command, using the '--env varname=value' or '-E varname=value' flags.

#### RUN

[Instruction documentation](https://docs.docker.com/reference/dockerfile/?ref=hackerstack.org#run)

-   The RUN instruction is used to execute commands that are required to construct the image.
-   The executed command creates a new layer on top of the current image.
-   That layer is then used in the next Dockerfile step.

Here is the syntax:

```
# Shell form:
RUN [OPTIONS] <command> ...

# Exec form:
RUN [OPTIONS] [ "<command>", ... ]
```

Here are some usage examples:

```
# Using heredocs

RUN <<EOF
apt update
apt install -y curl zip
EOF

# Using escape

RUN apt update && \
    apt install -y \
    curl \
    zip
```

#### ENTRYPOINT and CMD

[ENTRYPOINT](https://docs.docker.com/reference/dockerfile/?ref=hackerstack.org#entrypoint) | [CMD](https://docs.docker.com/reference/dockerfile/?ref=hackerstack.org#cmd)

These two instructions can be used to execute the first command of the container. That command will run during the container startup and will create a process that has the PID 1. Here are examples that will help you understand these instructions.

Let's build the example1 image, run a container using that image and see what happens:

```
# Example1 Dockerfile content
FROM ubuntu
CMD ["echo", "hello"]

# Build the example1 image
$ docker build -t example1 .
[+] Building 0.2s (5/5) FINISHED                                                                                                                                             docker:default
 => [internal] load build definition from Dockerfile                                                                                                                                   0.0s
 => => transferring dockerfile: 198B                                                                                                                                                   0.0s
 => [internal] load metadata for docker.io/library/ubuntu:latest                                                                                                                       0.1s
 => [internal] load .dockerignore                                                                                                                                                      0.0s
 => => transferring context: 2B                                                                                                                                                        0.0s
 => CACHED [example1 1/1] FROM docker.io/library/ubuntu:latest@sha256:66460d557b25769b102175144d538d88219c077c678a49af4afca6fbfc1b5252                                                 0.0s
 => exporting to image                                                                                                                                                                 0.0s
 => => exporting layers                                                                                                                                                                0.0s
 => => writing image sha256:36423e8273020c1fc9186f698397a8d4524b46ecd0b638c4a3315ce1028ffcd6                                                                                           0.0s
 => => naming to docker.io/library/example1                                                                                                                                            0.0s

# Launch a container using that image
$ docker run example1
hello
```

The container runs, outputs 'hello' as expected and exits because the command has terminated. If we launch the container again and specify another command to run, that latter command will override the one defined inside the Dockerfile CMD instruction:

```
$ docker run example1 cat /etc/issue
Ubuntu 24.04.3 LTS \n \l
```

* * *

Now let's build and run the example2 image using ENTRYPOINT instead of CMD and see the difference:

```
# Example2 Dockerfile content
FROM ubuntu
ENTRYPOINT ["echo", "hello"]

# Build the example2 image
$ docker build -t example2 .
[+] Building 0.2s (5/5) FINISHED                                                                                                                                             docker:default
 => [internal] load build definition from Dockerfile                                                                                                                                   0.0s
 => => transferring dockerfile: 198B    
(...)

# Launch a container using that image
$ docker run example2
hello
```

Again, the container runs, outputs 'hello' as expected and exits because the command has terminated. If we launch the container again and specify another command to run, that latter command will not override the one defined inside the Dockerfile ENTRYPOINT instruction. Instead, that command will be added as an argument to the ENTRYPOINT command:

```
$ docker run example2 cat /etc/issue
hello cat /etc/issue
```

The example2 container runs the 'echo hello cat /etc/issue' command instead of replacing the 'echo hello' command by 'cat /etc/issue' as seen in example1. It is however possible to reset the ENTRYPOINT during the container launch and run a new command if required:

```
$ docker run --entrypoint="" example2 cat /etc/issue
Ubuntu 24.04.3 LTS \n \l
```

By resetting the ENTRYPOINT, we are able to run the new 'cat /etc/issue' command as in example1.

* * *

If you are going to containerize a CLI program for instance, you should prefer using ENTRYPOINT over CMD because it will provide nearly the same feeling to your users:

```
# Running a containerized version of mycli program
docker run mycli --help
docker run mycli <ARGS>
```

You run the image and directly use the CLI program flags and arguments that you want, no need to call the program again.

* * *

Another example I want to show you is the effect of using a combination of ENTRYPOINT and CMD inside the Dockerfile:

```
# Example3 Dockerfile content
FROM ubuntu
ENTRYPOINT ["echo"]
CMD ["hello"]

# Build the example3 image
docker build -t example3 .
[+] Building 3.3s (5/5) FINISHED                                                                                                                                             docker:default
 => [internal] load build definition from Dockerfile                                                                                                                                   0.0s
 => => transferring dockerfile: 83B                                                                                                                                                    0.0s
 => [internal] load metadata for docker.io/library/ubuntu:latest                                                                                                                       0.6s
 => [internal] load .dockerignore                                                                                                                                                      0.0s
 => => transferring context: 2B                                                                                                                                                        0.0s
 => [1/1] FROM docker.io/library/ubuntu:latest@sha256:66460d557b25769b102175144d538d88219c077c678a49af4afca6fbfc1b5252                                                                 2.5s
 => => resolve docker.io/library/ubuntu:latest@sha256:66460d557b25769b102175144d538d88219c077c678a49af4afca6fbfc1b5252                                                                 0.0s
 => => sha256:d22e4fb389065efa4a61bb36416768698ef6d955fe8a7e0cdb3cd6de80fa7eec 424B / 424B                                                                                             0.0s
 => => sha256:97bed23a34971024aa8d254abbe67b7168772340d1f494034773bc464e8dd5b6 2.30kB / 2.30kB                                                                                         0.0s
 => => sha256:4b3ffd8ccb5201a0fc03585952effb4ed2d1ea5e704d2e7330212fb8b16c86a3 29.72MB / 29.72MB                                                                                       0.5s
 => => sha256:66460d557b25769b102175144d538d88219c077c678a49af4afca6fbfc1b5252 6.69kB / 6.69kB                                                                                         0.0s
 => => extracting sha256:4b3ffd8ccb5201a0fc03585952effb4ed2d1ea5e704d2e7330212fb8b16c86a3                                                                                              1.8s
 => exporting to image                                                                                                                                                                 0.0s
 => => exporting layers                                                                                                                                                                0.0s
 => => writing image sha256:16805ffadd43c7f568fb26de593c3baf2b4c756c0be65af397c7ec5a2581681c                                                                                           0.0s
 => => naming to docker.io/library/example3
 
# Launch a container using that image
$ docker run example3
hello
```

In this example, the argument to the 'echo' command which is 'hello', is not part of the ENTRYPOINT command, but defined inside another CMD command. Using it that way sets 'hello' as the default argument to the command defined with the ENTRYPOINT instruction. So when we run the example3 container, we get the same result as before: it outputs 'hello'. Users of this image will be able to easily override the default argument as shown below:

```
$ docker run example3 Welcome to hackerstack.org
Welcome to hackerstack.org
```

* * *

Last thing I want to show you is how the state of the first command of the container and the way it is run (in foreground or background) impact the state of the container itself.

Here is the content of the Dockerfile we will be using for all the examples:

```
FROM ubuntu
COPY script.sh /opt
ENTRYPOINT ["/opt/script.sh"]
```

We will only change the content of 'script.sh' between examples.

-   The container's first command exits successfully. The state of the container indicated in the 'STATUS' column of the 'docker ps' command is:
    
    -   'Exited (0)'

```
# Content of script.sh

#!/bin/bash

echo "Exiting with exit code 0"
exit 0

# After building example image with that script

$ docker run example
Exiting with exit code 0

$ docker ps -a
CONTAINER ID   IMAGE          COMMAND             CREATED          STATUS                        PORTS     NAMES
fd33afbaf973   example        "/opt/script.sh"    8 seconds ago    Exited (0) 8 seconds ago                clever_northcutt
```

-   The container's first command exits unsuccessfully. The state of the container indicated in the 'STATUS' column of the 'docker ps' command is:
    
    -   'Exited (1)'

```
# Content of script.sh

#!/bin/bash

echo "Exiting with exit code 1"
exit 1

# After building example image with that script

$ docker run example
Exiting with exit code 1

$ docker ps -a
CONTAINER ID   IMAGE          COMMAND             CREATED          STATUS                        PORTS     NAMES
84fc7e7854ed   example        "/opt/script.sh"    4 seconds ago    Exited (1) 3 seconds ago                angry_lichterman
```

-   The container's first command keeps running in the foreground. The state of the container indicated in the 'STATUS' column of the 'docker ps' command is:
    
    -   'Up' (it is running) followed by an uptime duration

```
# Content of script.sh

#!/bin/bash

echo "Keeping a command running in foreground"
sleep 300 # 300 secondes

# After building example image with that script

$ docker run example
Keeping a command running in foreground
# the command docker command also run in the 
# foreground and block shell interaction

# To run the docker command in background, we can
# use the -d flag as follows

$ docker run -d example
129bab01dc1a1b3529aa620943d432fcd7ce7c5b882cc0575afb01f5bb9f8e05

# Show running containers

$ docker ps
CONTAINER ID   IMAGE     COMMAND            CREATED         STATUS         PORTS     NAMES
129bab01dc1a   example   "/opt/script.sh"   2 minutes ago   Up 2 minutes             admiring_shamir
```

-   The container's first command, again, keeps running, but this time, in the background. The state of the container indicated in the 'STATUS' column of the 'docker ps' command is:
    
    -   'Exited (0)'

```
#!/bin/bash

echo "Keeping a command running in foreground"
sleep 300 &

$ docker run -d example
638bb58dd066da0cc43921bf0739eb9763f1ff14e46b85d06eb2fc6f66f5ea64

$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

$ docker ps -a
CONTAINER ID   IMAGE          COMMAND            CREATED          STATUS                        PORTS     NAMES
638bb58dd066   example        "/opt/script.sh"   7 seconds ago    Exited (0) 6 seconds ago                crazy_nash
```

-   In all the previous examples, text messages sent to the standard output (stdout) or standard error (stderr) with the 'echo' command or whatever, will be visible inside the container's logs:

```
$ docker logs 638bb58dd066
Keeping a command running in foreground
```

* * *

[ENTRYPOINT](https://docs.docker.com/reference/dockerfile/?ref=hackerstack.org#entrypoint) | [CMD](https://docs.docker.com/reference/dockerfile/?ref=hackerstack.org#cmd)

To end this section, here are important rules to keep in mind when working with ENTRYPOINT and CMD:

-   At least one ENTRYPOINT or CMD command is mandatory inside a Dockerfile.
    
-   If there are multiple CMD instructions, only the last one will run.
    
-   ENTRYPOINT can be used to set a default executable for the container. Any command that is run through that container will be an argument to the defaut executable set with ENTRYPOINT.
    
-   An ENTRYPOINT instruction can be followed by a CMD instruction, to set default arguments for the executable defined with ENTRYPOINT. A user can override the default arguments defined with CMD by passing new ones when lauching the container.
    
-   The state of a container is linked to the state of its first process: the process with PID 1.
    
-   If there is no ENTRYPOINT instruction, the process with PID 1 inside the container will be created from the executable specified inside the last CMD instruction.
    
-   If there is an ENTRYPOINT instuction, the process with PID 1 inside the container will be created from the executable specified inside the ENTRYPOINT instruction.
    
-   The logs we will get from the container (with 'docker logs' or whatever) will come from the container's standard output (stdout) and standard error (stderr).
    
-   The foreground notion for the first process of the container is very important because it determines if your container keeps running or not. Container's process with PID 1 (created with ENTRYPOINT or CMD) is runnning in the foreground = container running, otherwise, container creates and exits.
    

One common way for running daemon services inside containers is by using a process control system like [supervisord](https://supervisord.org/?ref=hackerstack.org) as explained [here](#running-multiple-services-inside-the-same-container-using-supervisord).

#### WORKDIR

[Instruction documentation](https://docs.docker.com/reference/dockerfile/?ref=hackerstack.org#workdir)

WORKDIR can be used to define the working directory for commands like RUN, COPY, CMD, ENTRYPOINT, ADD that follow it inside the Dockerfile. The directory specified will be created if it does not exist. Usage example:

```
WORKDIR /app
```

#### USER

[Instruction documentation](https://docs.docker.com/reference/dockerfile/?ref=hackerstack.org#user)

USER can be used to set the user (and optionally the group) to use for the RUN instructions that follow it inside the Dockerfile.

This will also be the default user inside the container, and will be used to run scripts/commands specified inside the Dockerfile with ENTRYPOINT or CMD instructions that follow it. Usage examples:

```
# Using username:groupname
# groupname is optional
USER gmkziz:admin

# Using UID:GID
# GID is optional
USER 1001:900
```

#### EXPOSE

[Instruction documentation](https://docs.docker.com/reference/dockerfile/?ref=hackerstack.org#expose)

The EXPOSE instruction can be used as an information to highlight a specific port on which the container will be listening. It doesn't actually publish the port.

Here is an example usage:

```
(...)

EXPOSE <port>
```

To publish the exposed port on the host when running the container, you can use the '-P <host\_port>:<container\_port>' of the 'docker run' command.

### Dockerfile examples and tips

#### Replacing variables inside config files by environment variables

This example leverages Docker [multi-stage build](https://docs.docker.com/build/building/multi-stage/?ref=hackerstack.org) to keep the image small.

Imagine you need to inject a configuration file inside the image, which content contains a variable that is declared inside the Dockerfile for use by different build stages.

```
ARG APP_BASE_DIR=/app
FROM myimage
ARG APP_BASE_DIR
COPY conf/myfile.conf /etc/nginx/http.d/default.conf
(...)
```

The APP\_BASE\_DIR variable is declared once inside the Dockerfile and contains the path to the application base directory.

The content of the configuration file called 'myfile.conf' that we are copying into the container should also contain the value of that same APP\_BASE\_DIR variable.

To achieve that, we can create a template of the configuration file locally. Here is an example content:

```
# File: myfile.conf.tpl
(...)
root {{ env "APP_BASE_DIR" }};
(...)
```

Then inside the Dockerfile, we create a temporary stage to perform the variable replacement using the [go-replace](https://github.com/webdevops/go-replace/tree/master?ref=hackerstack.org) tool:

```
(...)
FROM webdevops/go-replace AS replace
ARG APP_BASE_DIR
WORKDIR /conf
RUN --mount=type=bind,source=./myfile.conf.tpl,target=myfile.conf.tpl,rw \
    --mount=type=cache,target=/tmp/cache \
    go-replace --mode=template myfile.conf.tpl:myfile.conf
```

This will create the '/conf/myfile.conf' file inside that stage by replacing:

-   `{{ env "APP_BASE_DIR" }}` by '/app'.

You can then copy that final configuration file into the image at next stages as follows:

```
COPY --from=replace conf/myfile.conf /etc/nginx/http.d/default.conf
```

#### Running multiple services inside the same container using supervisord

One common way for running multiple services inside containers in a reliable way is by using a process control system like [supervisord](https://supervisord.org/?ref=hackerstack.org) that will run as the ENTRYPOINT (PID 1) and launch the required services (for instance: nginx and php-fpm).

Here is a sample Dockerfile snippet using supervisord for starting multiple services through an ENTRYPOINT script:

```
(...)
RUN apk update && apk add --no-cache supervisor
COPY --chown=www-data:www-data --chmod=755 conf/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY --chown=www-data:www-data --chmod=755 --from=replace conf/entrypoint.sh /usr/local/bin/entrypoint.sh
ENTRYPOINT ["entrypoint.sh"]
```

Here is an example content for the ENTRYPOINT script:

```
# file: /usr/local/bin/entrypoint.sh

#!/bin/sh
set -e

# Start supervisord and pass it the configuration file
exec /usr/bin/supervisord -c /etc/supervisor/conf.d/supervisord.conf
```

In a container running php-fpm and nginx for instance, [supervisord](https://supervisord.org/?ref=hackerstack.org) can be configured to launch the php-fpm in the background and nginx in the foreground. Also, it can be configured in a way that avoid saving its logs inside the container filesystem:

```
# File: /etc/supervisor/conf.d/supervisord.conf

[supervisord]
nodaemon=true
logfile=/dev/null
logfile_maxbytes=0

[program:php-fpm]
command=php-fpm
autostart=true
autorestart=true
priority=5
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0

[program:nginx]
command=nginx -g 'daemon off;'
autostart=true
autorestart=true
priority=10
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0
```

With this configuration, the container will be considered running as long as nginx is running and nginx logs that are sent to stdout will show when viewing the container's logs with the 'docker logs' command for instance. Here is a sample configuration file for nginx:

```
upstream php {
    server localhost:9000;
}

server {
        listen 80; 
        server_name localhost;
        root /app;
        server_tokens off;

        location / { 
                try_files $uri /index.php$is_args$args;
        }

        location ~ \.php(/|$) {
                include fastcgi_params;
                fastcgi_pass php;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                error_log /dev/stdout;
                access_log /dev/stderr;
        }
}
```

Most importantly [supervisord](https://supervisord.org/?ref=hackerstack.org) will automatically restart php-fpm or nginx processes in case of failure.

#### Using multi-stage builds

Here is an example with a PHP application where development and production dependencies are built inside different stages.

The results from those stages are then reused at development and production stages, from where the app development or production image will be built.

The development or production stage will be created from a common stage called base containing things that should be available for development and production (php and php extensions, app source code, etc).

This allows us to independently build development and production images, each image containing only the dependencies it needs (smaller image).

Here is the stage for development dependencies:

```
FROM composer:lts AS dev-deps
WORKDIR /app
RUN --mount=type=bind,source=./src/composer.lock,target=composer.lock \
    --mount=type=bind,source=./src/composer.json,target=composer.json \
    --mount=type=cache,target=/tmp/cache \
    composer install --ansi --no-scripts --no-progress
```

And the stage for production dependencies:

```
FROM composer:lts AS prod-deps
WORKDIR /app
RUN --mount=type=bind,source=./src/composer.lock,target=composer.lock \
    --mount=type=bind,source=./src/composer.json,target=composer.json \
    --mount=type=cache,target=/tmp/cache \
    composer install --ansi --no-scripts --no-progress --no-dev
```

The two previous stages will create a 'vendor' directory in '/app' containing the required dependencies files. We will inject the corresponding 'vendor' directory inside the final stages for development and production images.

Now, here is the base stage that is common to both development and production environment images:

```
(...)
FROM php:8.4.14-fpm-alpine3.21 AS base
ARG APP_BASE_DIR
RUN apk update && apk add --no-cache \
      build-base \
      git \
      (...)
RUN docker-php-ext-install \
      gd \
      xml \
      (...)
COPY --chown=www-data:www-data --chmod=755 ./src "$APP_BASE_DIR"
(...)
WORKDIR $APP_BASE_DIR
```

And finally, the stage that will be used to build the development image:

```
FROM base AS developement
RUN mv "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini"
COPY --chown=www-data:www-data --chmod=755 --from=dev-deps app/vendor /app/includes/vendor
ENTRYPOINT ["entrypoint.sh"]
EXPOSE 80
```

And the one for the production image:

```
FROM base AS production
RUN mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"
COPY --chown=www-data:www-data --chmod=755 --from=prod-deps app/vendor /app/includes/vendor
ENTRYPOINT ["entrypoint.sh"]
EXPOSE 80
```

### Managing containers with Docker

#### Launching containers

-   For details about the syntax and options, use:

```
docker run --help
```

-   Run a container using the busybox image to test connectivity to 'google.com' using the 'nc' command:

```
docker run busybox nc -vz google.fr 80
```

-   Ensure the container is removed from the containers list (shown with 'docker ps') after execution:

```
docker run --rm busybox nc -vz google.fr 80
```

-   Give a name to the container:

```
docker run --name debug busybox nc -vz google.fr 80
```

-   Pass environment variables to the container:

```
$ docker run -e MYVAR=VALUE busybox env | grep MYVAR
MYVAR=VALUE
```

-   Mount a volume into the container:

```
$ docker run -v "$PWD/Test:/test" busybox ls /test
hello
```

-   Launch a container and access a shell inside it. For that, simply run that specific shell command (bash, sh, etc) when launching the container and use the '-it' flag:

```
# -i, --interactive: Keep STDIN open even if not attached
# -t, --tty: Allocate a pseudo-TTY

$ docker run -it busybox sh
/ # env
HOSTNAME=841da4299bb8
SHLVL=1
HOME=/root
TERM=xterm
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
PWD=/
```

#### Executing commands inside running containers

-   For details about the syntax and options, use:

```
docker exec --help
```

-   Execute the 'ls' command inside a running container called myapp:

```
docker exec myapp ls
```

-   Get a shell inside a running container called myapp:

```
# Sh
docker exec -it myapp sh

# Bash
docker exec -it myapp bash
```

#### Listing containers and images

-   For details about the command syntax and options, use:

```
# Help on listing containers
docker ps --help

# Help on listing container images
docker images --help
```

-   List running containers

```
docker ps
```

-   List all containers (running or not):

```
docker ps -a
```

-   List container images

```
docker images
```

#### Inspecting containers

-   For details about the syntax and options, use:

```
docker inspect --help
```

-   Inspect the container named myapp:

```
docker inspect myapp
```

The 'docker inspect' command is not limited to containers and can also be used for other docker objects like images, volumes, etc:

```
# Inspecting a docker image
docker inspect myimage

# Inspecting a docker volume
docker inspect myvolume
```

#### Viewing containers logs

-   For details about the syntax and options, use:

```
docker logs --help
```

-   Show all the available logs lines for the container named myapp:

```
docker logs myapp
```

-   Show only the last 10 lines of logs for the container named myapp:

```
docker logs -n 10 myapp
```

-   Follow the logs output stream for the container named myapp starting at the 10 last output lines:

```
docker logs -n 10 -f myapp
```

#### Limit containers output logs file size

The output of the docker logs command is by default taken from a file stored at:

-   `/var/lib/docker/containers/<container_id>/<container_id>-json.log`

To limit the size of that file on the Docker host/machine filesystem, the following directives can be added inside the docker daemon configuration file:

```
# File: /etc/docker/daemon.json

{
  (...)
  "log-opts": {
    "max-size": "<max_size_of_log_files>",
    "max-file": "<max_number_of_log_files>",
  }
  (...)
}
```

Have a look at [Configure docker default logging driver](https://docs.docker.com/engine/logging/configure/?ref=hackerstack.org#configure-the-default-logging-driver) for more.

#### Viewing containers resources consumption

-   For details about the syntax and options, use:

```
docker stats --help
```

-   Display a live stream of containers resources usage statistics:

```
docker stats
```

#### System info and unused data cleanup

```
# Info about the Docker client (version, mode, plugins, etc) and the Docker daemon
docker system info

# Info about data size used by images,
# containers, volumes, build caches
docker system df

# Remove unused data: images, containers,
# volumes, build caches
docker system prune [--all, --force]

# Cleanup only unused images, containers, volumes, etc
docker image|container|volume prune [--force]
```

#### Use a proxy for pulling docker images

You have to configure the proxy at the docker daemon level. We will demonstrate one way of doing this, if you are using systemd as the system services manager.

Create the following drop-in configuration file:

```
# File: /etc/systemd/system/docker.service.d/docker-service-override.conf

[Service]
Environment="https_proxy=<proxy_ip>:<proxy_port>"
```

Then, make systemd daemon aware of the docker service configuration file change by running the following command:

```
systemctl daemon-reload
```

Then, restart the docker daemon using the following command:

```
systemctl restart docker
```

#### Configure a proxy for all docker containers

This will make all your running docker containers use the configured proxy for their http/https requests.

Add the following to the docker client configuration file:

```
# File: $HOME/.docker/config.json

{
  "proxies": {
    "default": {
      "httpProxy": "<proxy_ip>:<proxy_port>",
      "httpsProxy": "<proxy_ip>:<proxy_port>"
    }
  }
}
```

### Managing containers with Docker Compose

#### What is Docker Compose?

[Docker Compose](https://docs.docker.com/compose/?ref=hackerstack.org)

Docker Compose is an additional tool that can be used to manage Docker containers declaratively.

Instead of running commands to run Docker containers individually, you create a configuration file where you declare the containers you want to run, with the parameters (volumes, env vars, etc) you want them to run with.

To install Docker Compose on Linux, have a look at [this](https://docs.docker.com/compose/install?ref=hackerstack.org#plugin-linux-only). It is available as a plugin for Docker and can be run through the 'docker compose' command.

#### Creating a Docker Compose file

[Docker Compose file reference](https://docs.docker.com/reference/compose-file?ref=hackerstack.org)

The most commonly used top-level section of a Docker Compose file is [service](https://docs.docker.com/reference/compose-file/services/?ref=hackerstack.org). That's where you declare you containers and their parameters. Here is an example:

```
services:
  myapp-frontend:
    image: myapp
    container_name: myapp-frontend
    volumes:
      - source: $PWD/.env
        type: bind
        read_only: true
        target: /app/config/.env
    environment:
      DEBUG: false
      DATABASE_HOST: myapp-database
    ports:
      - "8000:80"

  myapp-database:
    image: mysql:8.0.43-bookworm
    container_name: myapp-database
    environment:
      MYSQL_ROOT_PASSWORD: "mysuperpass"
      MYSQL_DATABASE: "myapp"
    volumes:
      - source: /var/tmp/myapp-db-data
        type: bind
        read_only: false
        target: /var/lib/mysql
    depends_on:
      - myapp-frontend
```

Inside the [service](https://docs.docker.com/reference/compose-file/services?ref=hackerstack.org) top-level section, here is the documentation of some of the other directives I want to highlight:

-   [deploy](https://docs.docker.com/reference/compose-file/deploy?ref=hackerstack.org): configure resources requests and limits, number of replicas, etc.
    
-   [volumes](https://docs.docker.com/reference/compose-file/services/?ref=hackerstack.org#volumes): mount volumes inside containers using bind mounts, docker volumes, data from containers images, etc.
    
-   [healthcheck](https://docs.docker.com/reference/compose-file/services/?ref=hackerstack.org#healthcheck): configure healthchecks for your containers.
    
-   [dns](https://docs.docker.com/reference/compose-file/services/?ref=hackerstack.org#dns): configure DNS servers adresses to use for DNS resolution inside your containers.
    
-   [depends\_on](https://docs.docker.com/reference/compose-file/services/?ref=hackerstack.org#depends_on): declare dependencies between your docker compose services.
    
-   [build](https://docs.docker.com/reference/compose-file/services/?ref=hackerstack.org#build): build containers images at launch, from local source code directories.
    
-   [develop](https://docs.docker.com/reference/compose-file/services/?ref=hackerstack.org#develop): useful for local development to maintain container in sync with app source code changes.
    

#### Development Docker Compose file

The main goal of a local Docker-compose file is to make easier to launch the app locally to fix bugs or add new features.

Here is what we want to achieve with a local development Docker-compose file:

-   automatically build the development image from local app source code and run it
    
-   ensure local app source code changes are synchronized into the docker container
    
-   expose the relevant application ports to local machine in order to access the app and make tests during developement
    
-   automatically run required database and inject necessary data for the app
    

```
services:
  myapp-frontend:
    # Build image from source code and Dockerfile 
    # present inside the current directory (.)
    # Use 'dev' as the Dockerfile target to build the image
    build:
      context: .
      target: dev
    container_name: myapp-frontend
    # Mount required config files inside the container
    volumes:
      - source: $PWD/.env
        type: bind
        read_only: true
        target: /app/config/.env
    # Set environment variables for the container
    environment:
      DEBUG: false
      DATABASE_HOST: myapp-database
    # Expose the container port 80 on local machine port 8080
    ports:
      - "8000:80"
    # Make this service run only when the database service is ready
    depends_on:
      - myapp-database
    # Keep the /app directory inside the  container in sync
    # with the local src directory containing the app source code
    develop:
      watch:
        - action: sync
          path: ./src
          target: /app

  myapp-database:
    image: mysql:8.0.43-bookworm
    container_name: myapp-database
    environment:
      MYSQL_ROOT_PASSWORD: "mysuperpass"
      MYSQL_DATABASE: "myapp"
    volumes:
      # Initialize the database with a dump file
      # when running for the first time
      - source: /var/tmp/dump.sql
        type: bind
        read_only: false
        target: /docker-entrypoint-initdb.d/dump.sql
      # Make database data persist on the local machine disk
      - source: /var/tmp/myapp-db-data
        type: bind
        read_only: false
        target: /var/lib/mysql
```

#### Running containers with Docker Compose

-   Start containers using docker compose

```
# If the docker-compose.yaml configuration is 
# located insite the current directory

# Start in the foreground
docker compose up

# When using the docker-compose.yaml
# develop.watch configuration directive
# you shoud use the --watch flag
docker compose up --watch

# Start in the background
docker compose up -d

# If the docker-compose.yaml configuration 
# is located elsewhere, use the -f option to 
# specify the path to the configuration file
docker compose -f $compose-config-file up
docker compose -f $compose-config-file up -d
```

-   Stop and restart containers using docker compose

```
# Stop
docker compose [-f $compose-config-file] stop

# Restart
docker compose [-f $compose-config-file] restart
```

* * *
