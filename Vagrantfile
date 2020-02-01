# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

NAME="centos7"
OS_NAME="centos/7"
URL="https://cloud.centos.org/centos/7/vagrant/x86_64/images/CentOS-7-x86_64-Vagrant-1803_01.VirtualBox.box"
NODE_COUNT = 1

boxes = [
    {
        :name => "docker-node1",
        :eth1 => "192.168.205.10",
        :mem => "1024",
        :cpu => "1"
    },
    {
        :name => "docker-node2",
        :eth1 => "192.168.205.11",
        :mem => "1024",
        :cpu => "1"
    }
]

unless Vagrant.has_plugin?("vagrant-proxyconf")
  puts 'Installing vagrant-proxyconf Plugin...'
  system('vagrant plugin install vagrant-proxyconf')
end

offset_sec = Time.now.gmt_offset
if (offset_sec % (60 * 60)) == 0
  offset_hr = ((offset_sec / 60) / 60)
  timezone_suffix = offset_hr >= 0 ? "-#{offset_hr.to_s}" : "+#{(-offset_hr).to_s}"
  SYSTEM_TIMEZONE = 'Etc/GMT' + timezone_suffix
else
  SYSTEM_TIMEZONE = 'UTC'
end

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  #if Vagrant.has_plugin?("vagrant-vbguest")
    #config.vbguest.auto_update = false
  #end
  config.vm.box = OS_NAME
  config.vm.box_url = URL

  config.vm.box_check_update = false
  config.vbguest.no_remote = true

  if Vagrant.has_plugin?("vagrant-proxyconf")
    puts "getting Proxy Configuration from Host..."
    if ENV["http_proxy"]
      puts "http_proxy: " + ENV["http_proxy"]
      config.proxy.http     = ENV["http_proxy"]
    end
    if ENV["https_proxy"]
      puts "https_proxy: " + ENV["https_proxy"]
      config.proxy.https    = ENV["https_proxy"]
    end
    if ENV["no_proxy"]
      config.proxy.no_proxy = ENV["no_proxy"]
    end
  end

  config.ssh.forward_x11 = true

  config.vm.define "ansible-server" do |cfg|
    cfg.vm.box = OS_NAME
 	cfg.vm.provider "virtualbox" do |vb|
	  vb.name = "Ansible-Server"
  end
	cfg.vm.host_name = "ansible-server"
	cfg.vm.network "public_network", ip: "192.168.1.25"
  cfg.vm.network "forwarded_port", guest: 22, host: 60025, auto_correct: true, id: "ssh"
  #cfg.vm.synced_folder "provisioning","/provisioning"
	cfg.vm.synced_folder "../data", "/vagrant", disabled: true
	cfg.vm.provision "shell", inline: "yum install epel-release -y"
	cfg.vm.provision "shell", inline: "yum install ansible -y"
	cfg.vm.provision "file", source: "./provisioning/ansible_env_ready.yml",
	  destination: "ansible_env_ready.yml"
	cfg.vm.provision "shell", inline: "ansible-playbook ansible_env_ready.yml"
  end

  #boxes.each do |opts|
  NODE_COUNT.times do |i|
    vm_hostname = "vmhost00#{i}"
    config.vm.define vm_hostname do |node|

      node.vm.provider "virtualbox" do |vb|
        vb.name = "#{vm_hostname}"
      end
      node.vm.provision "shell", path: "./provisioning/vmhosts.sh", args: ""
      node.vm.network "forwarded_port", guest: 80, host: "890#{i}"
      node.vm.network "forwarded_port", guest: 1521, host: 1521
      node.vm.network "forwarded_port", guest: 5500, host: 5500
      node.vm.network "private_network", :dev => "eth0", :ip => "192.200.10.1#{i}"
      node.vm.hostname = "#{vm_hostname}"
      node.vm.provider :virtualbox do |vb|
          vb.customize ["modifyvm", :id, "--memory", "4096"]
          vb.customize ["modifyvm", :id, "--cpus", "2"]
      end
    end
  end
  #end

  config.vm.define "docker_server" do |docker_server|
    docker_server.vm.provider "virtualbox" do |vb|
      vb.name = "docker_server"
    end
    docker_server.vm.network "private_network", :dev => "eth0", :ip => "192.200.10.100"
    docker_server.vm.network "forwarded_port", guest: 8080, host: 8080
  end
  config.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "2048"]
      vb.customize ["modifyvm", :id, "--cpus", "2"]
  end

  config.vm.provision "shell", inline: <<-SHELL
    sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
    /bin/systemctl restart sshd
  SHELL

  config.vm.provision "shell", path: "./provisioning/default_setting.sh", args: ""
  config.vm.provision "shell", path: "./provisioning/docker_install.sh"
  config.vm.synced_folder "scripts/",   "/vagrant_scripts"
  config.vm.synced_folder "utilities/",   "/vagrant_utilities"
  config.vm.synced_folder "provisioning/",   "/vagrant_provisioning"

end