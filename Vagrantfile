# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.


# This script to install Kubernetes will get executed after we have provisioned the box 
$script = <<-SCRIPT
# set http proxy
export http_proxy=http://192.168.10.1:1087
export https_proxy=http://192.168.10.1:1087

# Install docker
curl -sSL https://get.daocloud.io/docker | sh

# use Docker as a non-root user, add vagrat to docker group    
sudo usermod -a -G docker vagrant

# docker 配置代理
sudo mkdir /etc/systemd/system/docker.service.d
sudo touch /etc/systemd/system/docker.service.d/http-proxy.conf
sudo echo '
[Service]
Environment="HTTP_PROXY=http://192.168.10.1:1087" "HTTPS_PROXY=http://192.168.10.1:1087"
' > /etc/systemd/system/docker.service.d/http-proxy.conf

# restart docker
sudo systemctl daemon-reload
systemctl restart docker
sudo systemctl show docker --property Environment

# Setup proxy for apt get
sudo touch /etc/apt/apt.conf
sudo echo 'Acquire::http::Proxy "http://192.168.10.1:1087";' > /etc/apt/apt.conf

# Install kubernetes
sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl

# kubelet requires swap off
sudo swapoff -a

# keep swap off after reboot
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

unset http_proxy
unset https_proxy

SCRIPT

Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-16.04"
  config.vm.network "public_network", bridge: "en0: Wi-Fi (AirPort)"

  config.vm.define "master" do |master|
  	master.vm.network "private_network", ip: "192.168.10.10"
    master.vm.hostname = "master"
    master.vm.provision "shell", inline: $script
  end

  config.vm.define "node1" do |node1|
  	node1.vm.network "private_network", ip: "192.168.10.11"
    node1.vm.hostname = "node1"
    node1.vm.provision "shell", inline: $script
  end

  config.vm.define "node2" do |node2|
  	node2.vm.network "private_network", ip: "192.168.10.12"
    node2.vm.hostname = "node2"
    node2.vm.provision "shell", inline: $script
  end
end
