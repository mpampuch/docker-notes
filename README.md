# docker-notes

Some tips / things of notes for myself while I'm learning Docker

## What is Docker?

Docker is a platform and tool designed to make it easier to create, deploy, and run applications using containers. Containers allow developers to package an application with all of its dependencies — such as libraries, frameworks, and other components — into a standardized unit for development, shipment, and deployment. 

![What is docker](what-is-docker.webp)

It promotes consistency, portability, and scalability across different environments, from development laptops to production servers and cloud platforms. As a result, Docker has become an essential tool in modern software development workflows, enabling faster development cycles, improved collaboration, and more efficient resource utilization.

![Docker example](docker-example.jpg)

## Docker Containers vs Virtual Machines

![Docker containers versus virtual machines](docker-vs-vm.jpg)

Virtual Machines:
- Each VM needs a full-blown OS
- Slow to start 
- Resource intensive

Containers:
- Allow running multiple apps in isolation
- Are lightweight
- Use OS of the host
- Start quickly
- Need less hardware resources

## How Docker Works

![How Docker Works](how-docker-works.png)

1. The client sends a REST API request to the server (aka the Docker Engine)
2. Docker can then create a container from a docker image
    - A container is just a process (a special kind of process with its own filesystem)

### How Docker Runs 

Docker works slightly differently on Windows, Linux, and macOS due to the underlying architecture of each operating system. Let's break it down:

- Linux: Docker was initially designed to run natively on Linux. On Linux systems, Docker uses the host operating system's kernel and namespaces to create isolated containers. This means that Docker containers on Linux share the same kernel as the host system but have their own isolated filesystem, processes, and network interfaces. Docker Engine, which is the underlying runtime for Docker, communicates directly with the Linux kernel through system calls.

- Windows: Docker on Windows uses a lightweight virtual machine (VM) to run Linux containers. This is because Windows and Linux have different kernel architectures, and Docker relies on Linux kernel features for containerization. To bridge this gap, Docker Desktop for Windows includes a small Linux VM called a "MobyLinuxVM" (using Hyper-V or WSL 2 backend) that runs a minimal Linux distribution. This VM hosts the Docker Engine and manages Linux containers. However, Docker also supports Windows containers, which run natively on Windows and do not require the Linux VM. Windows containers use Windows-specific kernel features for isolation.

- macOS: Similar to Windows, macOS doesn't natively support Linux containers due to differences in kernel architecture. Docker on macOS also uses a lightweight Linux VM (HyperKit) to run Linux containers. Docker Desktop for Mac includes a component called "LinuxKit" that creates a small Linux environment running on the macOS host. This Linux environment hosts the Docker Engine and manages Linux containers. As with Windows, Docker also supports running native Windows containers on macOS using a Windows VM, which is included in Docker Desktop.

![Docker on different operating systems](docker-operating-systems.png)

## Docker Images

A Docker image serves as the blueprint for creating Docker containers. It is a lightweight, standalone, and executable package that contains everything needed to run a piece of software. This includes:
- A cut-down OS
- A runtime environemnt (e.g. Node)
- Application files
- Third-party libraries
- Configuration files
- Environmental variables

### Docker Image Commands

```bash
docker build -t <name> .
# Builds a Docker image from a Dockerfile in the current directory
# and tags it with the specified name (<name>).

docker images
# Lists all Docker images stored locally on the system.
# This is a deprecated command, and it's recommended to use
# `docker image ls` instead.

docker image ls
# Lists all Docker images stored locally on the system.
# This is the preferred command for listing Docker images,
# replacing the deprecated `docker images`.
```

## Running a Docker Container

```bash
docker start <containerID>
docker stop <containerID> 
```

## Entering a Docker Container Interactively


You can enter Docker containers interactively to inspect the virtual environment. Here is how it is done:

1. First, start a container from the Docker image you want to enter. You can do this using the docker run command. For example:

```bash
docker run -it <image_name_or_id> /bin/bash
```

Replace `<image_name_or_id>` with the name or ID of the Docker image you want to run. The `-it` flag is used to start the container in interactive mode and allocate a pseudo-TTY for the session. `/bin/bash` specifies the command to run inside the container, which in this case is the Bash shell. You can replace `/bin/bash` with the command appropriate for the shell available in the image (e.g., `/bin/sh`).

2. If the container is already running, you can use the `docker exec` command to enter it interactively. For example:

```bash
docker exec -it <container_name_or_id> /bin/bash
```

Replace `<container_name_or_id>` with the name or ID of the running container you want to enter. Again, `/bin/bash` specifies the command to run inside the container.

After running either of these commands, you should be dropped into an interactive shell session within the Docker container, allowing you to execute commands and interact with the container's filesystem and environment. 

### The Interactive Docker Container Prompt

The interactive Docker container prompt typically appears as something like `root@container_id:/app#` when you enter an interactive session within a Docker container. Here's a breakdown of its components:

#### User and Hostname:

- `root`: This indicates the username of the current user inside the container. In this case, it's "root," which is the superuser or administrator account in Unix-like operating systems.
- `@`: This symbol separates the username from the hostname.
- `container_id`: This is the unique identifier of the Docker container. It could also be the container's name if you provided one when creating the container.

#### Current Directory:

- `/app`: This represents the current working directory inside the container. In this example, the prompt indicates that you are currently in the /app directory.
Prompt Symbol:

- `#`: This symbol indicates that the user is operating with root privileges. In Unix-like systems, the dollar sign `$` typically indicates a _regular user_, while the pound sign `#` indicates the _root user_ or _superuser_.

So, when you see a prompt like `root@container_id:/app#`, it means that you are logged in as the root user inside a Docker container, currently working in the /app directory, and have superuser privileges, allowing you to execute commands with elevated permissions.

> [!WARNING]
> It is typically dangerous to give users root privileges inside a container (despite this being the default Docker behaviour). If you are building an app, it is advised fix the user and group permissions in the Dockerfile.

### Exiting Interactive Docker Containers

When you're finished, you can exit the interactive session by typing `exit` and pressing Enter.

## Dockerfiles

```docker
FROM         # to specify the base image 
WORKDIR      # to set the working directory
COPY         # to copy files/directories
ADD          # to copy files/directories
RUN          # to run commands 
ENV          # to set environment variables
EXPOSE       # to document the port the container is listening on
USER         # to set the user running the app
CMD          # to set the default command/program
ENTRYPOINT   # to set the default command/program
```

### `FROM`

The `FROM` instruction in Dockerfile specifies the base image from which you want to build your Docker image. It is typically the first instruction in a Dockerfile and defines the starting point for the build process.

> [!CAUTION]
> If not specified, by default `FROM` pulls the latest image. 
> 
> For example:
> 
> `FROM node`
>
> is equivalent to
> 
> `FROM node:latest`
>
> **You should never do this!** Upon successsive builds of your docker image the environment might change. 
> 
> Instead, use a specific version for your base image. 
> 
> Example:
> 
> `FROM node:14.16.0-alpine3.13`

### `WORKDIR`

The `WORKDIR` instruction in a Dockerfile sets the working directory for any subsequent `RUN`, `CMD`, `ENTRYPOINT`, `COPY`, and `ADD` instructions. It allows you to specify the directory where commands will be executed within the Docker container.

- If the directory specified in `WORKDIR` does not exist, Docker will create it for you.
- If multiple `WORKDIR` instructions are specified in a Dockerfile, only the last one will take effect.
- Paths specified in `WORKDIR` are relative to the previous `WORKDIR` or to the Dockerfile's directory if no previous `WORKDIR` is specified.
- You can use both absolute and relative paths in the `WORKDIR` instruction.

### `COPY`

The `COPY` instruction in a Dockerfile copies files or directories from the Docker host's filesystem into the filesystem of the Docker image being built. It allows you to add files from your local machine or from a directory on the Docker host into the Docker image during the build process.

- If the source is a file, the destination must be a directory. If the source is a directory, the destination can be a directory or a file.
- You can specify multiple source files or directories, and they will all be copied into the destination directory.
    - When using `COPY` with more than one source file, the destination must be a directory and end with a `/` (or `\`). Example: `/app` must be `/app/`.

> [!NOTE]
> 
> A common pattern is 
> 
> ```docker
> WORDIR /app
> COPY . .
> ```
>
> This copies all files and directories from the Docker host's current directory (the directory containing the Dockerfile) into the `/app` directory within the Docker container. The first `.` represents the source directory on the Docker host, and the second `.` represents the destination directory within the Docker container.


#### Relative Paths:

- Paths specified in the `COPY` instruction are relative to the build context, which is typically the directory containing the Dockerfile.
- The build context is sent to the Docker daemon when you run the docker build command, and the paths specified in the `COPY` instruction are resolved relative to this build context.

#### Permissions:

- When files are copied into the Docker image, they are copied with the same permissions and ownership as on the Docker host. This means that the user and group ownership of the files will be preserved.
- You can use the `--chown` option with `COPY` to change the ownership of the copied files in the Docker image.

### `ADD`

The `ADD` instruction in a Dockerfile copies files, directories, or remote URLs from the Docker host's filesystem or from a URL into the filesystem of the Docker image being built. It's similar to the `COPY` instruction but with additional capabilities.

#### Automatic Extraction:

- If the source is a compressed archive (such as a `.tar`, `.tar.gz`, `.tgz`, `.tar.bz2`, `.tbz2`, .`tar.xz`, `.txz`, `.zip`, `.gz`, `.bz2`, `.xz`, or `.Z` file), `ADD` will automatically extract it into the destination directory in the Docker image.
- This automatic extraction feature makes `ADD` convenient for adding compressed files or archives into a Docker image and automatically extracting them during the build process.

#### Remote URLs:

- If the source is a remote URL, `ADD` will download the content from the URL and copy it into the Docker image.
- This feature allows you to directly add files from the internet into your Docker image during the build process.

#### Permissions:

- Similar to the `COPY` instruction, when files are copied or downloaded into the Docker image using `ADD`, they are copied with the same permissions and ownership as on the Docker host.
- You can use the `--chown` option with `ADD` to change the ownership of the copied files in the Docker image.

### `RUN`

The `RUN` instruction in a Dockerfile executes commands within the Docker image during the build process. You can use it to execute any command that you would normally run in a terminal within the Docker image. It's a fundamental command that allows you to install packages, update software, create directories, or perform any other actions necessary to configure the environment inside the Docker container.

#### Layer creation

Each `RUN` command creates a new layer in the Docker image. This means that changes made by one `RUN` command are committed to the image and become available to subsequent `RUN` commands. This layering mechanism ensures that Docker can efficiently cache and reuse intermediate images during the build process.

Since each `RUN` command creates a new layer, it's essential to clean up any temporary files or dependencies that are no longer needed after each command to avoid bloating the image size unnecessarily. This can be done using additional `RUN` commands to remove files or using the `&&` operator to chain cleanup commands with the main command.

> [!NOTE]
> It's good practice to combine multiple commands into a single `RUN` instruction when possible, using shell chaining (`&&`) to execute them sequentially. This helps minimize the number of layers created in the image, reducing its size.

`RUN` supports both shell form and exec form. 

#### Shell form 

In shell form, commands are executed using the default shell specified in the Dockerfile (`/bin/sh -c` on Linux-based images). 

Example:    

```docker
RUN apt-get update && apt-get install -y \
    package1 \
    package2
```

#### Exec form 

In exec form, commands are executed directly without a shell.

Example:    

```docker
RUN ["apt-get", "update", "&&", "apt-get", "install", "-y", "package1", "package2"]
```

> [!NOTE]
> Exec form is preferred for clarity and avoids issues with shell parsing.

### `ENV`


### `EXPOSE`


### `USER`


### `CMD`


### `ENTRYPOINT`





## Choosing a Suitable Base Image

If you specify 

## Excluding Files and Directories with `.dockerignore`

A `.dockerignore` file is used to specify files and directories that should be excluded when building a Docker image. It works similarly to `.gitignore` files in Git, allowing you to define patterns to exclude specific files or directories from being included in the Docker image during the build process.

Example, inside a `.dockerignore` file you can have 

```
node_modules/
```

## Optimizing Builds

![Dockerfile structure](structuring-docker-file.png)

## Removing Docker Images

```docker
docker container rm <containerID> 
docker rm <containerID> 
docker rm -f <containerID>        # to force the removal
docker container prune            # to remove stopped containers
```

## Tagging Docker Images

## Sharing Docker Images

## Saving and Loading Images

## Important Linux Information for Docker

### Linux Distributions

Linux distributions, often abbreviated as "distros," are variations of the Linux operating system bundled with different combinations of software packages and configurations. These distributions are built around the Linux kernel and typically include a package management system for installing, updating, and removing software.

Here are some key points about Linux distributions:

- Kernel: The Linux kernel is at the core of all Linux distributions. It provides the essential functionality for interacting with hardware, managing processes, and handling system resources. While the kernel remains consistent across distributions, each distribution may include different kernel versions or patches tailored to specific needs.

- Package Management: Linux distributions typically come with a package management system that simplifies software installation and management. Examples include APT (Advanced Package Tool) used by Debian and Ubuntu, YUM (Yellowdog Updater, Modified) used by Fedora and CentOS, and Pacman used by Arch Linux. These tools allow users to install, update, and remove software packages, as well as resolve dependencies automatically.

- Desktop Environment: Linux distributions often offer a choice of desktop environments, such as GNOME, KDE Plasma, Xfce, LXDE, and Cinnamon. These desktop environments provide graphical user interfaces (GUIs) for interacting with the operating system, including window management, file browsing, and application launching. Users can choose the desktop environment that best suits their preferences and hardware requirements.

- Default Software: Linux distributions come with a set of default software packages, including text editors, web browsers, email clients, media players, and system utilities. The specific selection of software may vary depending on the distribution's target audience and use case. For example, distributions designed for servers may include server-oriented software, while desktop distributions may focus on productivity and multimedia applications.

Popular Linux distributions include Ubuntu, Debian, Fedora, and CentOS

### The Apline Linux distribution

Alpine Linux is a lightweight and security-focused Linux distribution known for its minimalist design, small footprint, and focus on simplicity. Alpine Linux is designed to be lightweight, with a small installation size and minimal resource requirements. It achieves this by using a musl libc (C library) instead of the more common GNU libc (glibc) used in many other Linux distributions. This allows Alpine to have smaller binaries and libraries, resulting in reduced memory usage and faster startup times.

Alpine Linux is widely used as a base image for Docker containers due to its lightweight nature, security features, and container-friendly design. 

Here are things to note about using Alpine Linux as a base image:

1. Alpine Linux does not have the `apt` package manager. Alpine Linux uses the `apk` package manager (Alpine Package Keeper) to manage software packages. `apk` is lightweight, fast, and designed for Alpine's musl libc-based system. It supports features such as package installation, upgrading, removal, and dependency resolution. The Alpine package repositories provide a wide range of pre-built packages, including common software components and libraries.
2. Apline Linux does not use `bash` as the default shell. Instead, you'll likely be interacting with it using `sh`. To use `bash` as the default shell, you can install it with `apk add bash`.

### Managing Linux Packages

- If available, `apt` should be used over `apt-get`
- `apt update` must be called before trying to install a package. It is essential for ensuring that `apt` has the latest information about available packages and their dependencies. 

```bash
apt update 
apt list 
apt install nano
apt remove nano
```

### Managing Environment Variables

```bash
printenv           # to list all variables and their value
printenv PATH      # to view the value of PATH
echo $PATH         # to view the value of PATH
export name=bob    # to set a variable in the current session
```

### Managing Users and Groups

```bash
useradd -m john    # to create a user with a home directory
adduser john       # to add a user interactively
usermod            # to modify a user
userdel            # to delete a user

groupadd devs      # to create a group 
groups john        # to view the groups for john
groupmod           # to modify a group
groupdel           # to delete a group
```

In Linux, each user account can belong to one primary group and zero or more secondary groups/ Here's a breakdown of primary and secondary groups:

#### Primary Group:

- Every user account in Linux is associated with a primary group.
- When a user creates a file or directory, the primary group of the user is typically assigned as the group owner of that file or directory by default.
- The primary group is specified in the user's entry in the `/etc/passwd` file.
- The primary group is identified by its Group ID (GID), which is a numeric value associated with the group.

#### Secondary Groups:

- In addition to the primary group, a user can be a member of multiple secondary groups.
- Secondary groups allow users to access files and resources that belong to those groups.
- Secondary groups are specified in the user's entry in the `/etc/group` file.
- Users can switch to one of their secondary groups temporarily using the newgrp command.
- Secondary groups are identified by their Group IDs (GIDs), which are numeric values associated with the groups.

Here's a summary of the files where primary and secondary group information is stored:

- `/etc/passwd`: This file contains user account information, including the primary group of each user.
- `/etc/group`: This file contains group information, including the names and members of each group, including both primary and secondary groups.

![Linux Groups](linux-groups.png)

### Managing File Permissions

```bash
comchmod u+x deploy.sh    # give the owning user execute permission
chmod g+x deploy.sh    # give the owning group execute permission
chmod o+x deploy.sh    # give everyone else execute permission
chmod ug+x deploy.sh   # to give the owning user and group execute permission
chmod ug-x deploy.sh   # to remove the execute permission from the owning user and group
```
