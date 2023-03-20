## vagrant-vm-docker
This is a repository that creates a VM with Vagrant to test Docker

## Purpose
Testing the exercises from the [Docker Up & running book](https://www.oreilly.com/library/view/docker-up/9781492036722/)

## Pre-requisites

1. Install [Vagrant](https://www.vagrantup.com/docs/installation)

2. Install [VirtualBox](https://www.virtualbox.org/)

## Clone the Repository

```shell
git clone git@github.com:dlavric/vagrant-vm-docker.git
cd vagrant-vm-docker
```

## How to use this repository

1. Start Vagrant:

```shell
vagrant up
```

2. Connect to the virtual machine:

```shell
vagrant ssh
```

# Install Docker - Chapter 3, Page 33
- Ensure there is no older Docker running
```shell
sudo apt-get remove docker docker-engine docker.io
```

- Add the required software dependencies and the *apt* repository for Docker
```shell
sudo apt-get update

sudo apt-get install \
apt-transport-https \
ca-certificates \
curl \
software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg |\
sudo apt-key add -

sudo add-apt-repository \
"deb [arch=amd64] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) \
stable"
```

- Install Docker
```shell
sudo apt-get update

sudo apt-get install docker-ce
```


# Chapter 6 - Exploring Docker

- Check the Docker version
```shell
docker version

Client: Docker Engine - Community
 Version:           23.0.1
 API version:       1.42
 Go version:        go1.19.5
 Git commit:        a5ee5b1
 Built:             Thu Feb  9 19:46:49 2023
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          23.0.1
  API version:      1.42 (minimum version 1.12)
  Go version:       go1.19.5
  Git commit:       bc3805a
  Built:            Thu Feb  9 19:46:49 2023
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.16
  GitCommit:        31aa4358a36870b21a992d3ad2bef29e1d693bec
 runc:
  Version:          1.1.4
  GitCommit:        v1.1.4-0-g5fd4c4d
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
  ```

 - Check the Docker server information (how many containers are running, on which OS is running etc.)
```shell
  docker info
Client:
 Context:    default
 Debug Mode: false
 Plugins:
  buildx: Docker Buildx (Docker Inc.)
    Version:  v0.10.2
    Path:     /usr/libexec/docker/cli-plugins/docker-buildx
  compose: Docker Compose (Docker Inc.)
    Version:  v2.16.0
    Path:     /usr/libexec/docker/cli-plugins/docker-compose
  scan: Docker Scan (Docker Inc.)
    Version:  v0.23.0
    Path:     /usr/libexec/docker/cli-plugins/docker-scan

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 23.0.1
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Using metacopy: false
  Native Overlay Diff: true
  userxattr: false
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Cgroup Version: 1
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: io.containerd.runc.v2 runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 31aa4358a36870b21a992d3ad2bef29e1d693bec
 runc version: v1.1.4-0-g5fd4c4d
 init version: de40ad0
 Security Options:
  apparmor
  seccomp
   Profile: builtin
 Kernel Version: 4.15.0-58-generic
 Operating System: Ubuntu 18.04.3 LTS
 OSType: linux
 Architecture: x86_64
 CPUs: 1
 Total Memory: 985.5MiB
 Name: dockerserver
 ID: 4eca8d4b-17f8-459d-8390-a076c76bd696
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Registry: https://index.docker.io/v1/
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false

WARNING: No swap limit support
```

- Download Image Updates
```shell
docker pull ubuntu:latest
```

- Start a Docker container and inspect it
```shell
docker run -d -t ubuntu /bin/bash
```

- List all the containers that are running in Docker
```shell
docker ps

CONTAINER ID   IMAGE     COMMAND       CREATED              STATUS              PORTS     NAMES
01420aeb5a3e   ubuntu    "/bin/bash"   About a minute ago   Up About a minute             charming_tharp
```

- We can inspect the container by specifying the `CONTAINER ID` or the `NAME` from the previous output
```shell
docker inspect charming_tharp
```

- Get inside the container using the `CONTAINER ID`
```shell
docker exec -i -t 01420aeb5a3e /bin/bash

root@01420aeb5a3e:/# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 14:33 pts/0    00:00:00 /bin/bash
root         8     0  0 15:30 pts/1    00:00:00 /bin/bash
root        15     8  0 15:30 pts/1    00:00:00 ps -ef
```

- Check the logs
```shell
docker logs -f 01420aeb5a3e
```
*NOTE*: Logs can be checked from a specific period with the `--since` option or a specific word with `--tail` followed by the keyword

- Start up a container where we can read `stats`
```shell
docker run -d ubuntu:latest sleep 1000
45579deea9d465e3923e61b16670cb3681d752288398f7c28c517b183d33bb43
```

- Get the ongoing stream of statistics from the container
NOTE: I installed `jq` (`sudo apt-get install jq`) on my VM so I can see the stats in a JSON format
```shell
curl --unix-socket /var/run/docker.sock \
http://v1/containers/45579deea9d465e3923e61b16670cb3681d752288398f7c28c517b183d33bb43/stats | jq '.'
```

# Chapter 7 - Debugging Containers

- Check what processes are running inside of the container:
```shell
docker top <container-id>
```

- Get a full tree of the PIDs of Docker:
```shell
pstree -p `pidof dockerd`
```


# Chapter 8 - Docker Compose

Docker Compose is a tool that is useful to create a stack of containers, helps to develoeprs to spin-un applications quickly.

Instead of using a shell script to create everything, Docker Compose uses a `yaml` configuration file to replace that.

- Check the configuration, an error will be prompted if the configuration is not good:
```shell
docker-compose config
```

- Build containers:
```shell
docker-compose build
```

- Start a web service with Docker:
```shell
docker-compose up -d
```

- Check the logs of the docker compose:
```shell
docker-compose logs
```
