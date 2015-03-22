# DevStack in a Vagrant VM

This repository contains a Vagrantfile and an accompanying Ansible playbook
that sets up a VirtualBox virtual machine that installs [DevStack](http://devstack.org).

Ansible generates a `local.conf` file that defaults to:

 * Use Neutron for networking
 * Install Swift for object storage
 * Install Solum for Application development
 * Install Tempest for functional testing

## Memory usage

By default, the VM uses 8GB of RAM. If you want to change this, edit the
following line in Vagrantfile:

    vb.customize ["modifyvm", :id, "--memory", 8192]

## Prereqs

Install the following applications on your local machine first:

 * [VirtualBox](http://virtualbox.org)
 * [Vagrant](http://vagrantup.com)
 * [Ansible](http://ansibleworks.com)
 
To Install Ansible on Mac OSX
Ansible uses Python and fortunately Python is already installed on modern versions of OSX.

Quick summary:

Install Xcode

```sudo easy_install pip```
```sudo pip install ansible```

Then, if you would like to update Ansible later, just do:

```sudo pip install ansible --upgrade```

## Boot the virtual machine and install DevStack

    git clone https://github.com/jolsenatx/devstack-vagrant
    cd devstack-vagrant
    vagrant up

The `vagrant up` command will:

 1. Download an Ubuntu 13.10 (saucy) vagrant box if it hasn't previously been downloaded to your machine.
 2. Boot the virtual machine (VM).
 3. Clone the DevStack git repository inside of the VM.
 4. Run DevStack inside of the VM.
 5. Add eth2 to the br-ex bridge inside of the VM to enable floating IP access from the host machine.

It will take at least ten minutes for this to run, and possibly much longer depending on your internet connection and whether it needs to download the Ubuntu vagrant box.

## Allow VMs to connect to the internet (Linux hosts only)

By default, VMs started by OpenStack will not be able to connect to the
internet. If you want your VMs to connect out, and you are running Linux
as your host operating system, you must configure your host machine to do
network address translation (NAT).

To enable NAT, issue the following commands in your host, as root:

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

## Troubleshooting

### Authentication or permission failure

If you see an error like this:

```
devstack | FAILED => Authentication or permission failure. In some cases, you may have been able to authenticate and did not have permissions on the remote directory
```

Then you may have incorrect file permissions in the id_vagrant file. Make sure it is only readable by the owner, by doing:

    chmod 0600 id_vagrant

### Fails to connect

You may ocassionally see the following error message:

```
[default] Waiting for VM to boot. This can take a few minutes.
[default] Failed to connect to VM!
Failed to connect to VM via SSH. Please verify the VM successfully booted
by looking at the VirtualBox GUI.
```

If you see this, retry by doing:

    vagrant destroy --force && vagrant up


## Logging in the virtual machine

The VM is accessible at 192.168.27.100

The easiest way to log in the VM, is to type `vagrant ssh`

You can also use access it using `vagrant` as username and password, or use the provided `id_vagrant` private key to avoid typing a password.

Note that you do not need to be logged in to the VM to run commands against the OpenStack endpoint.

## Loading OpenStack credentials

From your local machine, to run as the demo user:

    source credentials/demo.openrc

To run as the admin user:

    source credentials/admin.openrc

## Horizon

* URL: http://192.168.27.100
* Username: admin or demo
* Password: password

## Initial networking configuration

![Network topology](assets/topology.png)

DevStack configures an internal network ("private") and an external network ("public"), with a router ("router1") connecting the two together. The router is configured to use its interface on the "public" network as the gateway.
