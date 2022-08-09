# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"
VAGRANT_BOX_UBUNTU = "bento/ubuntu-22.04"
VAGRANT_BOX_CENTOS = "bento/centos-7.9"
VAGRANT_BOX_ROCKY = "bento/rockylinux-9"
VAGRANT_BOX_AMAZON = "bento/amazonlinux-2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.boot_timeout = 300
  # Define VMs with static private IP addresses, vcpu, memory and vagrant-box.
  boxes = [
    {
      :name => "server-1",
      :box => VAGRANT_BOX_UBUNTU,
      :ram => 512,
      :vcpu => 1,
      :ip => "192.168.56.10",
      :port_forward => [
        {
          :guest_port => 22,
          :host_port => 2222,
          :host_ip => "",
          :app_id => "ssh"
        }
      ]
    },
    {
      :name => "server-2",
      :box => VAGRANT_BOX_UBUNTU,
      :ram => 512,
      :vcpu => 1,
      :ip => "192.168.56.11",
      :port_forward => [
        {
          :guest_port => 22,
          :host_port => 2223,
          :host_ip => "",
          :app_id => "ssh"
        }
      ]
    },
    {
      :name => "server-3",
      :box => VAGRANT_BOX_UBUNTU,
      :ram => 512,
      :vcpu => 1,
      :ip => "192.168.56.12",
      :port_forward => [
        {
          :guest_port => 22,
          :host_port => 2224,
          :app_name => "ssh"
        },
      ]
    },
    {
      :name => "server-4",
      :box => VAGRANT_BOX_ROCKY,
      :ram => 512,
      :vcpu => 1,
      :ip => "192.168.56.13",
      :port_forward => [
        {
          :guest_port => 22,
          :host_port => 2225,
          :app_name => "ssh"
        }
      ]
    },
  ]

  # Provision each of the VMs.
  boxes.each do |instance|
    config.vm.define instance[:name] do |machine|

      # Setup instance machine
      machine.vm.box = instance[:box]
      machine.vm.hostname = instance[:name]

      # hostmanager so we can have a custom domain for the server
      machine.hostmanager.enabled = true
      machine.hostmanager.manage_host = true

      # DNS
      machine.dns.patterns = ["/^(\w+\.)*" + instance[:name] + "\.local$/"]

      # Private Network
      machine.vm.network :private_network,
        ip: instance[:ip]

      # Forward Network
      if !instance[:port_forward].empty?
        instance[:port_forward].each do |item|
          if item[:host_ip] != ""
            machine.vm.network "forwarded_port",
            guest: item[:guest_port],
            host: item[:host_port],
            host_ip: item[:host_ip],
            id: item[:app_id],
            auto_correct: true
          else
            machine.vm.network "forwarded_port",
            guest: item[:guest_port],
            host: item[:host_port],
            id: item[:app_id],
            auto_correct: true
          end
        end
      end

      # SSH
      machine.ssh.insert_key = false
      machine.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'" # avoids 'stdin: is not a tty' error.
      machine.ssh.private_key_path = ["#{ENV['HOME']}/.ssh/id_rsa","#{ENV['HOME']}/.vagrant.d/insecure_private_key"]

      # Provision
      machine.vm.provision "shell",
        inline: <<-SCRIPT
        printf "%s\n" "#{File.read("#{ENV['HOME']}/.ssh/id_rsa.pub")}" > /home/vagrant/.ssh/authorized_keys
        chown -R vagrant:vagrant /home/vagrant/.ssh
      SCRIPT

      # Virtualbox
      machine.vm.provider :virtualbox do |vb|
        vb.name = instance[:name]
        vb.memory = instance[:ram]
        vb.cpus = instance[:vcpu]
      end

    end
  end

end

# optional
VagrantDNS::Config.logger = Logger.new("dns.log")
