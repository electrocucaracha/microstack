# -*- mode: ruby -*-
# vi: set ft=ruby :
##############################################################################
# Copyright (c)
# All rights reserved. This program and the accompanying materials
# are made available under the terms of the Apache License, Version 2.0
# which accompanies this distribution, and is available at
# http://www.apache.org/licenses/LICENSE-2.0
##############################################################################

$no_proxy = ENV['NO_PROXY'] || ENV['no_proxy'] || "127.0.0.1,localhost"
(1..254).each do |i|
  $no_proxy += ",10.0.2.#{i}"
end
$debug = ENV['DEBUG'] || "true"
$num_instances = ENV['NUM_COMPUTES'] || 3

Vagrant.configure("2") do |config|
  config.vm.provider :libvirt
  config.vm.provider :virtualbox

  config.vm.box = "generic/ubuntu2004"
  config.vm.box_check_update = false
  config.ssh.insert_key = false
  config.vm.synced_folder "./", "/vagrant"

  # Install microstack packages
  config.vm.provision 'shell', inline: <<-SHELL
    set -o errexit
    snap install microstack --devmode --beta
  SHELL

  config.vm.provider "virtualbox" do |v|
    v.gui = false
    v.customize ["modifyvm", :id, "--nested-hw-virt", "on"]
  end

  config.vm.provider :libvirt do |v|
    v.random_hostname = true
    v.management_network_address = "10.0.2.0/24"
    v.management_network_name = "administration"
    v.cpu_mode = 'host-passthrough'
    v.nested = true
  end

  if ENV['http_proxy'] != nil and ENV['https_proxy'] != nil
    if Vagrant.has_plugin?('vagrant-proxyconf')
      config.proxy.http     = ENV['http_proxy'] || ENV['HTTP_PROXY'] || ""
      config.proxy.https    = ENV['https_proxy'] || ENV['HTTPS_PROXY'] || ""
      config.proxy.no_proxy = $no_proxy
      config.proxy.enabled = { docker: false }
    end
  end

  # OpenStack controller configuration
  config.vm.define :control, primary: true do |control|
    control.vm.hostname = "control"
    control.vm.network :private_network, ip: "192.168.123.10", libvirt__network_name: "management"

    %i[virtualbox libvirt].each do |provider|
      control.vm.provider provider do |p|
        p.cpus = 2
        p.memory = 8192
      end
    end
    # Setup SSH keys on target nodes
    control.vm.provision "shell", inline: <<-SHELL
      mkdir -p /root/.ssh
      cat /vagrant/insecure_keys/key.pub | tee /root/.ssh/authorized_keys
      chmod 700 ~/.ssh
      chmod 600 ~/.ssh/authorized_keys
      sudo sed -i '/^PermitRootLogin no/d' /etc/ssh/sshd_config
    SHELL

    control.vm.provision 'shell', inline: <<-SHELL
      set -o errexit
      microstack init --auto --control

      # admin's password:
      snap get microstack config.credentials.keystone-password
    SHELL
  end

  # OpenStack Compute configuration
  (1..$num_instances).each do |i|
    config.vm.define vm_name = "compute%01d" % i , autostart: false do |compute|
      compute.vm.hostname = vm_name
      compute.vm.network :private_network, ip: "192.168.123.1%01d" % i, libvirt__network_name: "management"
  
      %i[virtualbox libvirt].each do |provider|
        compute.vm.provider provider do |p|
          p.cpus = 2
          p.memory = 4096
        end
      end
  
      compute.vm.provision "shell", privileged: false, inline: <<-SHELL
        cd /vagrant
        sudo mkdir -p /root/.ssh/
        sudo cp insecure_keys/key /root/.ssh/id_rsa
        cp insecure_keys/key ~/.ssh/id_rsa
        sudo chmod 400 /root/.ssh/id_rsa
        chown "$USER" ~/.ssh/id_rsa
        chmod 400 ~/.ssh/id_rsa
      SHELL
  
      compute.vm.provision 'shell', inline: <<-SHELL
        set -o errexit
        microstack init --auto --compute --join $(ssh -o StrictHostKeyChecking=no 192.168.123.10 microstack add-compute | tail -1)
      SHELL
    end
  end
end
