# -*- mode: ruby -*-
# vi: set ft=ruby :
# Before execute this Vagrant File, Type the command below in your Linux Terminal
#   export VAGRANT_EXPERIMENTAL="disks"

# Vagrantfile configurado para provisionamento da VM conforme especificado no projeto
Vagrant.configure("2") do |config|
  
  config.vm.box = "roboxes/ubuntu2204"
  
  config.vm.provider "virtualbox" do |vb|
    vb.name = "SamuelSilva"
    vb.memory = "1024"
    vb.cpus = 1
  end

  config.vm.network "private_network", ip: "192.168.57.10"

  # Configuração do Ansible
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yml"
    ansible.extra_vars = {
      hostname: "server.samuel.silva"
    }
  end

end

