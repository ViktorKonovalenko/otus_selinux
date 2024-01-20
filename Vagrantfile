# -*- mode: ruby -*- 
# vi: set ft=ruby : vsa
Vagrant.configure(2) do |config| 
 config.vm.box = "centos/7" 
# config.vm.box_version = "2004.01" 
 config.vm.provider "virtualbox" do |v| 
 v.memory = 256 
 v.cpus = 1 
 end 
 config.vm.define "selinux" do |selinux| 
 selinux.vm.network "private_network", ip: "192.168.56.10",  virtualbox__intnet: "net1"
 selinux.vm.network "forwarded_port", guest: 4881, host: 4881
 selinux.vm.hostname = "selinux" 
 selinux.vm.provision "shell", inline: <<-SHELL
          #install epel-release
          yum install -y epel-release
          #install nginx
          yum install -y nginx
          #change nginx port
          sed -ie 's/:80/:4881/g' /etc/nginx/nginx.conf
          sed -i 's/listen       80;/listen       4881;/' /etc/nginx/nginx.conf
          #disable SELinux
          #setenforce 0
          #start nginx
          systemctl start nginx
          systemctl status nginx
          #check nginx port
          ss -tlpn | grep 4881
        SHELL
 end 
end 
