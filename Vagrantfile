# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :lvm => {
    :box_name => "centos/7",
    :box_hostname => "grub"
  }
}

Vagrant.configure("2") do |config|
    MACHINES.each do |boxname, boxconfig|
  
        config.vm.define boxname do |box|
  
            box.vm.box = boxconfig[:box_name]
            box.vm.host_name = boxconfig[:box_hostname]
  
            box.vm.provider :virtualbox do |vb|
                    vb.gui = true
                    vb.customize ["modifyvm", :id, "--memory", "1024"]
                    needsController = false
            end
  
            box.vm.provision "shell", inline: <<-SHELL
              mkdir -p ~root/.ssh
              cp ~vagrant/.ssh/auth* ~root/.ssh
              #yum install -y mdadm smartmontools hdparm gdisk
            SHELL
  
        end
    end
  end

