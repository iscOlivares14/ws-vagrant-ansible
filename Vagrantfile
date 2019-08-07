# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.define "webappserver" do |webappserver|
     webappserver.vm.box = "ubuntu/bionic64"
     webappserver.vm.network "forwarded_port", guest: 8000, host: 8080
     webappserver.vm.provider "virtualbox" do |vb|
          vb.memory = "2048"
          vb.name = "WebApplicationServer"
          vb.cpus = 2
        end
  end
end
