# -*- mode: ruby -*-
# vi: set ft=ruby :

require "csv"

ENV["LC_ALL"] = "en_US.UTF-8"

Vagrant.configure("2") do |config|
  config.vm.define "node01" do |node01|
    node01.vm.provider "virtualbox" do |virtualbox|
      virtualbox.name = "node01"
    end

    node01.vm.box = "centos/7"
    node01.vm.hostname = "node01"
    node01.vm.synced_folder ".", "/vagrant"
  end

  config.vm.define "ansible01" do |ansible01|
    ansible01.vm.provider "virtualbox" do |virtualbox|
      virtualbox.name = "ansible01"
    end
    ansible01.vm.box = "centos/7"
    ansible01.vm.hostname = "ansible01"
    ansible01.vm.synced_folder ".", "/vagrant"

#    ansible01.vm.provision "ansible_local" do |ansible|
#      ansible.playbook = "ansible/main.yml"
##     ansible.verbose        = true
#      ansible.install        = true
##     ansible.limit          = "all" # or only "nodes" group, etc.
#      ansible.extra_vars = {
#        is_vagrant: true,
#      }
#    end
  end
end

if ARGV.include? "up" or ARGV.include? "provision" or ARGV.include? "--provision" then

  vmStatus = `vagrant status --machine-readable`

  hosts = []
  CSV.parse(vmStatus, :quote_char => "|") do |row|
    hosts << row[1]
  end

  hosts = hosts.uniq
  hosts.delete(nil)

  hosts.each do |host|
    cmd = "vagrant ssh-config #{host}"
    hostDetails = `#{cmd}`
    puts hostDetails
    arrHostDetails = hostDetails.split
    puts arrHostDetails
  end
end
