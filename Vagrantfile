Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/xenial64"
  config.vm.define "minikube" do |kube|
  config.vm.provider "virtualbox" do |vb|
      vb.memory = "4096"
      vb.cpus = "4"
  end

  config.vm.network "forwarded_port",
      guest: 30000,
      host:  30000,
      auto_correct: true

  config.vm.network "forwarded_port",
      guest: 8001,
      host:  8001,
      auto_correct: true

  config.vm.network "forwarded_port",
      guest: 80,
      host:  80,
      auto_correct: true

  config.vm.network "forwarded_port",
      guest: 443,
      host:  443,
      auto_correct: true

    config.vm.provision "kubectl", type: "shell",  inline: <<-SCRIPT
echo "Installing kubectl"
apt-get -qq update && apt-get install -qqy apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubectl.list
apt-get update -qq
apt-get install -qqy kubectl
SCRIPT


    config.vm.provision "docker", type: "shell", inline: <<-SCRIPT
sudo apt-get remove docker docker-engine docker.io
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"

sudo apt-get update -qq
sudo apt-get install -yqq docker-engine
usermod -aG docker vagrant
sudo docker run hello-world
SCRIPT

    config.vm.provision "minikube", type: "shell", inline: <<-SCRIPT
echo "Downloading minikube"
curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.26.1/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
SCRIPT

    config.vm.provision "k8s", type: "shell", inline: <<-SCRIPT
echo "Setting up and starting K8S"
minikube config set WantReportErrorPrompt false
minikube start --vm-driver none

minikube dashboard --url
kubectl cluster-info
SCRIPT

	config.vm.provision "shell", inline: <<-SHELL
#!/bin/bash
echo  "Configuring vagrant user dirctory"
printf "source <(kubectl completion bash)\n" >> /home/vagrant/.bashrc
# Permissions
sudo mkdir /home/vagrant/.kconfig
sudo cp /etc/kubernetes/admin.conf /home/vagrant/.kconfig
sudo chown vagrant:vagrant /home/vagrant/.kconfig/admin.conf
printf "export KUBECONFIG=/home/vagrant/.kconfig/admin.conf" >> /home/vagrant/.bashrc 
SHELL
  end

end
