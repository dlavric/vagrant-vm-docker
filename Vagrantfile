# -*- mode: ruby -*-
# vi: set ft=ruby :

$script = <<-SCRIPT
which docker 2>/dev/null || {
  export DEBIAN_FRONTEND=noninteractive
  apt-get update
  apt-get install -y docker.io
  hash -r
  docker version
  docker run hello-world
}
SCRIPT



Vagrant.configure("2") do |config|
    config.vm.box = "hashicorp/bionic64"
    config.vm.hostname = "dockerserver"
    config.vm.network "private_network", ip: "192.168.57.151"
    config.vm.provider "virtualbox"
    config.vm.provision "shell", inline: $script
end
