# encoding: utf-8
# -*- mode: ruby -*-
# vi: set ft=ruby :
require "json"
MACHINES = JSON.parse(File.read(File.join(File.dirname(__FILE__),
    "vagrant-machines.json"
)))
VAGRANTFILE_API_VERSION = "2"
Vagrant.require_version ">= 1.6.0"
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    # starting each instance of the JSON file(vagrant-machines.json) ---------------------------------
    (MACHINES).each_with_index do |cluster, index|
        # Verify that the cluster is active.
        if cluster["active"] == true then
            iterate = 0 # define "auto increment" of active machines.
            # Browse active machine list.
            cluster["machines"].each do |machine|
                # Verify that the machine is active and initiating virtual cluster configuration -----
                if machine["active"] == true then
                    iterate = (iterate + 1) # auto increment of active machines.
                    # define host name and domain ----------------------------------------------------
                    _hostname = machine["hostname"]
                    _domain   = machine["domain"]
                    # define (Network) of the machine ------------------------------------------------
                    _ip_address_static = machine["ip_address_static"]
                    _interface = machine["interface"]
                    # starting the configuration of the virtual machine ------------------------------
                    config.vm.define "#{_hostname}.#{_domain}" do |subconfig|
                        # define host name -----------------------------------------------------------
                        subconfig.vm.hostname = _hostname
                        # define the Vagrant Box -----------------------------------------------------
                        subconfig.vm.box = machine["box"]
                        # define access --------------------------------------------------------------
                        # This sets the username that Vagrant will SSH.
                        subconfig.ssh.username = "vagrant"
                        # Always use Vagrants insecure key.
                        subconfig.ssh.insert_key = false
                        # Forward ssh agent to easily ssh into the different machines.
                        subconfig.ssh.forward_agent = true
                        # provider virtualbox --------------------------------------------------------
                        subconfig.vm.provider "virtualbox" do |virtualbox|
                            virtualbox.name = "vm-" + _hostname
                            virtualbox.cpus = machine["cpus"]
                            virtualbox.memory = machine["memory"]
                            virtualbox.gui = false
                            virtualbox.functional_vboxsf = false
                            virtualbox.check_guest_additions = false
                            virtualbox.customize ["modifyvm", :id, "--cpuexecutioncap", "100"]
                        end
                        # provider - network ---------------------------------------------------------
                        if machine["public_network"] == true then
                            # provider - public network ----------------------------------------------
                            subconfig.vm.network "public_network", ip: "#{_ip_address_static}"
                        else
                            # provider - private network ---------------------------------------------
                            subconfig.vm.network "private_network", ip: "#{_ip_address_static}"
                        end
                        # provider base system -------------------------------------------------------
                        subconfig.vm.provision "shell", inline: "hostnamectl set-hostname '#{_hostname}'"
                        subconfig.vm.provision "shell", inline: "sleep 1s && hostnamectl status"
                        subconfig.vm.provision "shell", inline: "echo '#{_ip_address_static} #{_hostname} #{_hostname}.#{_domain}' >> /etc/hosts"
                    end
                end
            end
        end
    end
    # plugin conflict
    if Vagrant.has_plugin?("vagrant-vbguest") then
        config.vbguest.auto_update = false
    end
end