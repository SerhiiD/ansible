# -*- mode: ruby -*-
# vi: set ft=ruby :
ENV["LC_ALL"] = "en_US.UTF-8"

require "csv"
require "yaml"
# require "fileutils"


# checking if vagrant-triggers plugin is installed
if !Vagrant.has_plugin?("vagrant-triggers")
  # install vagrant-triggers plugin and rerun vagrant with current arguments
  system("vagrant plugin install vagrant-triggers")
  system("vagrant #{ARGV.join(" ")}")
  abort
end

Vagrant.configure("2") do |config|
  config.vm.network "private_network", type: "dhcp"

  config.vm.define "node01" do |node01|
    node01.vm.provider "virtualbox" do |virtualbox|
      virtualbox.name = "node01"
    end
    node01.vm.box = "centos/7"
    node01.vm.hostname = "node01"
  end

  config.vm.define "controller" do |controller|
    controller.vm.provider "virtualbox" do |virtualbox|
      virtualbox.name = "controller"
    end
    controller.vm.box = "centos/7"
    controller.vm.hostname = "controller"

    controller.vm.synced_folder ".", "/vagrant" #, rsync__exclude: ""
    
    # controller.vm.provision "file", source: ".vagrant/machines/node01/virtualbox/private_key", destination: "/home/vagrant/.ssh/node01_private_key"
    # controller.vm.provision "file", source: ".vagrant/machines/controller/virtualbox/private_key", destination: "/home/vagrant/.ssh/node01_private_key"
    # controller.vm.provision "shell", inline: "sudo chmod 600 /home/vagrant/.ssh/*_private_key"
    # controller.vm.provision "shell", inline: "sudo chmod 600 /home/vagrant/.ssh/config"
    
    controller.vm.provision "trigger" do |trigger|
      trigger.fire do
        addVMsIPtoVMsHostsFiles
        # puts getPrivateKeyFiles
        getPrivateKeys
      end        
    end

    # controller.vm.provision "ansible_local" do |ansible|
    #   # ansible.verbose = true
    #   ansible.playbook = "main.yml"
    #   ansible.limit = "controller"
    #   ansible.extra_vars = {
    #     is_vagrant: true,                                                                              
    #   }
    # end
  
    # controller.vm.provision "ansible_local" do |ansible|
    #   ansible.groups = {
    #     "nodes" => ["node01"]
    #   }
    #   ansible.playbook = "main.yml"
    #   # ansible.inventory_path = "hosts.yml"
    #   # ansible.verbose = true
    #   ansible.limit = "all"
    #   ansible.extra_vars = {
    #     is_vagrant: true,
    #   }
    # end
    
  end


end # Vagrant.configure

def getRunningVMs
  hosts = []
  if not ARGV.include? "status" then
    # get host list
    output = `vagrant status --machine-readable`
    # hosts = []
    CSV.parse(output) do |row|
      if row.include? ("state") then
        if row[row.index("state")+1] == "running" then
          hosts << row[1]
        end
      end
    end
    hosts = hosts.uniq
  end

  return hosts
end

def addVMsIPtoVMsHostsFiles
  hostsFooter = "# added by vagrant:\n"
  # in the code below vagrant commands are recursively executed. this check helps to avoid infinite loop.
  if not ARGV.include? "ssh" then
      hosts = getRunningVMs

      hosts.each do |host|
      cmd = "vagrant ssh  #{host} -c 'hostname -s' -- -q"
      hostName = (`#{cmd}`).gsub!(/[^0-9A-Za-z\.\-]/, '')

      # get IP address from the hosts
      # probably should be suppressed by https://github.com/devopsgroup-io/vagrant-hostmanager, but this plugin doesn't work well
      cmd = "vagrant ssh  #{host} -c \"hostname -I | cut -d\' \' -f2\" -- -q"
      hostIP = (`#{cmd}`).gsub!(/[^0-9\.]/, '')

      hostsFooter += "#{hostIP}\t#{hostName}\n"
    end

    hosts.each do |host|
      puts "adding information to #{host}:/etc/hosts"
      cmd = "vagrant ssh #{host} -c \"sudo sed -i -e '/^# added by vagrant:$/,$ d\' /etc/hosts && echo '#{hostsFooter}' | sudo tee -a /etc/hosts \" -- -q"
      # puts cmd
      system(cmd)
    end
  
  end    
end

def getPrivateKeyFiles
  # in the code below vagrant commands are recursively executed. this check helps to avoid infinite loop.
  if not ARGV.include? "ssh-config" then
    hosts = getRunningVMs
    keyFiles = {}
    hosts.each do |host|
      cmd = "vagrant ssh-config #{host}"
      ((`#{cmd}`).to_s.split).each_slice(2) do |pair|
        if pair[0] == "IdentityFile" then
          keyFiles[host] = pair[1]
          # FileUtils.cp(pair[1], "#{hostName}_private_key")
        end
      end
    end
    return keyFiles
  end    
end

def getPrivateKeys
  # in the code below vagrant commands are recursively executed. this check helps to avoid infinite loop.
    keyFiles = getPrivateKeyFiles
    keys = {}
    keyFiles.each do |host, file|
      puts host + "\t" + file
    end
    return keys
end
