IMAGE_NAME = "centos/7"
NODES = 3

$node_setup_script = <<-SCRIPT

echo "RUNNING NODE SETUP SCRIPT"

# adding the regular user (vagrant) to the docker user group to run
# docker commands as vagrant user without sudo privileges
usermod -aG docker vagrant

# Install kubernetes
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

yum install -y kubelet kubeadm kubectl

# kubelet requires swap off
swapoff -a
# keep swap off after reboot
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

SCRIPT

$master_setup_script = <<-SCRIPT

echo "RUNNING MASTER SETUP SCRIPT"

# ip of this box
IP_ADDR=`ifconfig enp0s8 | grep Mask | awk '{print $2}'| cut -f2 -d:`

# install k8s master
HOST_NAME=$(hostname -s)
kubeadm init --apiserver-advertise-address=$IP_ADDR --apiserver-cert-extra-sans=$IP_ADDR  --node-name $HOST_NAME --pod-network-cidr=192.168.50.0/24

SCRIPT

Vagrant.configure("2") do |config|
    # TODO: what does this command do??
    config.ssh.insert_key = false

    config.vm.provider "virtualbox" do |v|
        v.memory = 2048
        v.cpus = 2
    end

    config.vm.provision :docker

    config.vm.define "k8s-master", primary: true do |master|
        master.vm.box = IMAGE_NAME
        master.vm.network "private_network", ip: "192.168.50.10"
        master.vm.hostname = "k8s-master"
        master.vm.provision "shell", inline: $node_setup_script
        master.vm.provision "shell", inline: $master_setup_script
#        master.vm.provision "ansible" do |ansible|
#            ansible.playbook = "kubernetes-setup/master-playbook.yml"
#            ansible.extra_vars = {
#                node_ip: "192.168.50.10",
#            }
#        end
    end

    (1..NODES).each do |i|
        config.vm.define "node-#{i}" do |node|
            node.vm.box = IMAGE_NAME
            node.vm.network "private_network", ip: "192.168.50.#{i + 10}"
            node.vm.hostname = "node-#{i}"
            # Install Docker
#            node.vm.provision :docker
            node.vm.provision "shell", inline: $node_setup_script
#            node.vm.provision "ansible" do |ansible|
#                # Disable default limit to connect to all the machines
#                ansible.limit = "all"
#                ansible.playbook = "kubernetes-setup/node-playbook.yml"
#                ansible.extra_vars = {
#                    node_ip: "192.168.50.#{i + 10}",
#                }
#            end
        end
    end
end
