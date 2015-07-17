# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # cluster nodes
  hosts = {
            :app1 => { :name => "node1", :address => "10.10.2.20" }, 
            :app2 => { :name => "node2", :address => "10.10.2.21" }  
          }

  # Host setup template
  hosts.each do |host, params|
    config.vm.define "#{ params[:name] }" do |host|
      host.vm.box = "Ubuntu precise 64 VirtualBox"
      host.vm.box_url = " http://files.vagrantup.com/precise64.box"

      host.vm.provider :virtualbox do |vb|
        vb.customize [
                         "modifyvm", :id,
                         "--name", "cluster-#{ params[:name] }",
                         "--memory", 512,
                         "--cpus", 1,
                     ]
      end
      host.vm.network :private_network, ip: "#{ params[:address] }"
      host.vm.hostname = "#{ params[:name] }.cluster-seed"
      host.vm.provision "shell", inline: "sudo apt-get install python-httplib2" # Required to use 'uri' Ansible module
    end
  end
end
