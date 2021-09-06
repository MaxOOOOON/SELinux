# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure("2") do |config|
    config.vm.box = "generic/centos8"
    config.vm.synced_folder "../SELinux/", "/vagrant_files"
  
      config.vm.provision "shell", inline: <<-SHELL
      mkdir -p ~root/.ssh
      cp ~vagrant/.ssh/auth* ~root/.ssh
      sudo yum install -y nginx
      yum install -y policycoreutils-python-utils
      SHELL
  
  end
  
  
  