$common_provisioning = <<-'SCRIPT'
# ----------------------- Container runtime - Install Docker ----------------------------------------------------
sudo apt-get update
sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io

# ----------------------- Configure Docker ----------------------------------------------------
usermod -aG docker vagrant
sudo mkdir /etc/docker
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
sudo systemctl enable docker
sudo systemctl daemon-reload
sudo systemctl restart docker

# ----------------------- Letting iptables see bridged traffic ----------------------------------------------------
sudo modprobe br_netfilter
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system

# ----------------------- Installing kubeadm, kubelet and kubectl ----------------------------------------------------
sudo apt-get update
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl restart kubelet

# Disable swap
swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
SCRIPT


Vagrant.configure("2") do |config|
  config.vm.provider :virtualbox do |vm|
    vm.memory = 2048
    vm.cpus = 2
  end

  config.vm.provision :shell, privileged: true, inline: $common_provisioning

  config.vm.define "master" do |master|
    master.vm.box = "hashicorp/bionic64"
    master.vm.hostname = "master"
    master.vm.network "private_network", ip: "10.0.0.10"

    config.vm.provision "shell", inline: <<-SHELL      
      # Start cluster
      sudo kubeadm init --apiserver-advertise-address=10.0.0.10 --pod-network-cidr=10.244.0.0/16

      # configure kubectl
      sudo --user=vagrant mkdir -p /home/vagrant/.kube
      cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config
      chown $(id -u vagrant):$(id -g vagrant) /home/vagrant/.kube/config

      echo 'Environment="KUBELET_EXTRA_ARGS=--node-ip=10.0.0.10"' | sudo tee -a /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
      sudo systemctl daemon-reload
      sudo systemctl restart kubelet

      # install Calico pod network addon
      export KUBECONFIG=/etc/kubernetes/admin.conf
      curl https://docs.projectcalico.org/manifests/calico.yaml -O
      kubectl apply -f calico.yaml

      # Get join commands for worker nodes
      KUBE_JOIN_FILE=/vagrant/kube_join.sh
      rm -rf $KUBE_JOIN_FILE
      kubeadm token create --print-join-command >> ${KUBE_JOIN_FILE}
      chmod +x $KUBE_JOIN_FILE

    SHELL
  end

  config.vm.define "worker1" do |worker1|
    worker1.vm.box = "hashicorp/bionic64"
    worker1.vm.hostname = "worker1"
    worker1.vm.network "private_network", ip: "10.0.0.11"
    worker1.vm.provision "shell", inline: <<-SHELL
      sudo /vagrant/kube_join.sh
      echo 'Environment="KUBELET_EXTRA_ARGS=--node-ip=10.0.0.11"' | sudo tee -a /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
      sudo systemctl daemon-reload
      sudo systemctl restart kubelet
    SHELL
  end

  config.vm.define "worker2" do |worker2|
    worker2.vm.box = "hashicorp/bionic64"
    worker2.vm.hostname = "worker2"
    worker2.vm.network "private_network", ip: "10.0.0.12"
    worker2.vm.provision "shell", inline: <<-SHELL
      sudo /vagrant/kube_join.sh
      echo 'Environment="KUBELET_EXTRA_ARGS=--node-ip=10.0.0.12"' | sudo tee -a /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
      sudo systemctl daemon-reload
      sudo systemctl restart kubelet
    SHELL
  end

end
