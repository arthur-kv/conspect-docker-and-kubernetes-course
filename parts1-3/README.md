# Docker and Kubernetes: The Complete Guide
https://www.udemy.com/docker-and-kubernetes-the-complete-guide/ 

## Parts 1 - 3

```
> docker run hello-world
```

**Container** is a running process along with a subset of physical resources on your computer that are allocated to that process specifically.

**Image** is kind of a snaphot of the file system along with a very specific start up command as well.

OS features that stand behind Docker:
* Namespacing - isolating resources per process.
* Control Groups (cgroups) - limit amount of resources used per process.

The above features are Linux specific only. So, a Linux virtual machine is installed with Docker. It is used to host all the containers.
```
> docker version
(OS/Arch: linux/amd64)
```

### Part 2 

`docker run` - Create and run a container from an image
```
> docker run <image name> 
```

```
> docker run <image name> <default command override>
> docker run busybox echo hi there
```

Prints folders that solely exist inside the container:
```
> docker run busybox ls
```

Both `echo` and `ls` exist inside busybox's filesystem.

#### 15. Listing Running Containers

List all runnning containers: 
```
> docker ps
> docker run busybox ping google.com - this container doesn't exit right after the launch
In a separate terminal:
> docker ps - should see the above container
```

`Ctrl + C` to stop a container

All the containers that have ever been created:
```
> docker ps --all
```

#### 16. Container lifecycle

**docker run = docker create + docker start;**

Create a Container out of an image:
```
> docker create <image name> <command>
```

Start a Container:
```
> docker start <container id>
```

**Creating:** take the filesystem and prepare it to use in a new container. 

**Start:** execute the start up command.

```
> docker create hello-world
> 1ec14f5a4ffc29158bb765cb1fb6a3024825f2cba7a715ca4222f59f4306367d
> docker start -a 1ec14f5a4ffc29158bb765cb1fb6a3024825f2cba7a715ca4222f59f4306367d
```

`-a` - *"Attach STDOUT/STDERR and forward signals"*: watch for output from the container and print it out to your terminal.

#### 17. Restarting Stopped Container
```
> docker ps --all 
Can take <container id> from there and put it to 
> docker start <container id>
```

We cannot replace a container's command when restarting a stopped container

#### 18. Removing Stopped Containers
```
> docker system prune - deletes also build cache (any image that was fetched from docker.hub)
```
After that we have to re-download images from docker.hub.

#### 19. Retrieving Log Outputs
```
> docker logs <container id>
```

#### 20. Stopping Containers 
`SIGTERM` is sent to the process inside a container (*"Shut down on its own time"*):
gives *10s* for the process to stop, then sends `SIGKILL`.
```
> docker stop <container id>
```

`SIGKILL` - *"Shut down right now"*:
```
> docker kill <container id>
```

#### 21. Multi-Command Containers
#### 22. Executing Commands in Running Containers
```
> docker exec -it <container id> <command> 
```
`-it` - type input directly into the container

```
> docker run redis
> docker exec -it <container id> redis-cli
```

#### 23. The purpose of the `-it` flag
Every process started in the Linux environment has 3 communication channels attached to it: 
* `STDIN` - communicate info into the process. We type something -> the stuff we type is being directed into `STDIN`
* `STDOUT` - conveys info from the process
* `STDERR` - conveys info from the process (for errors), similar to `STDOUT`

`-i` - attach our terminal to the STDIN of the running process.

`-t` - show text in terminal in a nicely formatted manner. (And it does a little bit more).

#### 24. Getting a Command Prompt in a Container
```
> docker exec -it <container id> sh
```
Press `Ctrl + D (or type "exit")` - to exit if `Ctrl + C` doesn't work 

*"bash", "powershell", "zsh", "sh"* - are command processors.

*"sh"* - allows to type commands in and have them be executed.

#### 25. Starting with a Shell 
```
> docker run -it <image name>
```
```
> docker run -it busybox sh
chances are that it will replace the primary command of the container
```

#### 26. Container Isolation

Containers do not share their file system automatically.

```
> docker run -it busybox sh
# ls 
# touch hithere

In a separate terminal window:
> docker run -it busybox sh
# ls - not gonna see "hithere"

> docker ps
```


### Part 3 Building Custom images Through Docker Server

#### 27. Creating Docker Images

**Dockerfile -> Docker Client -> Docker Server -> Usable Image!**

**Dockerfile:** a plain text file with configs. How our container behaves. What different programs it's going to contain and what it does when it starts.

**Docker Server:** heavy lifting. Builds a usable image from a Dockerfile.

**Creating a Dockerfile flow:**
1. Specify a base image
2. Run some commands to install additional programs 
3. Specify a command to run on container startup

#### 28. Building a Dockerfile

Task: create an image that runs `redis-server`.

```bash
> mkdir redis-image
> cd redis-image 
```
```bash
> touch Dockerfile
```
The contents of the Dockerfile:
```Dockerfile
# Use an existing docker image as a base
FROM alpine
# Download and install a dependency
RUN apk add --update redis
# Tell the image what to do when it starts as a container
CMD ["redis-server"]
```
```
> docker build .
> docker run f8d9b040b5ae
```

#### 29. Dockerfile Teardown

* `FROM` - a base image 
* `RUN` - execute a command while preparing an image
* `CMD` - what should be executed when our image is used to start a container

#### 30. What's a Base Image?

> **Analogy:** Writing a Dockerfile = Being given a computer with no OS and being told to install Chrome.

`apk` - apache package. A package manager that is preinstalled in alpine images. 

#### 31. The Build Process in Detail
```
> docker build . 
Giving our file off to docker CLI. Generate an image out of a Dockerfile. 
```

`FROM` - first it looks for cache. If no cache exists it goes to docker.hub and downloads the corresponding image

For `RUN` and `CMD` we receive this output during the build proccess:

> ...

> Running in <id> 

> ...

> Removing intermediate container <id>

> ...

**Build process:**
* Looks back to the previous step
* Creates a new temporary container in memory out of the image from the prev. step
* Executes a command as a primary command of the container.
* Stops the container
* Takes its filesystem snapshot
* Saves it as a temporary image with <id>
* Throws away the temporary container.

`СMD` - tells the container: *"If you run for real execute this command"*. So the `CMD` command is not executed during the build proccess.

#### 33. Rebuilds with Cache

As we get a temporary image from each command in Dockerfile, Docker uses cache if nothing has changed.
As soon as something changes in Dockerfile, Docker will re-run commands starting from the first one changed till the end.

#### 34. Tagging an image
After using `docker build` we can use `docker run <id>`. 

```
> docker build -t <name of an image> .
```

A name can follow this convention: 

`<your docker ID>/<Repo/Project name>:<version, e.g. latest>`

```
> docker run <name>
```
We can skip version if the latest version is intended to be used.

#### 35. Manual Image Generation with Docker Commit
**Image -> Container -> Image (generate a useful image from a container)**

Manually make the same as Dockerfile does.

```
> docker run -it alpine sh
/# apk add --update redis

Then in a new terminal window:

> docker ps 
> docker commit -c 'CMD ["redis-server"]' <id of a running container>
<id of a new image>

> docker run <id of the image>
```

