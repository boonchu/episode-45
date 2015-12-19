# Defines our Vagrant environment
#
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  # default VM settings
  config.ssh.username = 'vagrant'
  config.ssh.insert_key = 'true'

  # create mgmt node
  config.vm.define :mgmt do |mgmt_config|
      #mgmt_config.vm.box = "ubuntu/trusty64"
      mgmt_config.vm.box = "puppetlabs/centos-7.0-64-nocm"
      mgmt_config.vm.hostname = "mgmt"
      mgmt_config.vm.network :private_network, ip: "10.0.15.10"
      mgmt_config.vm.provider "virtualbox" do |vb|
        vb.memory = "256"
      end
      mgmt_config.vm.provision :shell, path: "bootstrap-mgmt.sh"
      mgmt_config.vm.provision :shell, path: "init.sh"
  end

  # create load balancer
  config.vm.define :lb do |lb_config|
      #lb_config.vm.box = "ubuntu/trusty64"
      lb_config.vm.box = "puppetlabs/centos-7.0-64-nocm"
      lb_config.vm.hostname = "lb"
      lb_config.vm.network :private_network, ip: "10.0.15.11"
      lb_config.vm.network "forwarded_port", guest: 80, host: 8080
      lb_config.vm.provider "virtualbox" do |vb|
        vb.memory = "256"
      end
      lb_config.vm.provision :shell, path: "init.sh"
  end

  # create some web servers
  # https://docs.vagrantup.com/v2/vagrantfile/tips.html
  (1..6).each do |i|
    config.vm.define "web#{i}" do |node|
        #node.vm.box = "ubuntu/trusty64"
        node.vm.box = "puppetlabs/centos-7.0-64-nocm"
        node.vm.hostname = "web#{i}"
        node.vm.network :private_network, ip: "10.0.15.2#{i}"
        node.vm.network "forwarded_port", guest: 80, host: "808#{i}"
        node.vm.provider "virtualbox" do |vb|
          vb.memory = "256"
        end
        node.vm.provision :shell, path: "init.sh"
    end
  end

end
