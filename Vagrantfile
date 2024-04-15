# -*- mode: ruby -*-
# vim: set ft=ruby :

API_VERSION = "2"
VM_PROVIDER = "libvirt"
WORKERS = 1
VM_MEMORY = 2048
VM_CPUS = 2

Vagrant.configure(API_VERSION) do |config|

  # IMAGE
  config.vm.box = "debian/bullseye64"

  # INSTALL CURL
  config.vm.provision "shell", privileged: true do |s|
    s.inline = "apt update"
    s.inline = "apt install curl -y"
  end

  # INSTALL DOCKER
  config.vm.provision "shell", privileged: true do |s|
    s.inline = "curl -fsSL https://get.docker.com | sh"
  end

  # INSTALL PORTAINER
  config.vm.provision "shell", privileged: true do |s|
    s.inline = "docker volume create portainer data"
    s.inline = "/usr/bin/docker run -d -p 8000:8000 -p 9000:9000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest"
  end

  # DOCKER REGISTER
  config.vm.provision "shell", privileged: true do |s|
    s.inline = "docker login registry.site -u username -p password"
  end

  # NETWORK
  config.vm.network "public_network", 
    dev: "enp0s20f0u2c2"
  config.vm.network "private_network", 
    ip: "192.168.10.10"
  config.vm.network "forwarded_port",
    guest: 22,
    host: 2222

  # SETTINGS
  config.vm.provider VM_PROVIDER do |v|
    v.memory = VM_MEMORY
    v.cpus = VM_CPUS
  end

end
