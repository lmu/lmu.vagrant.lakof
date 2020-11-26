# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|

  # Use SSH-Key of host machine
  config.ssh.forward_agent = true

  # Set Base Box and URL
  config.vm.box = "generic/debian10"

  # Disable SharedFolder by default.
  config.vm.synced_folder ".", "/vagrant", disabled: true

  # General Provider Settings
  config.vm.provider "virtualbox" do |vb|
    # Disable GUI --> No GUI Window will pop up on start, could be opened on virtual Box app via double click on machine.
    vb.gui = false

    # special VirtualBox Customizations:
    vb.customize ["modifyvm", :id,
                  "--nicpromisc1", "allow-all",
                  "--nicpromisc2", "allow-all",
                  "--cpuexecutioncap", "50",
                  # Make a Structure within Virtualbox to group Machines
                  "--groups", "/Vagrant/LMU/Jenkins"
                 ]
  end

  config.vm.define "lakof-server", primary: true, autostart: true do |node|
    node.vm.hostname = "LAKOF-Server"

    node.vm.provider "virtualbox" do |vb|
      vb.name = "lakof-server"
      vb.memory = 4096
      vb.cpus = 1
    end
    node.vm.network :public_network, ip: "192.168.5.100", netmask: "255.255.255.0", bridge: [
      "en9: Thunderbolt Ethernet Slot  2",
      "en0: WLAN (Wireless)",
      "bridge0",
    ]
  end

  config.vm.provision "bootstrap", type: "shell" do |s|
    s.inline = "echo Bootstrap Machine"
  end
  config.vm.provision "ssh-key", type: "shell", privileged: false do |s|
    ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
    s.inline = <<-SHELL
      echo #{ssh_pub_key} >> ~/.ssh/authorized_keys
    SHELL
  end
  config.vm.provision "update", type: "shell", privileged: true do |s|
    s.inline = <<-SHELL
      apt update
      apt full-upgrade -y
    SHELL
  end
  config.vm.provision "vagrant-preseed", type: "ansible" do |ansible|
    ansible.compatibility_mode = "2.0"
    ansible.playbook = "lmu.ansible.playbooks/vagrant-preseed.yml"
    #ansible.verbose = "vvv"
    ansible.extra_vars = {
      ansible_python_interpreter: "python3"
    }
  end
  config.vm.provision "application", type: "ansible" do |ansible| # run: "never"
    ansible.compatibility_mode = "2.0"
    ansible.playbook = "lmu.ansible.playbooks/lakof.yml"
    #ansible.playbook = "lmu.ansible.playbooks/jenkins.yml"
    ansible.groups = {
      "lakof" => ["lakof-server",],
    }
    #ansible.verbose = "vvvv"
    #ansible.verbose = "vvvv"
    #ansible.verbose = "vv"
    ansible.verbose = "v"
    #ansible.verbose = ""
    #ansible.start_at_task = ""
    #ansible.stop_at_task = ""
    ansible.limit = "all"
    #ansible.tags = ["setup", "configuration", "update"]
    #ansible.skip_tags = ["update"]
    ansible.extra_vars = {
    #  ansible_connection: 'ssh',
    #  ansible_port: '2222',
    #  ansible_ssh_args: '-o ForwardAgent=yes',
    #  ansible_ssh_private_key_file: ['~/.ssh/id_rsa'],
      ansible_python_interpreter: "python3"
    }
  end
end
