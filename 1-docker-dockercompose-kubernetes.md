# Docker, Docker Compose, & Kubernetes

## Tutorial videos 

- [The intro to Docker I wish I had when I started](https://www.youtube.com/watch?v=Ud7Npgi6x8E)
- [The Only Docker Tutorial You Need To Get Started](https://www.youtube.com/watch?v=DQdB7wFEygo)
- [Docker Compose Tutorial](https://www.youtube.com/watch?v=HG6yIjZapSA)
- [Docker Compose will BLOW your MIND!! (a tutorial)](https://www.youtube.com/watch?v=DM65_JyGxCo)
- [Top 8 Docker Best Practices for using Docker in Production](https://www.youtube.com/watch?v=8vXoMqWgbQQ)
- [Docker Image BEST Practices - From 1.2GB to 10MB](https://www.youtube.com/watch?v=t779DVjCKCs)
- [What is Kubernetes | Kubernetes explained in 15 mins](https://www.youtube.com/watch?v=VnvRFRk_51k)
- [Kubernetes Crash Course for Absolute Beginners [NEW]](https://www.youtube.com/watch?v=s_o8dwzRlu4)

## Docker

Docker essentially allows us to "Package up" our code and run it anywhere, on any machine!

### Container introduction

**Containers**: They are a way to build reprodudicable lightweight environments for processes to run
- We use containers everywhere, even without knowing about it
- Almost anything deployed to the cloud has some form of containerization to hold our application

---

**What is a host?**
- A host is a "computer" somewhere...
    - It can be within your PC
    - It can be a computer in a data center
    - It can be a host connected to the cloud and made accessible to you by the cloud provider
- This, in fact also means, that a host is a "piece of hardware". It will have the following components:
    1. CPU
    2. Memory
    3. I/O (Ex: File system on a hard drive, N/W cards, etc)


```
       _________________________________________________
      |                                                 |
      |                 HOST (The Computer)             |
      |_________________________________________________|
      |                                                 |
      |   [ 1. CPU ] <-----------> [ 2. MEMORY ]        |
      |  (Processing)             (RAM / Storage)       |
      |        ^                         ^              |
      |________|_________________________|______________|
               |                         |
               v                         v
      |_________________________________________________|
      |                                                 |
      |              3. I/O COMPONENTS                  |
      |_________________________________________________|
      |        |                        |               |
      | [ File System ]          [ Network Card ]       |
      |   (Data/OS)              (Cloud/Internet)       |
      |_________________________________________________|
               |                         |
               |                         |
      _________v_________       _________v_________
     /                   \     /                   \
    |     Hard Drive      |   |   Remote Access     |
    \_____________________/    \___________________/
```

**Virtualization and Virtual Machines**
- We take "little pieces" of the host's CPU, Memory, and I/O like hardware and separate them out into a "virtual" machine
- The Virtual Machine (VM) then runs a whole, i.full, Operating System (OS) on top of it
- It is a commonly used pattern in the cloud! Ex: AWS EC2 or Google Compute instance deployments involve having a VM spun up and then deploying your app on top of that
- Virtual machines have a special type of program called the **"Hypervisor"**
    - It runs and manages the lifecycle of these VMs
        - Starts and stops VMs
        - Creates, deletes, and provisions them
    - Common and popular VMs: **VMWare** and **VirtualBox**

> Think of the Hypervisor as the "Traffic Controller." It carves up the physical hardware into smaller slices and hands them to the VMs. It ensures that VM 1 cannot "see" or interfere with the memory of VM 2.

> To make it even simpler, think of the Hypervisor as a "divider" that slices a single physical pie (the Host) into individual slices (the VMs).


Simple diagram:
```
       ________________________________________
      |          VIRTUAL MACHINES (VMs)        |
      |   [ VM 1 ]      [ VM 2 ]      [ VM 3 ] | <--- Each has its 
      |   (Linux)       (Win 11)      (Linux)  |      own full OS
      |_____|___________|___________|__________|
            |           |           |
       _____v___________v___________v_________
      |                                       |
      |        HYPERVISOR (The Manager)       | <--- Slices the hardware
      |_______________________________________|
                        |
       _________________v_____________________
      |                                       |
      |         PHYSICAL HOST HARDWARE        | <--- The actual "Box"
      |      ( CPU  +  RAM  +  STORAGE )      |
      |_______________________________________|
```

Detailed diagram:
```
___________________________________________________________
|                                                           |
|                  VIRTUAL MACHINE (VM)                     |
|   _____________________________________________________   |
|  |                                                     |  |
|  |      [ YOUR APP ]  <-- Runs on the Guest OS         |  |
|  |_____________________________________________________|  |
|  |                                                     |  |
|  |      [ GUEST OS ]  (Windows, Linux, etc.)           |  |
|  |_____________________________________________________|  |
|  |    (Virtual CPU)    (Virtual RAM)    (Virtual I/O)  |  |
|  |_____________________________________________________|  |
|___________________________________________________________|
             |                   |                   |
             v                   v                   v
 ___________________________________________________________
|                                                           |
|                 HYPERVISOR (The Manager)                  |
|        (Examples: VMWare, VirtualBox, KVM, Xen)           |
|___________________________________________________________|
             |                   |                   |
             +-------------------+-------------------+
                                 |
 ___________________________________________________________
|                                                           |
|                PHYSICAL HOST (The Hardware)               |
|   _____________________________________________________   |
|  |            |                 |                      |  |
|  |  [ CPU ]   |    [ MEMORY ]   |   [ I/O / NETWORK ]  |  |
|  |____________|_________________|______________________|  |
|___________________________________________________________|
```

Virtualization is similar to containerization but **NOT** the same!

**Containerization**
- We use the same host computer, but...
- How do we run processes in "isolation"? One process should not conflict with another and the resources must not be shared
- We can use some techniques such as:
    - `chroot` to change the root user for the process
    - `rlimit` to limit the number of OS resources the process can use, and many more tools
- To do this manually to enforce isolation is tedious, tricky, difficult...
- **"Containers"** solve this by providing this isolation out-of-the-box! 
    - Containers allow processes to run on a host OS where they share all the same things provided by the OS
    - The constraint is that they cannot touch anything outside of their "little bounded box" i.e only access resources assigned to them
- **"Docker"** manages the lifecycle of these containers! It is a "container enginer"
    - It edits, runs, manages, stops them and so on!

```
 ___________________________________________________________
|                                                           |
|                  CONTAINERIZED APPS                       |
|   ________________     ________________     ___________   |
|  |                |   |                |   |           |  |
|  |   [ APP A ]    |   |   [ APP B ]    |   | [ APP C ] |  |
|  |  (Binaries/    |   |  (Binaries/    |   | (Binaries/|  |
|  |   Libraries)   |   |   Libraries)   |   |  Libs)    |  |
|  |________________|   |________________|   |___________|  |
|          |                    |                  |        |
|__________|____________________|__________________|________|
           |                    |                  |
 ___________________________________________________________
|                                                           |
|                 CONTAINER ENGINE (e.g. Docker)            |
|      (Handles chroot, rlimits, and process isolation)     |
|___________________________________________________________|
|                                                           |
|             HOST OPERATING SYSTEM (Linux/Windows)         |
|___________________________________________________________|
|                                                           |
|                 PHYSICAL HOST HARDWARE                    |
|___________________________________________________________|
```

### Docker installation

1. Recommended: [MacOS Docker desktop application](https://docs.docker.com/desktop/setup/install/mac-install/) - It also comes with a GUI
2. Alternate but difficult to manages dependencies and updates: [MacOS via homebrew](https://www.reddit.com/r/docker/comments/g1oafc/how_to_install_docker_on_macos_using_homebrew/)

**`docker --version`** command should help verify a successful docker installation

#### Verify Docker is running on your system

Use **`docker run hello-world`** command

```sh
➜  ~ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
58dee6a49ef1: Pull complete
Digest: sha256:f9078146db2e05e794366b1bfe584a14ea6317f44027d10ef7dad65279026885
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (arm64v8)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

➜  ~
```

If you see the above response that means Docker is working fine on your system!

Explanation:

> Unable to find image 'hello-world:latest' locally
> latest: Pulling from library/hello-world

This means that Docker tried to find an "image" locally (What is an image is explained later) so it downloaded it from the central repository [hub.docker.com](https://hub.docker.com/). Think of it like NPM or a Maven repository but for pre-built Docker "images". Ex: You can search for postgres here and download an image for it instead of locally (manually) installing postgres on your system!

***Note***: `'hello-world:latest'` refers to the "image" being "hello-world" and "latest" is the tag (think version) for that image. By default, Docker will fetch the latest tag!

> Status: Downloaded newer image for hello-world:latest

This means that Docker was able to successfully download the "hello-world" image and run it in a container! The output that follows the status message is coming from within the image execution in a container (explained below)

### Dockerfiles, images and containers

In order to understand what an image is and how containers are created from it, we need to understand what a **Dockerfile** is!

Every project will have a **`Dockerfile`** at the root of the project:

```sh
~/my-coffee-app
    Dockerfile
    /src
    /build
```

**What does the Dockerfile contain?**
- It contains instructions on an "image to use" (Base image) (`FROM`)
- Defines instructions such as scripts, packages, tools, etc to download into that image's container (`RUN`)
- Copy commands, say, to copy your project executables from the local (host) into your image's container (`COPY`)
- A default command to run that is likely meant to start our application inside the container (`CMD`)
- ... highly customizable "initialization" instructions ...

**Note 1**: The `WORKDIR` instruction sets the working directory inside the image/container. Any subsequent instructions run from it. It is a recommended best practice to use `WORKDIR` instead of `RUN cd ... && do-something`. Docker creates it automatically during build if it does not exist inside the image. It has nothing to do with the "Host" machine's folder structure. Reasons to use it: 
1. In Docker, each `RUN` command starts in a fresh shell at the default root. A cd inside one `RUN` block will not affect the next `RUN` block
2. Readability: It provides a clear visual cue of where the application files are located and where commands are being executed
3. Container Entry: When a container is started from the image, it will automatically open in the last defined `WORKDIR` (i.e The `WORKDIR` is inherited from the image by the container when it is spun up)

Example:
```Dockerfile
FROM ubuntu:latest
WORKDIR /app
RUN apt-get update && apt-get install -y coffee-maker
COPY ./build
RUN ./build/prepare-app.sh
CMD ["./build/app.sh"]
```

**Note 2**: We can also have a `.dockerignore` file to list the files and folders to ignore while running `COPY` commands (Ex: Avoid copying `node_modules` for a node app). Works similar to `.gitignore` for git

**Note 3**: We can run `docker init` from our project repo and walkthrough an interactive tool that automatically creates a Dockerfile for the project based on our requirements

**We take a Dockerfile and "build" an "image" from it!**
- We can give our image a name and a tag
- We can give any tag we want but "latest" is the default that Docker gives every image

**What is an image?**: An image is like a "filesystem" for the container to run and it is ***immutable***!
- Immutable means we can only build one image (for a given Dockerfile)
- If we want to change the image, we need to ***update*** the Dockerfile and re-build an image from it (as the newer version of that image!)
- The image is built from the Dockerfile using: **`docker build`**
    - The tag (`-t`) flag: `docker build -t <name-tag> <directory>` will create an image using the Dockerfile at the root of the specified directory with a name replaced by `<name-tag>` and the version tag will be *"latest"* since it was not been specified
    - Ex: Executing this from the project root (Containing Dockerfile) of a coffee-maker app: `docker build -t coffee-maker .`
- Listing the images:
    - We use the **`docker images`** command to list the images that have been created so far

**How does an image relate to a container?**:
- A container is created out of an image and run (Image is like a "recipe" for the container) 
- The container will invoke the **`CMD`** defined in the `Dockerfile` to start our application! Points to note:
    - `RUN` commands execute during the build
    - `CMD` commands execute during the container execution. There can be *only one* of this in Dockerfile!
- The container is run from the image using: **`docker run`** (A new container is spun up on the system based on the image)
    - Full command: `docker run <image-name>` to run docker for a given image
    - You can also optionally add the tag (using the format: `<image-name>:<tag>`) but by default it is "latest"

**Note**: Notice how easy it is to define the dependencies to install by mentioning the install steps in the image after a base image has been selected. We can literally build an image for any sort of application with the dependencies to be bundled for it! This flexibility makes Docker so powerful!

**Building a new image: How?**
- Images are immutable. How can we build new images then?
- Scenario: Imagine you have built an image earlier but the application code changed and you need to rebuild it
- Solution: Create an image with a different tag (Ex: `docker build -t <image-name>:<tag> <directory>`)


```
  1. THE PLAN              2. THE PACKAGE             3. THE PROCESS
  (Dockerfile)               (Image)                   (Container)
   __________               __________                _____________
  | FROM ... |             |          |              |             |
  | RUN  ... |  BUILD!     |  BINARY  |    RUN!      |  APP RUNNING|
  | COPY ... | ----------> |  FILES   | -----------> |  (Isolated) |
  | CMD  ... | (Immutable) |  LIBS    | (Read/Write) |_____________|
  |__________|             |__________|
       ^                        ^                          ^
       |                        |                          |
    "The Recipe"           "The Snapshot"            "The Instance"
 (Text instructions)    (Saved on disk)           (Active in Memory)
```

### Real example

Directory structure:
```
~/Desktop/docker-example
    - Dockerfile
    - random-color-message.sh
```

Dockerfile contents:
```Dockerfile
FROM ruby:3.2-slim

WORKDIR /app
RUN gem install lolcat
COPY random-color-message.sh ./
RUN chmod +x random-color-message.sh

CMD ["bash", "./random-color-message.sh"]
```

```sh
➜ docker build -t randomcolor:v1 .
[+] Building 11.4s (10/10) FINISHED                                                                             docker:desktop-linux
 => [internal] load build definition from Dockerfile                                                                            0.0s
 => => transferring dockerfile: 518B                                                                                            0.0s
 => [internal] load metadata for docker.io/library/ruby:3.2-slim                                                                4.4s
 => [internal] load .dockerignore                              
 ...
 ...
 => [1/5] FROM docker.io/library/ruby:3.2-slim@sha256:84184c9e2c368885a1d0c93ad1953c33d81081058d274b87b4aa6f3e209e5d16   
=> [2/5] WORKDIR /app                                                                                                          0.1s
 => [3/5] RUN gem install lolcat                                                                                                2.5s
 => [4/5] COPY random-color-message.sh ./                                                                                       0.0s
 => [5/5] RUN chmod +x random-color-message.sh                                                                                  0.1s
 => exporting to image                                                                                                          0.1s 
 => => exporting layers                       
 ...
 ...
```

```sh
➜ docker images         
REPOSITORY               TAG       IMAGE ID       CREATED              SIZE
randomcolor              v1        2ad53bad574e   About a minute ago   260MB
hello-world              latest    f9078146db2e   3 weeks ago          17.1kB
postgres                 latest    a9abf4275f9e   4 weeks ago          665MB
bitnami/zookeeper        3.8       4d450d4b1dab   16 months ago        927MB
bitnami/kafka            3.6       287cec8bf1b2   16 months ago        1.06GB
provectuslabs/kafka-ui   latest    8f2ff02d64b0   2 years ago          411MB
```

```sh
➜ docker run randomcolor:v1
Random message, random color, random fun.
```

### Layer caching in Docker

Every Dockerfile content is split into "layers"
- You may roughly think of it as instruction being a layer
- When we build subsequent images, Docker checks to see if the layer has changed
- If it has changed, it executes the layer instruction(s) again
- If it has not, the layer is "re-used" (Ex: If dependencies are unchanged, it will not download them again) i.e ***caches*** the layer!
- This makes Docker builds fast!

```
        DOCKER BUILD PROCESS (Layer Caching)
      _______________________________________
     |                                       |
     |  [ Layer 4: CMD ]      <-- Check      | 
     |  [ Layer 3: COPY ]     <-- Check      |
     |  [ Layer 2: RUN ]      <-- Check      |
     |  [ Layer 1: FROM ]     <-- Check      |
     |_______________________________________|
                        |
            IF INSTRUCTIONS ARE THE SAME:
      _______________________________________
     |                                       |
     |       (   USING CACHE   )             |
     |  [---> RE-USE PREVIOUS LAYER <---]    |
     |       ( Lightning Fast! )             |
     |_______________________________________|
                        |
               IF A LINE CHANGES:
      _______________________________________
     |                                       |
     |       (   CACHE BROKEN!   )           |
     |  [!!! RE-RUN INSTRUCTION !!!]         |
     |  [+ ALL FOLLOWING STEPS MUST RE-RUN]  |
     |_______________________________________|
```

How to Stay Fast:
1. **Top-Down Logic**: Docker checks layers from top to bottom. As soon as it finds a change (like a line of code you edited), it *breaks the cache for that line and every line below it*.
2. **Ordering Matters**: You should put things that rarely change (like installing Python or Ubuntu) at the top of your Dockerfile, and things that change often (like your source code) at the bottom.
3. **The Benefit**: Instead of waiting 5 minutes to download packages every time you fix a typo, Docker re-uses the "package layer" and only re-runs the "copy code" layer, taking only 2 seconds.

### Setting environment variables and ports

Use the **`ENV`** command 

Example for setting up a port for our web server application:
```Dockerfile
ENV PORT=9000
# We also need to expose the port to our local machine i.e host if we need to run it on our system
EXPOSE 9000
```

Note: `EXPOSE` helps us "expose" the Docker application port to the host machine so that we can run the application on the same port from the host machine. Else, it will not be reachable from the host machine.

**Running a docker image by attaching a port**
- Defining the port number (as an environment variable or otherwise, hardcoded) is not sufficient
- We need to "expose" the port from the container to the host machine (Shown above using `EXPOSE`)
- We also need to implement **PORT FORWARDING**
    - It is a command that creates a mapping between a host port and a running Docker container port (Else, how would we know that a request to the port on the host needs to go to the container? Exposing it from the container only makes it available for a mapping but it does not automatically forward it)
    - Docker run command provides a **`-p`** command to map the ports of the host and container
        - Ex: `docker run -p <hostport>:<dockerport> <image-tag>` (Host port is always on the "left"!)

```
        YOUR COMPUTER (HOST)             DOCKER CONTAINER
      ______________________          ______________________
     |                      |        |                      |
     |   [ Browser/App ]    |        |    [ Web Server ]    |
     |          |           |        |           |          |
     |   _______v_______    |        |    _______v_______   |
     |  |  PORT 8080    |   |        |   |    PORT 80    |  |
     |  |__(Left Side)__|   |        |   |__(Right Side)_|  |
     |__________|___________|        |___________^__________|
                |                                |
                |----------- TUNNEL -------------|
                     (Mapping/Publishing)
```

Example for a node (express) http server:
```Dockerfile
FROM node:20-slim

WORKDIR /app
COPY package.json ./
RUN npm install --production
COPY server.js ./

ENV PORT=9000
EXPOSE 9000
CMD ["npm", "start"]
```

```sh
➜ docker run -p 9000:9000 samplenodeapp

> random-color-message-server@1.0.0 start
> node server.js

Server is running on port 9000
```

### Managing a container

**`docker ps`**
- Provides a list of running containers

**`docker ps -a`**
- Provides a list of ALL contaners, incl. the ones not running

Every container has an id and a name.

Useful columns:
```
CONTAINER ID
NAMES
IMAGE
STATUS
PORTS
```

Example:
```sh
➜ docker ps -a
CONTAINER ID   IMAGE                           COMMAND                  CREATED        STATUS                      PORTS                                                  NAMES
4dd194ef8be0   samplenodeapp                   "docker-entrypoint.s…"   8 hours ago    Exited (1) 5 minutes ago                                                           inspiring_dijkstra
f6a486815c7a   hello-world                     "/hello"                 10 hours ago   Exited (0) 2 hours ago                                                             beautiful_faraday
7af5ca411568   provectuslabs/kafka-ui:latest   "/bin/sh -c 'java --…"   7 months ago   Exited (255) 5 months ago   0.0.0.0:9100->8080/tcp                                 desktop-kafka-ui-1
cf94c392cb84   bitnami/kafka:3.6               "/opt/bitnami/script…"   7 months ago   Exited (255) 5 months ago   0.0.0.0:9092->9092/tcp                                 desktop-kafka1-1
7cfa6fd9ad64   bitnami/zookeeper:3.8           "/opt/bitnami/script…"   7 months ago   Exited (255) 5 months ago   2888/tcp, 3888/tcp, 0.0.0.0:2181->2181/tcp, 8080/tcp   desktop-zookeeper-1
```

#### Providing a name for a container

While creating a container, we can give it a specific name instead of Docker providing it a default generated name (like `beautiful_faraday`)

Use `--name` with `docker run`.

Syntax: `docker run --name <container-name> <image-tag>`

Example:
```sh
# Terminal 1
➜ docker run --name my-web-app -p 9000:9000 samplenodeapp

> random-color-message-server@1.0.0 start
> node server.js

Server is running on port 9000
```
```sh
# Terminal 2
➜ docker ps
CONTAINER ID   IMAGE           COMMAND                  CREATED              STATUS              PORTS                    NAMES
7712a14b9fc4   samplenodeapp   "docker-entrypoint.s…"   About a minute ago   Up About a minute   0.0.0.0:9000->9000/tcp   my-web-app
```

**`docker rename <old-name> <new-name>`**
- Provides a way to rename an existing container

#### Stop a container

**`docker stop my-web-app`**
- "my-web-app" here is the container name

Example:
```sh
➜ docker stop my-web-app
my-web-app

➜ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

➜ docker ps -a
CONTAINER ID   IMAGE                           COMMAND                  CREATED         STATUS                      PORTS                                                  NAMES
7712a14b9fc4   samplenodeapp                   "docker-entrypoint.s…"   3 minutes ago   Exited (1) 31 seconds ago                                                          my-web-app
...
```

#### View logs for a container

**`docker logs <container-name>`**
- Presents the logs stored until now for that container (p.s the container need not be running - it maintains logs from the previous run as well)

Example:
```sh
➜ docker logs my-web-app

> random-color-message-server@1.0.0 start
> node server.js

Server is running on port 9000
npm error path /app
npm error command failed
npm error signal SIGTERM
npm error command sh -c node server.js
npm error A complete log of this run can be found in: /root/.npm/_logs/2026-04-18T04_23_17_149Z-debug-0.log
```

#### Restarting a container

To start a container (new one), you use: `docker run <image-tag>`

To restart a container, use: **`docker restart <container-name>`**

Example:
```sh
➜ docker restart my-web-app
my-web-app # runs in the background
```

### Using the container ID

First find the ID with:

```bash
docker ps
```

Then use the ID instead of the name:

- Stop:
  ```bash
  docker stop <container-id>
  ```

- Restart:
  ```bash
  docker restart <container-id>
  ```

- View logs:
  ```bash
  docker logs <container-id>
  ```

- Exec into shell:
  ```bash
  docker exec -it <container-id> /bin/sh
  ```

- Inspect:
  ```bash
  docker inspect <container-id>
  ```

> Container ID works exactly like a name in Docker commands, so you can use whichever is easier.

### Removing images and containers

**`rm`**: Use the `docker image rm` or `docker container rm` commands

1. Removing images
    - Use **`docker image rm <image-id-or-name-1> <image-id-or-name-2> ...`**
2. Removing containers
    - Use **`docker container rm <container-id-or-name-1> <container-id-or-name-2> ...`**
    - By default, we get an error with this command if any of the specified containers is "still running"
        - Use **`-f`** flag to force stop any specified containers that are running and remove them
3. **Shortcut**: Remove all images/containers by listing the only the image/container IDs as a sub-command and supplying it to `rm`
    - List all the images using `docker image ls -q`
    - List all the containers using `docker container ls -q` (Use additional `-a` to list stopped containers as well)
    - Use the **`$()`** format to supply the container list to the `rm` command. To clear all images on your system:
        - *Step 1*: Remove all containers (force remove running ones too): **`docker container rm -f $(docker container ls -qa)`**
        - *Step 2*: Remove all images: **`docker image rm $(docker image ls -q)`**
        - Verification: `docker ps -a` should give you an empty list of containers and `docker images` should list no images either 

Example result when all containers and images have been removed (Clean slate):
```sh
➜ docker-example docker ps -a    
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES


➜  docker-example docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
```

### Debugging with Docker

1. **The `docker exec` command**
    - `docker exec` runs a command *inside an already running container*
    - It does *not* start a new container
    - It attaches to a container that is already up and lets you execute commands in its filesystem and process environment

```sh
docker exec [options] <container> <command> [args...]
```

Example to open a shell inside the container (to be able to run commands):
```sh
docker exec -it my-web-app /bin/sh
```

- `-i` keeps STDIN open
- `-t` allocates a pseudo-TTY
- `my-web-app` is the container name or ID
- sh is the command to run inside the container

In simpler words: `-t` gives you a terminal interface, and `-i` keeps input open. Together they make the shell session behave like a normal terminal

Common uses:
- Open a shell:
  ```bash
  docker exec -it my-web-app /bin/sh
  ```
- Inspect files:
  ```bash
  docker exec -it my-web-app ls -la /app
  ```
- Check environment:
  ```bash
  docker exec my-web-app env
  ```
- Run debugging commands:
  ```bash
  docker exec -it my-web-app ps aux
  docker exec my-web-app cat /app/server.js
  ```

Why it is useful:
- Debug running containers without rebuilding or restarting
- Inspect the live container state
- Run one-off commands inside the container
- Check configuration, logs, or app files from within the container’s environment

`docker exec` is the main tool for interacting with a container that is already running.

2. **The `docker scout` command**
- `docker scout` scans Docker images for vulnerabilities and security issues. It analyzes the layers and packages in an image and reports any known CVEs (Common Vulnerabilities and Exposures) found.

```bash
docker scout cves <image-name>
```

Example:

```bash
docker scout cves randomcolor:v1
```

Common uses:
- Scan an image for vulnerabilities:
  ```bash
  docker scout cves randomcolor:v1
  ```

- Get a quick summary:
  ```bash
  docker scout cves --ignore-base randomcolor:v1
  ```
  (ignores vulnerabilities from the base image)

- Recommend fixes:
  ```bash
  docker scout recommendations randomcolor:v1
  ```

Output shows:
- Package name and version
- Vulnerability severity (critical, high, medium, low)
- CVE ID and description
- Fix available (if one exists)

Why it is useful:
- Identify security risks in your container images before deploying
- Know which packages need upgrades
- Follow security best practices
- Prevent running vulnerable code in production

`docker scout` is Docker's built-in security scanning tool to keep your images safe.

Example output: (Note: It requires you to be logged in via `docker login` before executing the scan)
```sh
➜ docker scout cves samplenodeapp
    i New version 1.20.4 available (installed version is 1.17.0) at https://github.com/docker/scout-cli
    ✓ Image stored for indexing
    ✓ Indexed 386 packages
    ✓ Provenance obtained from attestation
    ✗ Detected 18 vulnerable packages with a total of 40 vulnerabilities

## Overview

                    │       Analyzed Image         
────────────────────┼──────────────────────────────
  Target            │  samplenodeapp:latest        
    digest          │  afba42d391eb                
    platform        │ linux/arm64                  
    vulnerabilities │    0C    11H     2M    27L   
    size            │ 74 MB                        
    packages        │ 386                          


## Packages and Vulnerabilities

   0C     6H     0M     0L  tar 6.2.1
pkg:npm/tar@6.2.1

    ✗ HIGH CVE-2026-23950 [Improper Handling of Unicode Encoding]
      https://scout.docker.com/v/CVE-2026-23950
      Affected range : <=7.5.3                                       
      Fixed version  : 7.5.4                                         
      CVSS Score     : 8.8                                           
      CVSS Vector    : CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:H/A:L  

...
...

40 vulnerabilities found in 18 packages
  CRITICAL  0   
  HIGH      11  
  MEDIUM    2   
  LOW       27  
```

### Docker GUI

Everything we have discussed about viewing the images created, the containers running, etc can be done easily from Docker Desktop as well.
- We can even analyse the images for vulnerabilities and fix them
- We can run commands on the running containers

... All this can be done via the CLI too!

### Docker vs Virtual Machines

|Feature       |Virtual Machines (VMs)                    |Docker (Containers)                          |
|--------------|------------------------------------------|---------------------------------------------|
|Isolation     |Total Isolation. Includes a full Guest OS.|Process Isolation. Shares the Host OS kernel.|
|Size          |Heavy. Usually Gigabytes (GB).            |Lightweight. Usually Megabytes (MB).         |
|Startup Time  |Slow. Minutes (needs to boot an OS).      |Fast. Seconds (just starts a process).       |
|Resource Usage|High. Reserves fixed CPU/RAM.             |Low. Uses only what the app needs.           |
|Efficiency    |Lower. Running multiple OSs is taxing.    |Higher. Can run hundreds on one host.        |

```
      [ VIRTUAL MACHINES ]                 [ DOCKER CONTAINERS ]
      ____________________                 _____________________
     |  App 1  |  App 2   |               |  App 1  |  App 2    |
     |---------|----------|               |---------|-----------|
     | Guest OS| Guest OS |               |   Docker Engine     |
     |---------|----------|               |---------------------|
     |     Hypervisor     |               |   Host OS (Kernel)  |
     |--------------------|               |---------------------|
     |   Infrastructure   |               |    Infrastructure   |
     |____________________|               |_____________________|
```

**Common Industry Pattern**: In most modern cloud setups, it isn't "one or the other." Companies often *run Docker containers inside a Virtual Machine*. This gives them the security of a VM with the speed and portability of Docker!

### Docker compose

**Docker Compose** allows you to define and run multiple related containers (like a database, a backend, and a frontend) as a single application using *one simple configuration file*

```
   WITHOUT COMPOSE                     WITH COMPOSE
   (Manual & Messy)                    (Automated & Clean)
  
   [Terminal 1]                      [ docker-compose.yml ]
   > docker run db...                |  services:
                                     |    db: image: postgres
   [Terminal 2]                      |    app: image: my-app
   > docker run app...               |______________________
                                                |
   [Terminal 3]                                 |
   > docker run redis...             > docker-compose up
          |                                     |
    (Hope they link!)                  (Everything starts 
                                         together perfectly)
```

Without "docker-compose", managing containers is difficult because you have to manually manage three separate "lives" that all need to talk to each other to work

```
   THE MANUAL HEADACHE
   _________________________________________________________
  |  1. ORDER:     If the App starts before the DB, it crashes.    |
  |  2. NETWORKING: You must manually link IP addresses/ports.      |
  |  3. CLEANUP:   Stopping one doesn't stop the others.           |
  |  4. REPETITION: You have to type 3+ long commands every time.   |
  |_________________________________________________________|

          FRONTEND          BACKEND             DATABASE
        (Port 3000) ----> (Port 8080) ----> (Port 5432)
             ^                 ^                  ^
             |                 |                  |
      [Manual Start]    [Manual Start]     [Manual Start]
```

The "Super Simple" Reason: Running them manually is like trying to conduct an orchestra by running around and whispering instructions to every musician individually
- Docker Compose is the conductor that gives one signal and makes everyone play together in sync

Docker compose comes built in with Docker Desktop
- Verify the installation with the command **`docker-compose --version`**

Ex: 
```sh
~ docker-compose --version
Docker Compose version v2.34.0-desktop.1
```

#### Docker Compose set up

For each of our applications, we define a `Dockerfile` that contains instructions to build images and create & run containers. This is nothing new related to Docker Compose

Outside these application folders though, we can have a **`docker-compose.yml`** configuration somewhere and it will contain the info on how to map and start these containers together

Example:
```
/my-directory
    /frontend
        Dockerfile
        ...
    /backend
        Dockerfile
        ...
    docker-compose.yml
```

**YAML files are used for configuration in Docker Compose**
1. They use indenting to create hierarchy, inspired by python
2. Use indented `-` prefixed items to define items of a list
3. Use indented `<key>: <value>` pairs to indicates properties of an object
4. Do not use `"` (quotes) unless we really need to be explicit that something is a string number and not confused for a string
    - This makes YAML parsing a bit slow since it needs to figure out on its own usually when a string ends (without the ending `"`)
    - Hence, JSON is more common for data transfer while YAML is generally good for configuration settings
5. The YAML file can have the extensions `.yaml` or `.yml`
6. Optionally, we can have blank lines between sections in the YAML

```yml
version: '3'

services:
  web-app:
    image: my-python-app
    ports:
      - 8080:5000
  database:
    image: postgres:latest
```

Comparison with JSON:
```
    YAML (Clean)                      JSON (Noisy)
    ___________________               ___________________
     app:                            {
       name: CoffeeApp                 "app": {
       version: 1.2                      "name": "CoffeeApp",
       tags:                             "version": 1.2,
         - roasted                       "tags": [
         - caffeinated                     "roasted",
                                           "caffeinated"
                                         ]
                                       }
                                     }
```

**Step 1: Define the docker-compose version**
- Every `docker-compose` is mapped to a docker version. Usually the latest version of each are compatible
- In your YAML: `version: "3.9"` (Using quotes since Docker Compose expects to read the version as a string)

**Step 2: List the applications as "services"**. These are the containers that will get created
- Use the `services` object. The *order* of the services you define *does not matter*!
- Provide a name and a `build` for every service. The build points to the directory containing the Dockerfile for creating the image for that application 
- For a database, we need not "build" one application manually ourselves. We can use an existing `image` (Like done for mongodb below)
```yml
services:
  backend:
    build: ./backend

  frontend:
    build: ./frontend

  mongodb:
    image: mongo:6.0
```

**Step 3: Map the host ports and environment variables for the services**
- Environment variables can be listed as a list or a property of `environment`
    - List syntax: `- KEY=VALUE`
    - Property syntax: `KEY: VALUE`
- To connect the frontend to the backend API:
    - List the environment variable value
    - Use the backend "service name" as the host name and define the port it is using
- Note: Mongodb's default port is `27017`
    - To connect to this DB from the backend, we use the service name ("mongodb" as the host for the connection & use the DB name, "todoapp", in the path)
    - `mongodb://` is simply the protocol used to connect to a mongo DB 
```yml
services:
  backend:
    build: ./backend
    ports:
      - PORT=5000
      - DB_URL=mongodb://mongodb:27017/todoapp

  frontend:
    build: ./frontend
    environment:
      - REACT_APP_API_URL=http://backend:5000/api
    ports:
      - "3000:3000"

  mongodb:
    image: mongo:6.0
    environment:
      - PORT=5000
      - MONGODB_URI=mongodb://root:example@mongodb:27017/todoapp?authSource=admin
    ports:
      - "27017:27017"
```

**Step 4: Define the volume to use for the database persistence**
- If we run docker compose without a "volume" for the DB, the data lives temporarily in the container only
- If the container stops and is remoed, the data is lost 
- We need to map the DB container to list the directory it uses inside the container to store data to a directory outside the container (on the host machine)
- Use the `volumes` option
    - Define it with a name in the DB service: We need to define the directoy in the container to create a volume for on the host machine
    - We also need to list the name of all volumes created anywhere in a separate `volumes` section (in the same nesting level as `services`) and provide a `:` suffix for each (not sure why!)
- MongoDB writes to `/data/db` by default and this is what we need to create a volume outside the container for
```yml
services:
  mongodb:
    image: mongo:6.0
    volumes:
      - mongo-data:/data/db
    ports:
      - "27017:27017"

volumes:
  mongo-data:
```

**FULL EXAMPLE `docker-compose.yml`**

```yml
version: "3.9"
services:
  mongodb:
    image: mongo:6.0
    volumes:
      - mongo-data:/data/db
    ports:
      - "27017:27017"

  backend:
    build: ./backend
    environment:
      - PORT=5000
      - DB_URL=mongodb://mongodb:27017/todoapp
    ports:
      - "5000:5000"

  frontend:
    build: ./frontend
    environment:
      - REACT_APP_API_URL=http://backend:5000/api
    ports:
      - "3000:3000"

volumes:
  mongo-data:
```

#### Running Docker Compose

1. Use **`docker-compose build`** from the directory where the `docker-compose.yml` file is
    - It builds the images for the "services" listed in the configuration
    - It, by default, uses the cache for the images just like how Docker works normally
        - Use the `--no-cache` flag to build afresh without reusing the cache! (Will take longer)

```sh
➜ docker-compose build
# TAKES SOME TIME TO DOWNLOAD DEPENDENCIES AND BUILD
...
...
 => => unpacking to docker.io/library/dsa-explanations-frontend:latest                                                                                                                0.1s
 => [frontend] resolving provenance for metadata file 0.0s
[+] Building 2/2
 ✔ backend   Built   0.0s                                    
 ✔ frontend  Built   0.0s   
...

➜ docker images
REPOSITORY                  TAG       IMAGE ID       CREATED              SIZE
dsa-explanations-frontend   latest    0006a469823a   About a minute ago   218MB
dsa-explanations-backend    latest    80cdfb1da4ad   2 minutes ago        262MB
mongo                       6.0       03cda579c8ca   3 months ago     1.01GB
```

*Note*: When docker compose creates images, it prefixes the image name with the root folder name (`dsa-explanations` in this case)

2. Running the build containers: Use **`docker-compose up`**
    - To run them in the background (detached mode), use the `-d` flag
    - To build them again before running, use the `--build` flag
        - This serves as a shortcut to "build and run the apps" in one go (`docker-compose up --build`)

```sh
➜ docker-compose up
[+] Running 4/4
 ✔ Network dsa-explanations_default       Created                                                                                                     0.0s 
 ✔ Container dsa-explanations-mongodb-1   Created                                                                                                     0.1s 
 ✔ Container dsa-explanations-backend-1   Created                                                                                                     0.1s 
 ✔ Container dsa-explanations-frontend-1  Created
 ...
 ...



# New terminal
➜  docker ps
CONTAINER ID   IMAGE                       COMMAND                  CREATED              STATUS              PORTS                      NAMES
ad4ee44fd204   mongo:6.0                   "docker-entrypoint.s…"   About a minute ago   Up About a minute   0.0.0.0:27017->27017/tcp   dsa-explanations-mongodb-1
b747128a80a3   dsa-explanations-backend    "docker-entrypoint.s…"   About a minute ago   Up About a minute   0.0.0.0:5000->5000/tcp     dsa-explanations-backend-1
1ed63f6089ed   dsa-explanations-frontend   "docker-entrypoint.s…"   About a minute ago   Up About a minute   0.0.0.0:3000->3000/tcp     dsa-explanations-frontend-1
```

What is the suffix `-1`?
- We see the names above as `dsa-explanations-mongodb-1`, `dsa-explanations-backend-1`, & `dsa-explanations-frontend-1`. Why?
- Reason: Docker compose helps with spinning up multiple containers i.e multiple instances of the same container
    - Hence, it numbers them by default
    - Advanced usage that is not required to be learnt (Reasons to use multiple container instances: High Availability and High Scalability)

3. Bring all the docker containers down: Use **`docker-compose down`**
```sh
➜ docker-compose down 
[+] Running 4/4
 ✔ Container dsa-explanations-frontend-1  Removed        0.2s 
 ✔ Container dsa-explanations-mongodb-1   Removed        0.3s 
 ✔ Container dsa-explanations-backend-1   Removed        0.7s 
 ✔ Network dsa-explanations_default       Removed        0.2s 



➜ docker ps 
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES



➜ docker images 
REPOSITORY                  TAG       IMAGE ID       CREATED          SIZE
dsa-explanations-backend    latest    1a9535893505   9 minutes ago    262MB
dsa-explanations-frontend   latest    b0585b0d8e19   16 minutes ago   218MB
mongo                       6.0       03cda579c8ca   3 months ago     1.01GB
```

#### Docker Compose creates a network

Docker creates a network of the containers for our apps.
- Docker compose uses a single network for the containers created and run from within it!

View the network details: Use **`docker network ls`**
```sh
➜ docker network ls 
NETWORK ID     NAME                         DRIVER    SCOPE
c23df1a7b617   bridge                       bridge    local
e25ee41f0617   desktop_default              bridge    local
1f33aa1f93d7   host                         host      local
478a391fb71c   none                         null      local
9f9b35e1e9b3   dsa-explanations_default     bridge    local # The one we composed and is running!
```

We can ping other containers within the composed set of containers! How?
1. DNS lookup table is created by Docker
2. Each container is assigned an IP address
3. Within each container we can directly connect to another container service "by name" set in the `docker-compose.yml`
    - This uses a DNS resolver that exists in each container
    - The resolver reads the mapping to the IP from the Docker DNS
    - It is able to connect to the other running container (Say, via the `ping` command)

Example:
```sh
➜ docker ps                           
CONTAINER ID   IMAGE                       COMMAND                  CREATED         STATUS         PORTS                      NAMES
6b9773187cdb   mongo:6.0                   "docker-entrypoint.s…"   7 minutes ago   Up 6 minutes   0.0.0.0:27017->27017/tcp   dsa-explanations-mongodb-1
97213e370e5b   dsa-explanations-backend    "docker-entrypoint.s…"   7 minutes ago   Up 6 minutes   0.0.0.0:5000->5000/tcp     dsa-explanations-backend-1
3a1e1328e644   dsa-explanations-frontend   "docker-entrypoint.s…"   7 minutes ago   Up 6 minutes   0.0.0.0:3000->3000/tcp     dsa-explanations-frontend-1


➜ docker exec -it 3a1e1328e644 /bin/sh
/app # ping backend
PING backend (172.19.0.2): 56 data bytes
64 bytes from 172.19.0.2: seq=0 ttl=64 time=0.344 ms
64 bytes from 172.19.0.2: seq=1 ttl=64 time=0.164 ms
64 bytes from 172.19.0.2: seq=2 ttl=64 time=0.164 ms
^C
--- backend ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.164/0.224/0.344 ms
/app # 
```

#### Viewing only the containers part of docker-compose

Use **`docker-compose ps`**

Example:
```sh
➜ docker-compose ps   
NAME                          IMAGE                       COMMAND                  SERVICE    CREATED         STATUS         PORTS
dsa-explanations-backend-1    dsa-explanations-backend    "docker-entrypoint.s…"   backend    2 seconds ago   Up 2 seconds   0.0.0.0:5000->5000/tcp
dsa-explanations-frontend-1   dsa-explanations-frontend   "docker-entrypoint.s…"   frontend   2 seconds ago   Up 2 seconds   0.0.0.0:3000->3000/tcp
dsa-explanations-mongodb-1    mongo:6.0                   "docker-entrypoint.s…"   mongodb    2 seconds ago   Up 2 seconds   0.0.0.0:27017->27017/tcp
```

### Docker `-u` option

- In Docker, `-u` is shorthand for `--user`.
- It sets the user identity that the container process runs as

Common usage:
1. `docker run -u 1000:1000 ...`: Run the container process as UID `1000` and GID `1000`
2. `docker run -u node ...` : Run the process as the named user `node` inside the image
3. `docker exec -u 1000 ...`: Run a command inside a running container as a specific user (UID)

Why use it?
- Avoid running as root inside the container
- Match host file permissions when mounting volumes
- Improve security by restricting container process privileges

In short, `-u` controls which user the container process runs as

### Docker `-u` option with a database container

Example DB used: PostgreSQL

Use `docker exec -u`: To run a shell inside a running Postgres container as a specific user:
```bash
docker exec -it -u postgres <container-name> bash
```

(Note: Can even use `/bin/sh` instead of `bash` (or `zsh` if it is installed within the container))

If the image has no `bash`, use `sh`:
```bash
docker exec -it -u postgres <container-name> sh
```

#### If you want to define the user beforehand

1. In `docker-compose.yml`:
- Add the `user` option
```yaml
services:
  db:
    image: postgres:latest
    user: postgres
```

2. In a Dockerfile:
- Add the `USER` instruction
```dockerfile
USER postgres
```

Notes
- The Postgres official image already includes a `postgres` user.
- You can also use a UID/GID pair:
  ```bash
  docker exec -it -u 1000:1000 <container> bash
  ```
- **For running `psql` directly**:
  ```bash
  docker exec -it -u postgres <container> psql -U postgres
  ```

`psql -U postgres` launches the Postgres CLI as the postgres database user

**What is `psql`?**
- `psql` is the **PostgreSQL interactive command-line client**. It's the tool you use to:
    - Connect to a PostgreSQL database
    - Run SQL queries
    - Manage databases and users
    - View database info

**Why both `-u` and `-U`?**
- They operate at **different levels**:
    1. `-u postgres` (lowercase)
        - **What**: Docker flag
        - **Controls**: Which OS user runs the command inside the container
        - **Example**: The `postgres` operating system user
    2. `-U postgres` (uppercase)
        - **What**: psql flag
        - **Controls**: Which database user to connect as
        - **Example**: The `postgres` database user account

**The difference**
```bash
docker exec -u root -U postgres <container> psql
```

This would:
- Run psql as the **root OS user** inside the container (`-u root`)
- But connect to the database as the **postgres database user** (`-U postgres`)

**Why use both?**
- **Best practice security**: Avoid running the container process as root (`-u postgres`)
- **Database authentication**: Connect as the right database user (`-U postgres`)

For PostgreSQL specifically, it's common to have both match (`postgres`), but they serve different purposes and can be different values in other scenarios.

This is the easiest way to exec into the Postgres container as a "non-root" user

### Docker best practices for production

1. **Use official Docker images**
    - Cleaner Dockerfile
    - Official Docker image (Well-supported, security, distribution and dependency aspects managed)
    - Verified and already built with best practices

❌ Bad practice:
```Dockerfile
FROM ubuntu
RUN apt-get update && apt-get install -y node && rm -rf /var/lib/apt/lists/*
```

✅ Good practice:
```Dockerfile
FROM node:20-alpine
```

2. **Use specifics version for the images**
    - "Latest" tag has problems:
        1. You might get different docker image versions
        2. It might break stuff!
        3. Latest tag is unpredictable

❌ Bad practice:
```Dockerfile
FROM node
```

✅ Good practice:
```Dockerfile
FROM node:20-alpine
```

3. **Use smaller images with leaner distros**
    - Problems with large images such as an `ubuntu` image:
        1. Large storage space (Bigger image)
        2. Pulling or transfering the images is going to take longer
        3. Many other utilities are bundled up (A lot more than we need!): The distro will have many known security vulnerabilities w.r.t these utilities. Bad for our image container!
    - Good reasons to use smaller images built on leaner distros:
        1. Small storage space (Smaller image)
        2. Pulling or transfering the images is fast
        3. Fewer vulnerabilties in the utilities that come with the distro

❌ Bad practice: `17.0.1` is a large image (Full-blown Linux OS)
```Dockerfile
FROM node:17.0.1
```

✅ Good practice: `20-alpine` is built on a leaner distro (Alpine linux: Lightweight distro, security oriented, popular)
```Dockerfile
FROM node:20-alpine
```

***Thumb rule: If your application does not require specific utilities, choose a leaner base image***

4. **Optimize caching image layers**
    - Each instruction (RUN, CMD, COPY, etc) inside an image file is a "LAYER"
    - Even the base image will be built on such layers (Can go online and check the layers of common base images on docker hub)
    - Each layer gets cached i.e written to the local filesystem
    - If the layer has *not changed* or **any preceding layers have not changed**, docker will **reuse the cached layer** (instead of downloading data again for that layer)!
        - Faster image building
        - Faster image pulling: If two image versions share some layers, pulling the image becomes faster if the previous version has been downloaded already (Only the newly added layers will be downloaded for the newer version)

**Once a layer changes, all the following layers are re-created as well!** (Cache gets invalidated for those layers)

**Note: Changes to the files you are copying invalidates the cache for the `COPY` command**

❌ Bad practice:
```Dockerfile
FROM node:20-alpine
WORKDIR /app
# If the /app files change then the cache is invalidated
COPY myapp /app 
# The following time-consuming process will run again instead of using the cache 
RUN npm install --production
CMD ["node", "src/index.js"]
```

✅ Good practice:
```Dockerfile
FROM node:20-alpine
WORKDIR /app
# Copy only the minimal files required for installing all packages
COPY package.json package-lock.json .
# The following time-consuming process will only run once
RUN npm install --production
# Now if cache gets invalidated by the next instruction if our app code changes, `npm install` cache is still valid!
COPY myapp /app 
CMD ["node", "src/index.js"]
```

(Note: `docker history <image>` will provide info on all the layers used to build the image!)

***Thumb rule: Order commands in your Dockerfile from the least to the most frequently changing ones to take advantage of caching***

5. **Use `.dockerignore` file to exclude files and folders**
    - You do not require these files and folders to be copied to the image as they are supposed to be "generated"
    - Similar to `.gitignore`, you list the files and folders to ignore. The file has to be in the project root directory
    - This ***reduces*** the image size and improves performance!

✅ Sample `.dockerignore`:
```
node_modules
npm-debug.log
.DS_Store
```

6. **Use multiple stages in the Dockerfile to reduce the image size**
    - We do not need development or build time dependencies once the image has been built
    - That is, the image need not contain:
        - Test frameworks (Jest, JUnit, ...)
        - Build tools (Maven, Gradle)
        - Temporary files, etc
    - Having these also increase the attack vector (Hence, a security concern)
    - Use two stages using **"AS"** syntax, one for the build, and one for the final image
        - Only copy what you need from the build stage into the final stage using the `--from` flag
    - Only the final image layers are used for caching!

```Dockerfile
# Build stage
FROM maven AS build
WORKDIR /app
COPY myapp /app
RUN mvn package

# Run stage
FROM tomcat
COPY --from=build /app/target/file.war /usr/local/tomcat/webserver
...
```

7. **Use the least privielged user to run containers**
    - Containers are run as a particular user
    - By default, it is the host machine's "root" user
        - This is a security issue!
        - A hacker might get control of the host machine's processes and data through the container since the root user accesses both
    - Also, we seldom need the root user privileges to run our container
    - Best practice: Create a **dedicated user and group** with the right permissions and use that account to run the container!
        - This can be done within the Dockerfile itself using the `USER` directive

✅ Good practice:
```Dockerfile
# Create group and user
RUN groupadd -r tom && useradd -g tom tom

# Set ownership and permissions
RUN chown -R tom:tom /app

# Switch to user
USER tom

CMD node index.js
```

***Note: Some base images (like "node") have a **generic user "bundled in"** so you do not have to use a new user yourself! In such a case do not use a `USER` directive***

8. **Scan your image for security vulnerabilities**
    - It is a good practice to perform this scan after the image has been built
    - Use `docker scout cves <image>` (Earlier: `docker scan <image>`)
    - Prerequisite: You need to be logged into Docker using the `docker login` command
    - Internally, Docker uses **Snyk** tool to do the vulnerability scanning
    - You can also use Docker Desktop to perform the scanning from the GUI easily

### How Docker works

Docker is written in **Go**

Docker uses the concept of **namespaces** to create containers.

Each container is a normal process on a host machine (unlike a VM)

Internally, Docker is essentially a user-friendly wrapper around powerful features already built into the **Linux Kernel**. It doesn't use "magic" to isolate containers; it uses **specific kernel primitives** to trick a process into thinking it is running on its own private computer

#### 1. The Core Engine: The "Isolation Secret"

Docker leverages *three* main technologies to create the container environment:

* **Namespaces (Isolation):** The "fences" that hide resources.
* **Control Groups / cgroups (Limitation):** The "meter" that restricts resources.
* **Union File Systems (Efficiency):** The "stack" of layers that creates the image.

#### 2. Namespaces: The Virtual Eyes
Namespaces provide the **isolation layer**. They ensure that a process inside a container cannot see or interfere with processes, files, or networks outside of its "bubble."

| Namespace | What it hides/isolates |
| :--- | :--- |
| **PID** | Process IDs. Container Process #1 thinks it is the system's "init" process. |
| **NET** | Network stacks. Each container gets its own virtual IP and port range. |
| **MNT** | Mount points. The container sees its own file system, not the host's. |
| **UTS** | Hostname and Domain name. |
| **IPC** | Shared memory and communication between processes. |
| **USER** | User and Group IDs. A "root" user inside a container isn't "root" on the host. |

### 3. Control Groups (cgroups): The Resource Manager

While namespaces hide things, **cgroups** (Control Groups) make sure a container doesn't act like a "noisy neighbor" and hog all the host's power.

If you have a host with 16GB of RAM, and one container starts leaking memory, it could crash the whole machine. **cgroups** prevent this by setting hard limits:
* **CPU:** Limit a container to only use 0.5 cores.
* **Memory:** Limit a container to exactly 512MB.
* **Disk I/O:** Limit how fast a container can read/write to the disk.

#### 4. Union File Systems (UnionFS): The Layered Cake

This is how Docker handles **Images**. Instead of copying a 500MB OS for every container, Docker uses a "copy-on-write" system

* **Image Layers:** These are ***read-only*** (immutable). Multiple containers can share the same base layers (like Ubuntu).
* **Container Layer:** When you start a container, Docker adds a thin "Writable Layer" on top. Any changes you make (saving a file) only happen in this tiny top layer.

```text
       [ Container 1 Layer (Write) ]    [ Container 2 Layer (Write) ]
                    |                                |
       _____________v________________________________v_____________
      |                                                            |
      |                 SHARED READ-ONLY IMAGE LAYERS              |
      |      (App Code) + (Dependencies) + (Base OS / Ubuntu)      |
      |____________________________________________________________|
```

#### 5. Putting it all together: The Internal Flow

1.  **Request:** You run `docker run`.
2.  **Creation:** The Docker Daemon (containerd) talks to the Linux Kernel.
3.  **Isolation:** The Kernel creates a set of **Namespaces** for that process.
4.  **Limitation:** The Kernel applies **cgroups** to that process.
5.  **Storage:** The **UnionFS** mounts the image layers and adds a writable layer.
6.  **Execution:** Your application starts, believing it is the only thing running on a fresh Linux machine.

#### How do Docker containers work on Windows or MacOS?

Since Namespaces and cgroups are features exclusive to the Linux Kernel, Windows and macOS have to do a little bit of "cheating" to make Docker work. They can't run Linux containers natively because their own kernels (Windows NT or macOS Darwin) don't understand those Linux-specific isolation tricks

**The Solution: The Lightweight VM**

When you install Docker Desktop on Windows or Mac, it automatically sets up a tiny, invisible Linux Virtual Machine in the background.

- On Mac: It uses a lightweight hypervisor called **Apple Hypervisor Framework**
- On Windows: It uses **WSL 2** (Windows Subsystem for Linux) or **Hyper-V**

```
  MACOS / WINDOWS HOST
  _______________________________________
 |                                       |
 |   [ Docker Desktop App / CLI ]        |
 |_______________________________________|
 |                                       |
 |      LIGHTWEIGHT LINUX VM             | <--- The "Hidden" Host
 |   _________________________________   |
 |  |                                 |  |
 |  |   [ Linux Kernel ]              |  |
 |  |   (Namespaces + cgroups)        |  |
 |  |_________________________________|  |
 |  |                                 |  |
 |  |  [ Container ]  [ Container ]   |  |
 |  |_________________________________|  |
 |_______________________________________|
 |                                       |
 |   HYPERVISOR (WSL2 / Apple HyperV)    |
 |_______________________________________|
```

## Kubernetes

Kubernetes is an **Orchestration Tool** allowing you to run and manage your container-based workloads

- It is essentially a container management tool, be it Docker containers or any other similar tool
    1. It helps you manage applications made up by 100s or 1000s of containers
    2. It also helps you manage them in *different environments*
- It was developed by **Google**

**Problem that K8S solves**
- Rise of microservices from monoliths caused an increase in usage of container technologies
- Managing those containers across multiple environments using self-made tools and scripts was too complex or even impossible
- This led to a need for container orchestration technologies

**What do orchestration tools guarantee?**
1. High availability (Users can almost always use the application)
2. High scalability (There is a certain level of performance always guaranteed)
3. Disaster recovery (Backup and restore when systems crashes, physical infrastructure destruction, etc)

### K8S basic architecture

K8S has:
1. A **master** node
2. One or more **worker** nodes
    - Each worker node has a k8s process called as a **Kubelet**
    - Each kubelet process is what enables to communicates with each other within a k8s cluster
    - Each worker node has containers (Docker containers, for example) of different applications deployed on it!
    - Obviously then, the worker nodes is where the actual work is happening i.e the applications are running

***Note***: Depending on how the workload is distributed, you have different number of containers on different worker nodes

**What does the master node run?**
- It runs many processes to manage and run the cluster properly (It does not run any applications)
- The processes running the master node are:
    1. 
