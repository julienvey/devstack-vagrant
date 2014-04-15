# -*- mode: ruby -*-
# vi: set ft=ruby :

$solum_script = <<SCRIPT
    mkdir /opt/stack/
    mkdir /opt/stack/solum
    git clone git://github.com/stackforge/solum.git /opt/stack/solum
    cd /opt/stack/solum/contrib/devstack
    cp lib/solum /home/vagrant/devstack/lib
    cp extras.d/70-solum.sh /home/vagrant/devstack/extras.d
SCRIPT



Vagrant.configure("2") do |config|

    config.vm.box = "saucy64"
    config.vm.box_url = "http://cloud-images.ubuntu.com/vagrant/saucy/current/saucy-server-cloudimg-amd64-vagrant-disk1.box"
    config.vm.network :forwarded_port, guest: 80, host: 8080 # Horizon
    config.vm.network :forwarded_port, guest: 8774, host: 8774 # Compute API
    # eth1, this will be the endpoint
    config.vm.network :private_network, ip: "192.168.27.100"
    # eth2, this will be the OpenStack "public" network, use DevStack default
    config.vm.network :private_network, ip: "172.24.4.225", :netmask => "255.255.255.224", :auto_config => false
    #config.vm.synced_folder ENV['STACKSOURCE'], "/opt/stack", :nfs => true
    config.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", 8192]
        vb.customize ["modifyvm", :id, "--cpus", "2"]  
        vb.customize ["modifyvm", :id, "--ioapic", "on"] 
       	# eth2 must be in promiscuous mode for floating IPs to be accessible
       	vb.customize ["modifyvm", :id, "--nicpromisc3", "allow-all"]
    end
    config.vm.provision :ansible do |ansible|
        ansible.host_key_checking = false
        ansible.playbook = "devstack.yaml"
        ansible.verbose = "v"
    end
    config.vm.provision :shell, :inline => $solum_script
    
    config.vm.provision :shell, :inline => "cd /home/vagrant/devstack; sudo -u vagrant env HOME=/home/vagrant ./stack.sh"
    config.vm.provision :shell, :inline => "ovs-vsctl add-port br-ex eth2"

end
