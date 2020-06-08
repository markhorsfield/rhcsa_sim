# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.

BOX_IMAGE = "centos/8"
NODE_COUNT = 2

  ##### DEFINE VM for nodeXX #####
  (1..NODE_COUNT).each do |i|
    config.vm.define "node#{i}" do |node|
      node.vm.hostname = "node#{i}"
      #node.vm.network :private_network, ip: "10.0.0.#{i + 10}"
      node.vm.box = BOX_IMAGE
      node.vm.provider :libvirt do |v|
        v.memory = "512"
      end
    end
  end
  
  ##### shell #####
  # set ssh server to allow remote login 
  # create ssh keypair  
  # (todo) copy public key to all remote nodes 
  config.vm.provision "shell", inline: <<-SHELL 
    sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
    sudo systemctl restart sshd
    ssh-keygen -t rsa -b 4096 -q -f /home/vagrant/.ssh/id_rsa -P ""
    sudo chown vagrant:vagrant /home/vagrant/.ssh/id_rsa
  SHELL

  ##### ansible #####
  # install new packages 
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.main.yml"
    ansible.groups = {
      "server" => ["node[1:2]"],
    }
  end
end
