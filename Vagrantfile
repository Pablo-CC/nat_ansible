# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"



Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

#Misma clave para todas las máquinas
config.ssh.insert_key = false

 config.vm.define "gateway" do |host|
  host.vm.hostname = "gateway"
  host.vm.box = "ubuntu/trusty64"
  host.ssh.username = "vagrant"
  
  host.vm.provider :virtualbox do |vb|
   vb.customize ["modifyvm", :id, "--nic2", "intnet"]
  end
 
 end

 
 config.vm.define "hostA" do |host|
  host.vm.hostname = "hostA"
  host.vm.box = "ubuntu/trusty64"
  host.ssh.username = "vagrant"

  host.vm.provider :virtualbox do |vb|
   vb.customize ["modifyvm", :id, "--nic2", "intnet"] 
  end
 
 end

end
