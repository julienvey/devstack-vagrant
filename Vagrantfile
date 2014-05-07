# -*- mode: ruby -*-
# vi: set ft=ruby :

$solum_prepare = <<SCRIPT
    mkdir -p /opt/stack/
    mkdir -p /opt/stack/solum
    git clone git://github.com/stackforge/solum.git /opt/stack/solum
    cd /opt/stack/solum/contrib/devstack
    cp lib/solum /opt/stack/devstack/lib
    cp extras.d/70-solum.sh /opt/stack/devstack/extras.d
SCRIPT

$murano_prepare = <<SCRIPT
    mkdir -p /opt/stack/
    mkdir -p /opt/stack/murano-api
    git clone git://github.com/stackforge/murano-api.git /opt/stack/murano-api
    cd /opt/stack/murano-api/contrib/devstack
    cp lib/murano /opt/stack/devstack/lib
    cp lib/murano-dashboard /opt/stack/devstack/lib
    cp extras.d/70-murano.sh /opt/stack/devstack/extras.d
SCRIPT

$nova_docker_prepare = <<SCRIPT
    mkdir -p /opt/stack/
    mkdir -p /opt/stack/nova-docker
    git clone git://github.com/stackforge/nova-docker.git /opt/stack/nova-docker
    cd /opt/stack/nova-docker/contrib/devstack
    cp lib/nova_plugins/hypervisor-docker /opt/stack/devstack/lib/nova_plugins
    cp extras.d/70-docker.sh /opt/stack/devstack/extras.d
    # Hack around https://bugs.launchpad.net/nova-docker/+bug/1309490
    mkdir -p /opt/stack/nova
    git clone git://github.com/openstack/nova.git /opt/stack/nova
SCRIPT

$stack_sh_run = <<SCRIPT
    cd /opt/stack/devstack;
    ./stack.sh
SCRIPT

$solum_dashboard_install = <<SCRIPT
   mkdir -p /opt/stack/solum-dashboard
   git clone git://github.com/stackforge/solum-dashboard.git /opt/stack/solum-dashboard
   sudo pip install -e /opt/stack/solum-dashboard
   cd /opt/stack/horizon/openstack_dashboard/local/enabled
   ln -s /opt/stack/solum-dashboard/_50_solum.py.example _50_solum.py
   service apache2 restart
SCRIPT

$devstack_post_install_1 = <<SCRIPT
    # docker driver hack
    cp /opt/stack/nova-docker/etc/nova/rootwrap.d/docker.filters /etc/nova/rootwrap.d/
SCRIPT

$devstack_post_install_2 = <<SCRIPT
    ovs-vsctl add-port br-ex eth2
SCRIPT

if ARGV.include? '--provider=openstack'
  require 'vagrant-openstack-provider'
  PROVIDER = 'openstack'
else
  PROVIDER = (ENV['VAGRANT_DEFAULT_PROVIDER'] || :virtualbox).to_sym
end

Vagrant.configure("2") do |config|
    config.vm.network :forwarded_port, guest: 80, host: 8080 # Horizon
    config.vm.network :forwarded_port, guest: 8774, host: 8774 # Compute API

    do_provision(config)
    config_virtualbox(config)
    config_openstack(config)
end

def do_provision(config)
    config.vm.provision :ansible do |ansible|
      ansible.host_key_checking = false
      ansible.playbook = "devstack.yaml"
      ansible.verbose = "v"
      ansible.extra_vars = {
        vagrant_provider: PROVIDER
      }
    end
    config.vm.provision :shell, :inline => $solum_prepare
    config.vm.provision :shell, :inline => $murano_prepare
    config.vm.provision :shell, :inline => $nova_docker_prepare
    config.vm.provision :shell, :privileged => false, :inline => $stack_sh_run
    config.vm.provision :shell, :inline => $solum_dashboard_install
    config.vm.provision :shell, :inline => $devstack_post_install_1
end

def config_virtualbox(config)
    config.vm.provider :virtualbox do |vb, override|
      override.vm.box = "saucy64"
      override.vm.box_url = "http://cloud-images.ubuntu.com/vagrant/saucy/current/saucy-server-cloudimg-amd64-vagrant-disk1.box"
      # eth1, this will be the endpoint
      override.vm.network :private_network, ip: "192.168.27.100"
      # eth2, this will be the OpenStack "public" network, use DevStack default
      override.vm.network :private_network, ip: "172.24.4.225", :netmask => "255.255.255.224", :auto_config => false
      override.vm.provision :shell, :inline => $devstack_post_install_2
      vb.customize ["modifyvm", :id, "--memory", 8192]
      vb.customize ["modifyvm", :id, "--cpus", "2"]
      vb.customize ["modifyvm", :id, "--ioapic", "on"]
      # eth2 must be in promiscuous mode for floating IPs to be accessible
      vb.customize ["modifyvm", :id, "--nicpromisc3", "allow-all"]
    end
end

def config_openstack(config)
    config.vm.provider :openstack do |os, override|
      override.vm.box = "dummy-openstack"
      override.vm.box_url = "https://github.com/ggiamarchi/vagrant-openstack/raw/master/source/dummy.box"
      override.ssh.private_key_path = '~/.ssh/id_rsa'
      os.server_name = "vagrant-devstack"
      os.username = ENV['OS_USERNAME']
      os.floating_ip = "185.39.216.118"
      os.api_key = ENV['OS_PASSWORD']
      #os.network = "private"
      os.flavor = /Linux-XL.2plus-4vCpu-32G/
      os.image = /ubuntu-12.04_x86_64_LVM/
      os.openstack_auth_url = ENV['OS_AUTH_URL']
      os.openstack_compute_url = ENV['OS_COMPUTE_URL']
      os.availability_zone = "nova"
      os.tenant_name = ENV['OS_TENANT_NAME']
      os.keypair_name = "julien-mac"
      os.ssh_username = "stack"
    end
end
