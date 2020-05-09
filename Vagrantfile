# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  (1..3).each do |i|

    if i == 1 then
      vm_name = "master"
    else
      vm_name = "node#{i-1}"
    end

    config.vm.define vm_name do |s|
      s.vm.box = "centos/7.8"
      s.vm.hostname = vm_name
      
      # network
      private_ip =  "172.42.42.10#{i-1}"
      s.vm.network "private_network", ip: private_ip

      s.vm.provider "virtualbox" do |vb|
        vb.gui = false
        if i == 1 then
          vb.cpus = 2
          vb.memory = 2048
        else
          vb.cpus = 1
          vb.memory = 1024
        end
      end

      s.vm.provision "shell", inline: <<-SHELL
# Set hosts
cat <<EOF >> /etc/hosts
172.42.42.100 master
172.42.42.101 node1
172.42.42.102 node2
EOF

# Enable ssh password authentication
sed -i 's/^PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
systemctl reload sshd

# Stop firewalld
systemctl stop firewalld
systemctl disable firewalld

# Install iptalbes
yum -y install iptables-services
systemctl start iptables
systemctl enable iptables
iptables -F
service iptables save

# Close swap
swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# Close SELinux
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config

# Set sysclt.cof
cat <<EOF >> /etc/sysctl.conf
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
net.ipv6.conf.all.disable_ipv6=1
EOF

sysctl --system


#=================
#====DOCKER ======
#=================

#1.SET UP THE REPOSITORY
sudo yum install -y yum-utils

sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

#2.INSTALL DOCKER ENGINE
sudo yum install -y docker-ce docker-ce-cli containerd.io

#3.START DOCKER
sudo systemctl start docker
sudo systemctl enable docker

#4.update Cgroup
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF

#=====================
#====KUBENETES=====
#=====================
#
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

# Set SELinux in permissive mode (effectively disabling it)
#setenforce 0
#sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

systemctl enable --now kubelet
systemctl start kubelet

SHELL

    end #s end
  end # node end
end
