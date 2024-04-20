# -*- mode: ruby -*-
# vim: set ft=ruby :
# VAGRANT_DISABLE_STRICT_DEPENDENCY_ENFORCEMENT=1 vagrant plugin install <plugin>

API_VERSION = "2"
# VM_PROVIDER = "libvirt"
# WORKERS = 1
# VM_MEMORY = 2048
# VM_CPUS = 2
# USERNAME = "username"
# PASSWORD = "password"
# REGISTRY = "registry_address_without_https"


Vagrant.configure(API_VERSION) do |config|

  # GENERAL
  config.vm.box = "debian/bullseye64"
  config.vm.synced_folder "./shared", "/vagrant", create: true
  config.env.enable

  # INSTALL CURL
  config.vm.provision "shell", privileged: true do |s|
    s.inline = "apt update"
    s.inline = "apt install curl httpie jq -y"
  end

  # INSTALL DOCKER
  config.vm.provision "shell", privileged: true do |s|
    s.inline = "curl -fsSL https://get.docker.com | sh"
  end

  # INSTALL PORTAINER
  config.vm.provision "shell", privileged: true do |s|
    s.inline = "/usr/bin/docker volume create portainer data"
    s.inline = "/usr/bin/docker run -d -p 8000:8000 -p 9000:9000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest --admin-password='$2y$05$5v0f1A2j/7yMaVwUHvWliO0MMAjpw8hJYgl0gL/J5VTUVGp6NudMW'"
  end

  # DOCKER REGISTER
  config.vm.provision "shell", privileged: true do |s|
    s.inline = "/usr/bin/docker login #{ENV['REGISTRY']} -u #{ENV['USERNAME']} -p #{ENV['PASSWORD']} || true"
  end

  # PULL CARAVEL
  config.vm.provision "shell", privileged: true do |s|
    s.inline = "/usr/bin/docker pull #{ENV['REGISTRY']}/#{ENV['IMAGE']} || true"
    s.inline = "/usr/bin/docker pull mysql:5.7 || true" 
  end

  # PORTAINER CONFIG STANDALONE MODE
  config.vm.provision "shell" do |s|
    s.inline = <<-SHELL
    JWT=$(http --ignore-stdin POST localhost:9000/api/auth Username="admin" Password="adminadminadmin" | jq -r ".jwt")
    http --ignore-stdin --form POST localhost:9000/api/endpoints \
        "Authorization: Bearer ${JWT}" \
        Name="localy" \
        EndpointCreationType=1 \
        URL="unix:///var/run/docker.sock"

    echo -n '{"authentication": true, "name": "senai_al_caravel", "password": "#{ENV['PASSWORD']}", "type": 3, "url": "#{ENV['REGISTRY']}", "username": "#{ENV['USERNAME']}"}' |\
    http POST localhost:9000/api/registries "Authorization: Bearer ${JWT}"
    SHELL
  end

  # PORTAINER ADDING CARAVEL STACK
  config.vm.provision "shell" do |s|
    s.inline = <<-SHELL
    JWT=$(http --ignore-stdin POST localhost:9000/api/auth Username="admin" Password="adminadminadmin" | jq -r ".jwt")  
    http POST localhost:9000/api/stacks/create/standalone/string?endpointId=1 "Authorization: Bearer ${JWT}" < /vagrant/caravel.json
    SHELL
  end
  
  # PORTAINER ADDING POSTGRES STACK
  config.vm.provision "shell" do |s|
    s.inline = <<-SHELL
    JWT=$(http --ignore-stdin POST localhost:9000/api/auth Username="admin" Password="adminadminadmin" | jq -r ".jwt")  
    http POST localhost:9000/api/stacks/create/standalone/string?endpointId=1 "Authorization: Bearer ${JWT}" < /vagrant/postgres.json
    SHELL
  end

  # NETWORK
  config.vm.network "public_network", 
    dev: "wlp0s20f3"
  config.vm.network "private_network", 
    ip: "192.168.10.10"
  config.vm.network "forwarded_port",
    guest: 22,
    host: 2222

  # SETTINGS
  config.vm.provider ENV['VM_PROVIDER'] do |v|
    v.memory = ENV['VM_MEMORY']
    v.cpus = ENV['VM_CPUS']
  end

end
