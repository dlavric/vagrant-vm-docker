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

# Chapter 9 - The path to production containers

Getting your applications to production on Docker:
1. Locally build and test a Docker image on your development box.
2. Build your official image for testing and deployment, usually from a CI or build
system.
3. Push the image to a registry.
4. Deploy your Docker image to your server, then configure and start the container.

As your workflow evolves, you will eventually collapse all of those steps into a single
fluid workflow:

1. Orchestrate the deployment of images and creation of containers on production
servers.

But there is a lot more to the story than that. At the most basic level, a production
story must encompass three things:
• It must be a repeatable process. Each time you invoke it, it needs to do the same
thing.
• It needs to handle configuration for you. You must be able to define your application’s
configuration in a particular environment and then guarantee that it will
ship that configuration on each deployment.
• It must deliver an executable artifact that can be started.

# Chapter 10 - Docker at scale

### Centurion

Centurion is an easy first step in moving from traditional deployments to a
Docker workflow and is a great option for people who aren’t ready for, or simply
don’t need, the features of Swarm, Kubernetes, or Mesos.

Centurion does not do much more than manage your container deployment in a reliable manner, 
and this makes it very easy to get started with. It assumes that a load balancer sits in 
front of your application instances.

- Check if you have ruby installed:
```shell
ruby -v
```

- Install Ruby then check the version:
```shell
sudo apt-get install ruby

vagrant@dockerserver:~$ ruby -v
ruby 2.5.1p57 (2018-03-29 revision 63029) [x86_64-linux-gnu]
```

- Once you have Ruby running, install Centurion with the Ruby package manager:
```shell
sudo gem install net-ssh -v 6.1.0
sudo apt-get install ruby-dev
sudo gem install centurion

Done installing documentation for bcrypt_pbkdf, ffi, rbnacl, rbnacl-libsodium, logger-colors, excon, trollop, centurion after 9 seconds
8 gems installed
```

- Invoke Centurion to make sure it is avaialble:
```shell
centurion --help

W, [2023-04-13T10:01:17.996696 #20058]  WARN -- : [DEPRECATION] The trollop gem has been renamed to optimist and will no longer be supported. Please switch to optimist as soon as possible.
Options:
  -p, --project=<s>          project (blog, forums...)
  -e, --environment=<s>      environment (production, staging...)
  -a, --action=<s>           action (deploy, list...) (default: list)
  -i, --image=<s>            image (yourco/project...)
  -t, --tag=<s>              tag (latest...)
  -h, --hosts=<s>            hosts, comma separated
  -d, --docker-path=<s>      path to docker executable (default: docker)
  -n, --no-pull              Skip the pull_image step
  --registry-user=<s>        user for registry auth
  --registry-password=<s>    password for registry auth
  -o, --override-env=<s>     override environment variables, comma separated
  -l, --help                 Show this message
```

- Deploy the public helloworld container and create directories to save the configuration:
```shell
cd /tmp
mkdir helloworld
cd helloworld
sudo gem install bundle
sudo gem install bundler -v 2.3.26

centurionize -p helloworld

Writing new Gemfile to /tmp/helloworld/Gemfile
Adding Centurion to the Gemfile


Remember to run `bundle install` before running Centurion

Done!
```

- Open the configuration file automatically generated in `config/centurion/helloworld.rake`:
```shell
cat config/centurion/helloworld.rake

namespace :environment do
  task :common do
    set :image, 'helloworld'
    # Point this to an appropriate health check endpoint for rolling deploys (defaults to '/')
    # set :status_endpoint, '/status/check'
     # Example on how to change docker registry to Dogestry.
    # This requires:
    # - aws_access_key_id
    # - aws_secret_key
    # - s3_bucket
    #
    # And optionally:
    # - s3_region (default to us-east-1)
    #
    # registry :dogestry
    # set :aws_access_key_id, 'abc123'
    # set :aws_secret_key, 'xyz'
    # set :s3_bucket, 'bucket-for-docker-images'
  end
   desc 'Staging environment'
  task :staging => :common do
    set_current_environment(:staging)
    # env_vars YOUR_ENV: 'staging'
    # host_port 10234, container_port: 9292
    # host 'docker-server-staging-1.example.com'
    # host 'docker-server-staging-2.example.com'
     # You can assign different docker daemon port. Example:
    # host 'docker-server-staging-3.example.com:4243'
  end
   desc 'Production environment'
  task :production => :common do
    set_current_environment(:production)
    # env_vars YOUR_ENV: 'production'
    # host_port 23235, container_port: 9293
    # host 'docker-server-prod-1.example.com'
    # host 'docker-server-prod-2.example.com'
    # host_volume '/mnt/volume1', container_volume: '/mnt/volume1'
  end
end
```

- Modify the file `config/centurion/helloworld.rake` with the following content:
```
namespace :environment do
desc 'Development environment'
task :development do
set_current_environment(:development)
# Add on the line after 'set_current_environment(:development)'
ssh_user = ENV['USER']
puts "[Info] Will SSH using: #{ssh_user}"
set :ssh, true
set :ssh_user, ssh_user
set :ssh_log_level, Logger::WARN
set :image, 'adejonge/helloworld'
env_vars MY_ENV_VAR: 'something important'
host_port 8080, container_port: 8080
host '192.168.2.10'
host '192.168.2.25'
end
end
```

- You’re now ready to deploy this to your development environment:
```shell
centurion -p helloworld -e development -a rolling_deploy

** Execute deploy:get_image
** Invoke deploy:pull_image (first_time)
** Execute deploy:pull_image
I, [2023-04-13T12:39:12.255289 #20217]  INFO -- : Fetching image adejonge/helloworld:latest IN PARALLEL

I, [2023-04-13T12:39:12.256039 #20217]  INFO -- : Using CLI to pull
I, [2023-04-13T12:39:12.257097 #20217]  INFO -- : Using CLI to pull
```

### Docker Swarm Mode

After building the container runtime in the form of the Docker engine, the engineers
at Docker turned to the problems of orchestrating a fleet of individual Docker hosts
and effectively packing those hosts full of containers. The first tool that evolved from
this work was called Docker Swarm.

The Docker Swarm mode lets you build your own Docker cluster for deployment.

- Use my server as a Swarm Manager:
```shell
sudo docker swarm init --advertise-addr 192.168.57.151

Swarm initialized: current node (6q488hpnkeezttptcxfne82l2) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-4on43snjhexap37fm83kikhfeyic6usuhmmsr0usgglk9ps4ka-7gg71vc41e74w71dcz0q4iadt 192.168.57.151:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

- Save the token previously provided:
```shell
SWMTKN-1-4on43snjhexap37fm83kikhfeyic6usuhmmsr0usgglk9ps4ka-7gg71vc41e74w71dcz0q4iadt
```

- If the token is lost, it can still be retrieved by the following command:
```shell
sudo docker swarm join-token --quiet worker
```

- Inspect my progress by running my local docker client pointed at the 
new manager node’s IP address:
```shell
docker -H 192.168.57.151 info
```

- If there are issues check the errors with and check the errors:
```shell 
systemctl status docker.service
```

- Restart Docker:
```shell
sudo systemctl restart docker
```

- List the nodes from the cluster:
```shell
docker -H 192.168.57.151  node ls
```

By default Swarm will prioritize ensuring that you have the number of instances 
that you requested over spreading individual containers across hosts when possible. 
If you don’t have enough nodes,you will get multiple copies on each node. In a real-world scenario, 
you need to think carefully about placement and scaling. You might not be able to get away with running
multiple copies on the same host in the event that you lose a whole node.

### Amazon ECS and Fargate

Amazon has spent a lot of engineering time building a service
that treats containers as first-class citizens: the Elastic Container Service (ECS).
In the last few years they have built upon this support with products like the ECS for
Kubernetes (EKS) and, more recently, Fargate.

Fargate is simply a marketing label Amazon uses for the new feature
of ECS that makes it possible for AWS to automatically manage
all the nodes in your container cluster so that you can focus on
deploying your service.

To work with the ECS, you need to create a role called ecsInstanceRole that has the
AmazonEC2ContainerServiceforEC2Role managed role attached to it.

- Create a cluster in the container service:
```shell
aws ecs create-cluster --cluster-name fargate-testing
```

Before AWS Fargate was released, you were required to create AWS EC2 instances
running docker and the ecs-agent and add them into your cluster. You can still use
this approach if you want (EC2 launch type), but Fargate makes it much easier to
run a dynamic cluster that can scale fluidly with your workload.

- Create your first task definition, copy in the following
JSON, and then save it as `webgame-task.json` in your current directory:
```json
{
"containerDefinitions": [
{
"name": "web-game",
"image": "spkane/quantum-game",
"cpu": 0,
"portMappings": [
{
"containerPort": 8080,
"hostPort": 8080,
"protocol": "tcp"
}
],
"essential": true,
"environment": [],
"mountPoints": [],
"volumesFrom": []
}
],
"family": "fargate-game",
"networkMode": "awsvpc",
"volumes": [],
"placementConstraints": [],
"requiresCompatibilities": [
"FARGATE"
],
"cpu": "256",
"memory": "512"
}
```

- To upload this task definition to Amazon, you will need to run a similar command:
```shell
aws ecs register-task-definition --cli-input-json file://./webgame-task.json
```

- List all the task definitions:
```shell
aws ecs list-task-definitions
```

- List all the services in the cluster:
```shell
aws ecs list-services --cluster fargate-testing
```

- Retrieve all the details about my service:
```shell
aws ecs describe-services --cluster fargate-testing \
--services fargate-game-service
```

- List individual tasks running in the cluster:
```shell
aws ecs list-tasks --cluster fargate-testing
```

# Chapter 11 - Advanced Topics

At the risk of mixing metaphors, we might make a comparison to a hotel. When running
Docker, your computer is the hotel. Without Docker, it’s more like a hostel with open
bunk rooms. In our modern hotel, each container that you launch is an individual
room with one or more guests (our processes) in it.

Namespaces make up the walls of the room, and ensure that processes cannot interact
with neighboring processes in any ways that they are not specifically allowed to. Control
groups are like the floor and ceiling of the room, trying to ensure that the inhabitants
have the resources they need to enjoy their stay, without allowing them to use
resources or space reserved for others. Imagine the worst kind of noisy hotel neighbors
and you can really appreciate good, solid barriers between rooms. Finally, SELinux
and AppArmor are a bit like hotel security, ensuring that even if something
unexpected or untoward happens, it is unlikely to cause much more than the headache
of filling out paperwork and filing an incident report.

### cgroups 

Control groups, or cgroups for short, allow you to set limits on resources for processes
and their children. This is the mechanism that Docker uses to control limits on memory,
swap, CPU, and storage and network I/O resources.

Every Docker container is assigned a cgroup that is unique to that container. All of
the processes in the container will be in the same group. This means that it’s easy to
control resources for each container as a whole without worrying about what might
be running.

We can find our container’s cgroup in the /sys filesystem.
/sys is laid out so that each type of setting is grouped into a module and that
module is exposed at a different place in the /sys filesystem:
```shell
ls /sys/fs/cgroup/cpu/docker/<container-id>

cgroup.clone_children cpu.cfs_period_us cpu.rt_runtime_us notify_on_release
cgroup.event_control cpu.cfs_quota_us cpu.shares tasks
cgroup.procs cpu.rt_period_us cpu.stat
```

Inspect CPU shares for the container:
```shell
cat /sys/fs/cgroup/cpu/docker/dcbbaa763daf/cpu.shares
512
```

### Namespaces

Inside each container, you see a filesystem, network interfaces, disks, and other
resources that all appear to be unique to the container despite sharing the kernel with
all the other processes on the system.

Namespaces take a traditionally global
resource and present the container with its own unique and unshared version of that
resource.

Rather than just having a single namespace, however, containers have a namespace
on each of the six types of resources that are currently namespaced in the kernel:
mounts, UTS, IPC, PID, network, and user namespaces. Essentially when you talk
about a container, you’re talking about a number of different namespaces that Docker
sets up on your behalf.

### Security

Containerized applications are more secure than noncontainerized applications
because cgroups and standard namespaces provide some important isolation from
the host’s core resources. But you should not think of containers as a substitute for
good security practices.

### Docker Daemon

From a security standpoint, the Docker daemon and its components are the only
completely new risk you are introducing to your infrastructure. Your containerized
applications are not any less secure and are, at least, a little more secure than they
would be if deployed outside of containers. But without the containers, you would
not be running dockerd, the Docker daemon.

You need to set up your own certificate authority in most cases. If
your organization already has one, then you are in luck. Certificate management
needs to be implemented carefully in any organization, both to keep certificates
secure and to distribute them efficiently. So, given that, here are the basic steps:
1. Set up a method of generating and signing certificates.
2. Generate certificates for the server and clients.
3. Configure Docker to require certificates with --tlsverify.

### Advanced configuration

The Structure of Docker

Docker is made of five major server-side components that
present a common front via the API. These parts are dockerd, containerd, runc,
docker-containerd-shim, and the docker-proxy.
Dockerd is responsible for orchestrating the whole set of components that make up
Docker. But when it starts a container, Docker relies on containerd to handle instantiating
the container.

Let’s take a look at the components and see what each of them does:

- dockerd
One per server. Serves the API, builds container images, and does high-level network
management including volumes, logging, statistics reporting, and more.

- containerd
One per server. Manages lifecycle, execution, copy-on-write filesystem, and lowlevel
networking drivers.

- docker-containerd-shim
One per container. Handles file descriptors passed to the container (e.g., stdin/
out) and reports exit status.

- runc
Constructs the container and executes it, gathers statistics, and reports events on
lifecycle.

# Chapter 12 - Container Platform Design

**The Twelve-Factor App**

Applications built with these 12 steps in mind are ideal candidates
for the Docker workflow:

1.Codebase
2.Dependencies
3.Config
4.Backing Services
5.Build,Release,Run
6.Processes
7.Port binding
8.Concurrency
9.Disposability
10.Development/Production Parity
11.Logs
12.Admin processes

Twelve-Factor Wrap-Up

While “The Twelve-Factor App” wasn’t written as a Docker-specific manifesto, almost
all of this advice can be applied to writing and deploying applications on a Docker
platform.

**The Reactive Manifesto**

“The Reactive Manifesto” states that “reactive systems” are responsive, resilient, elastic,
and message-driven.


