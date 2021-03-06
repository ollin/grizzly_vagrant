# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box = "grizzly_base"
  config.vm.box = "precise32"

  # The url from where the 'config.vm.box' box will be fetched if it
  # doesn't already exist on the user's system.
  config.vm.box_url = "http://files.vagrantup.com/precise32.box"

  config.vm.provision :shell, :path => "basic_setup.sh"

  config.vm.define :controller do |controller_config|
    controller_config.vm.hostname = "controller"
    config.vm.provider :virtualbox do |vb|

      vb.customize ["modifyvm", :id, "--memory", "1200"]
      vb.customize ["modifyvm", :id, "--rtcuseutc", "on"]
      file_to_disk = './disks/cinder_vol.vdi'

      vb.customize ['createhd', '--filename', file_to_disk, '--size', 100 * 1024]
      vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', file_to_disk]
    end
    controller_config.vm.network :private_network, ip: "10.0.0.3" # eth1 mgt
    controller_config.vm.network :private_network, ip: "172.16.0.3" # eth2 tenant api
    controller_config.vm.network :forwarded_port, guest: 80, host: 8080
    config.vm.provision :shell, :path => 'controller_setup.sh'
  end

  config.vm.define :compute1 do |compute1_config|
    compute1_config.vm.hostname = "compute1"
    config.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "1024"]
      vb.customize ["modifyvm", :id, "--rtcuseutc", "on"]
    end
    compute1_config.vm.network :private_network, ip: "10.0.0.20" # eth1 mgt
    compute1_config.vm.network :private_network, ip: "172.16.0.20" # eth2 external net
    compute1_config.vm.network :private_network, ip: "10.0.99.20" #  eth3 dummy
    compute1_config.vm.provision :shell, :inline => "ip link set mtu 1546 dev eth3"
    config.vm.provision :shell, :path => 'compute_setup.sh'
  end


  config.vm.define :netnode do |netnode_config|
    netnode_config.vm.hostname = "netnode"
    config.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "512"]
      vb.customize ["modifyvm", :id, "--rtcuseutc", "on"]
      vb.customize ["modifyvm", :id, "--nicpromisc4", "allow-all"]
    end
    netnode_config.vm.network :private_network, ip: "10.0.0.7" # eth1 mgt
    netnode_config.vm.network :private_network, ip: "172.16.0.7" # eth2 external net
    netnode_config.vm.network :private_network, ip: "10.0.99.7" # eth3 OpenStack Private Network
    netnode_config.vm.provision :shell, :inline => "ip link set mtu 1546 dev eth2"

    config.vm.provision :shell, :path => 'netnode_setup.sh'
  end


end
