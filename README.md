
openstack-playbooks
===========================

Included in this repo are the ansible playbooks I use to build
my openstack cluster.  They are constantly being changed
and, of course, your computers will very likely have
different components and network interfaces that mine.  
So the playbooks provided here are more for guidance than anything else.

All these playbooks were based on the instructions at openstack.org
for installing a Newton release openstack cluster.

The playbooks create two versions of an openstack cluster.
The first is a standalone version, with a nova/compute agent
on the same node as the controller.  The standalone cluster is
the easiest to build and to test.

The second version is a non-standalone cluster.  Here the controller
resides on its own node.  Any compute agents are placed on 
other nodes.

The playbooks also build a nvidia card passthrough capability
into each node.   The relevant files are control\_passhthru.yml
and agent\_passthru.yml.  The imports of these tasks can
be commented out.

To start the install, I use this command: 

    ansible-playbook stack.yml -vv -e hosts=${HOSTS} \
    -e hostname=control -e no_private_key= \
    -e myif=enp0s31f6 -e provider_if=enp4s0 \
    -e myip=192.168.0.6 -e gateway=192.168.0.1 -e nameserver=192.168.0.1

Here, $HOST 
is the address of the computer for ansible to ssh into.  The user
is assumed to be your username with the password set to passw0rd.
The ssh address can be the DHCP address of the node.  Using
an environmental variable for setting "hosts" is useful when 
the address is initially set by dhcp.

The playbook will change the address to a static address if 
"myip" and "myif" are set.  

Two ethernet ports need to be provided, one management and 
the other provider.  No ip address is needed for the provider 
network port, since it is set to manual.  Both need access to
the internet.

The playbook builds a very minimal Ubuntu server with the
the appropriate network settings and the passed hostname.  

Please be aware that adding a graphic card might
change the name of one of your network ports.
Your public ssh key will also be copied over as an authorized key.
It will then reboot the server.

Next you need to build the openstack controller.  I use the following
playbook to build it.  The parameter "hosts" is now set to the new 
static ip address of the server.

    ansible-playbook stack_control.yml -vv -e hosts=192.168.0.6 \
    -e provider_if=enp4s0 -e myip=192.168.0.6

If you want to build a standalone cluster, the next command is used.

    ansible-playbook stack_local.yml -vv -e hosts=192.168.0.6 \
    -e provider_if=enp4s0

Then you can create instances and test it.

To add a nova/compute node to your openstack cluster, build the node
as a base server again, passing its interface names and its static ip address.
For example, I run:

    ansible-playbook stack.yml -vv -e hosts=${HOSTS} \
    -e hostname=compute -e no_private_key= \
    -e myif=eno1 -e provider_if=enp4s0
    -e myip=192.168.0.7 -e gateway=192.168.0.1 -e nameserver=192.168.0.1

Next we install the nova/compute agent onto the node.  The master\_ip is the
address of the controller node.

    ansible-playbook stack_nova.yml -vv -e hosts=192.168.0.7 \
    -e myip=192.168.0.7 -e master_ip=192.168.0.6 \
    -e provider_if=enp4s0 

After all of this is done, ssh into the controller node to build
images and create servers.  A few helper bash scripts will be 
in your home directory.  

Run "bash status.sh" to see the status of the cluster.
Run "bash build.sh" to create the provider network as well as 
download debian and ubuntu images and register them with the cluster.
The script will also create a security group and upload your public key
for the demo user

Run "bash list.sh" to show your running instances.  Run "bash kill.sh"
will kill an instance.  To create an instance run either "bash ubuntu.sh"
or "bash debian.sh".  The script nvidia.sh will create a ubuntu
based instance with an nvidia card attached.  (The nvidia card must
be whitelisted in /etc/nova/nova.conf on the controller. At the moment
only the 1050Ti and 750Ti cards are whitelisted.)

copyright (c) 2017 roseengineering




