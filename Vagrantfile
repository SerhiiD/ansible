# -*- mode: ruby -*-
# vi: set ft=ruby :
ENV["LC_ALL"] = "en_US.UTF-8"

require "csv"
require "yaml"
require "fileutils"


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

    controller.vm.synced_folder ".", "/vagrant"

    controller.vm.provision "trigger" do |trigger|
      trigger.fire do
        addAllGuestsIPtoAllGuestsHostsFiles
      end        
    end

    controller.vm.provision "ansible_local" do |ansible|
      # ansible.verbose = true
      ansible.playbook = "main.yml"
      ansible.limit = "controller"
      ansible.extra_vars = {
        is_vagrant: true,
      }
    end
  
    # controller.vm.provision "ansible_local" do |ansible|
    #   ansible.groups = {
    #     "nodes" => ["node01"]
    #   }
    #   ansible.playbook = "main.yml"
    #   ansible.inventory_path = "hosts.yml"
    #   ansible.verbose = true
    #   ansible.limit = "all"
    # end
    
  end

end # Vagrant.configure

def addAllGuestsIPtoAllGuestsHostsFiles
  hostsFooter = "# added by vagrant:\n"
  # in the code below vagrant commands are recursively executed. this check helps to avoid infinite loop.
  # if ARGV.include? "up" or ARGV.include? "provision" or ARGV.include? "--provision" then
  if not ARGV.include? "status" and not ARGV.include? "ssh" and not ARGV.include? "ssh-config" then
    # get host list
    cmd = `vagrant status --machine-readable`
    hosts = []
    CSV.parse(cmd) do |row|
      if row.include? ("state") then
        if row[row.index("state")+1] == "running" then
          hosts << row[1]
        end
      end
    end
    hosts = hosts.uniq

    hosts.each do |host|
      cmd = "vagrant ssh  #{host} -c 'hostname -s' -- -q"
      hostName = (`#{cmd}`).gsub!(/[^0-9A-Za-z\.\-]/, '')

      # get IP address from the hosts
      # probably should be suppressed by https://github.com/devopsgroup-io/vagrant-hostmanager
      cmd = "vagrant ssh  #{host} -c \"hostname -I | cut -d\' \' -f2\" -- -q"
      hostIP = (`#{cmd}`).gsub!(/[^0-9\.]/, '')

      hostsFooter += "#{hostIP}\t#{hostName}\n"
    end

    hosts.each do |host|
      puts "adding information to #{host}:/etc/hosts"
      cmd = "vagrant ssh #{host} -c \"sudo sed -i '/^# added by vagrant:$/,$d\' /etc/hosts && echo '#{hostsFooter}' | sudo tee -a /etc/hosts \" -- -q"
      system(cmd)
    end
  
  end    
end
