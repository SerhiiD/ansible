# -*- mode: ruby -*-
# vi: set ft=ruby :

require "csv"
require "yaml"
require "fileutils"

ENV["LC_ALL"] = "en_US.UTF-8"

# check is vagrant-triggers plugin installed
if (!Vagrant.has_plugin?("vagrant-triggers"))
  # install vagrant-triggers plugin and rerun vagrant with current arguments
  system("vagrant plugin install vagrant-triggers")
  system("vagrant #{ARGV.join(" ")}")
  abort
else
  Vagrant.configure("2") do |config|
    config.vm.network "private_network", type: "dhcp"

    config.vm.define "node01" do |node01|
      node01.vm.provider "virtualbox" do |virtualbox|
        virtualbox.name = "node01"
      end
  
      node01.vm.box = "centos/7"
      node01.vm.hostname = "node01"
      
      node01.vm.synced_folder ".", "/vagrant", id: "vagrant-root",
      owner: "vagrant",
      group: "vagrant",
      mount_options: ["dmode=700,fmode=600"]
    end
  
    config.vm.define "controller" do |controller|
      controller.vm.provider "virtualbox" do |virtualbox|
        virtualbox.name = "controller"
      end
      controller.vm.box = "centos/7"
      controller.vm.hostname = "controller"

      controller.vm.synced_folder ".", "/vagrant", id: "vagrant-root",
      owner: "vagrant",
      group: "vagrant",
      mount_options: ["dmode=700,fmode=600"]
            
      controller.vm.provision "ansible_local" do |ansible|
        ansible.provisioning_path = "/vagrant"
        ansible.playbook = "ping.yml"
        ansible.inventory_path = "hosts.yml"
        #  ansible.verbose        = true
        ansible.install        = true
        ansible.limit          = "all" # or only "nodes" group, etc.
        ansible.extra_vars = {
          is_vagrant: true,
        }
      end
    end

    config.trigger.after [:up, :resume, :provision] do
      if ARGV.include? "up" or ARGV.include? "provision" or ARGV.include? "--provision" then
        vmStatus = `vagrant status --machine-readable`

        hosts = []
        CSV.parse(vmStatus) do |row|
          if row.include? ("state") then
            if row[row.index("state")+1] == "running" then
              # puts "#{row [1]} #{row[row.index("state")]} #{row[row.index("state")+1]}"
              hosts << row[1]
            end
          end
        end

        hosts = hosts.uniq
        # puts hosts
        
        inventory = {"all" => {"hosts" => {}}}
        hosts.each do |host|
          cmd = "vagrant ssh  #{host} -c 'hostname -s' -- -q"
          hostName = (`#{cmd}`).gsub!(/[^0-9A-Za-z\.\-]/, '')

          # probably this is a hack
          cmd = "vagrant ssh  #{host} -c \"hostname -I | cut -d\' \' -f2\" -- -q"
          hostIP = (`#{cmd}`).gsub!(/[^0-9\.]/, '')

          cmd = "vagrant ssh-config #{host}"

          ((`#{cmd}`).to_s.split).each_slice(2) do |pair|
            if pair[0] == "IdentityFile" then
              FileUtils.cp(pair[1], "#{hostName}_private_key")
            end
          end

          inventory["all"]["hosts"][hostName] = {"ansible_host" => hostIP, "ansible_port" => "22", "ansible_ssh_private_key_file" => "#{hostName}_private_key"}
        end

        puts inventory.to_yaml

        File.open("hosts.yml", "w+") do |f|
          f.write inventory.to_yaml
        end
        # system("vagrant rsync")
      end
    end
  end
end

# END {
#   if ARGV.include? "up" or ARGV.include? "provision" or ARGV.include? "--provision" then
#     system("vagrant reload")
#   end
# }