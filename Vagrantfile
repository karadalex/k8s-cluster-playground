$script = <<-'SCRIPT'
sudo apt-get update
sudo apt-get install -y docker.io
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
SCRIPT


Vagrant.configure("2") do |config|

  config.vm.define "manager1" do |manager1|
    manager1.vm.box = "hashicorp/bionic64"  
    manager1.vm.network "private_network", ip: "192.168.99.100"
    manager1.vm.provision "shell", inline: $script
  end

  config.vm.define "worker1" do |worker1|
    worker1.vm.box = "hashicorp/bionic64"
    worker1.vm.network "private_network", ip: "192.168.99.101"
    worker1.vm.provision "shell", inline: $script
  end

  config.vm.define "worker2" do |worker2|
    worker2.vm.box = "hashicorp/bionic64"
    worker2.vm.network "private_network", ip: "192.168.99.102"
    worker2.vm.provision "shell", inline: $script
  end

end
