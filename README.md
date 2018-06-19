#NAT + Ansible#

The goal of this task is to build a virtual environment using Vagrant and authomatize the whole deployment with Ansible. The environment will consist of two machines: one will be a regular host and the other one will act as a gateway for external connections.

##Requirements##
*Vagrant
*VirtualBox
*Ansible


##Network infraestructure##
In the virtual environment we will see two networks.
####10.0.0.0/16####
This network is shared by VirtualBox and both VM. VirtualBox acts like a router between your PC and the Virtual Env. This network is already deployed by Vagrant.
IP addresses:
*10.0.2.2 -> VirtualBox
*10.0.0.15 -> hostA and gateway
####192.168.10.0/24####
This is the private network we have to configure with Ansible. It will be shared by hostA and gateway.
IP addresses:
*192.168.10.1 -> gateway
*192.168.10.2 -> hostA

By default, Vagrant will configure both hosts to send external packets through *10.0.2.2* (VirtualBox) so we have to configure hostA's IP static route table so that it sends them through gateway (using Ansible's __interfaces_file__ module). We will also activate IP Forwarding on gateway and use __iptable__ module to provide NAT functioning.


##Explanation##
###Vagrantfile###
This file is used by Vagrant to build up the virtual machines (VM). With these blocks:

~~~
host.vm.provider :virtualbox do |vb|
   vb.customize ["modifyvm", :id, "--nic2", "intnet"]
  end
~~~
We only allow the VMs to communicate with each other.

###inventory###
Ansible needs to know about the existence of both remote machines and how to connect with them via SSH. This files does that.

###ansible.cfg###
In this file we just define some default configuration parameters for Ansible.

###playbook.yml###
This file contains all the tasks executed by Ansible on the remote hosts. Some comments on this tasks:

*gateway's private interface (name eth1) is defined using a template file with the configuration parameters that will be located on the VM in /etc/network/interfaces.d/eth1.cfg. These changes will be made after executing ~~~ifup eth1~~~
*hostA's private interface (name eth1) is defined using Ansible's __interfaces_file__ module wich modifies /etc/network/interfaces file in the VM. Once again, we need to execute ~~~ifup eth1~~~. Note we first need to add two lines to this file (using __lineinfile__ module) so that the VM recognizes the interface.
*hanlders are executed by __notify__ clauses only if the task applys any changes in the remote machine.
*the _with_items_ clause is used to perform loops in a task (i.e: execute the task multiple times changing its parameters).


###Execution###
~~~
vagrant up
ansible-playbook playbook.yml
~~~

###Test###
You can connect using SSH to one of the VMs with:
~~~
vagrant ssh hostA | gateway
~~~
You should see the shell login screen.
To verifiy that the playbook does what it should you can ping from hostA to any public IP.
For example:
~~~
ping www.aupaathletic.com -c 1
~~~

The ping should work without problems. You can capture the ping in one of gateway's interfaces using tcpdump:
~~~
tcpdump -i eth0|eth1 -n icmp
~~~

