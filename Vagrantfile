# -*- mode: ruby -*-
# vi: set ft=ruby :

# Basic Config
Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-16.04"
  config.vm.hostname = "kube1"
  config.vm.network "private_network", ip: "10.0.0.4"  
  #config.vm.network "private_network", type: "dhcp"  

# VirtualBox specific
  config.vm.provider "virtualbox" do |vb|
	vb.name = "vagrant-kube1"
  vb.memory = "2048"
	vb.cpus = 2
  end
  
############################################################
  # Provisioning VM
  config.vm.provision "shell", inline:
  <<-SHELL
    echo "Installing base packages"
    apt-get update
    apt-get install -y \
      git \
      zsh \
      apt-transport-https \
      ca-certificates \
      curl \
      gnupg2 \
      software-properties-common
    echo "DOCKER - Adding Docker’s official GPG key"
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    echo "DOCKER - Verify fingerprint 9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88"
    apt-key fingerprint 0EBFCD88
    echo "DOCKER - Setup stable repo"
    add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
     $(lsb_release -cs) \
     stable"
    apt-get update
    echo "DOCKER - Install Docker CE version 18.06.1 (Kubernetes compatible)"
    apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu
    echo "Adding vagrant user to the docker group"
    usermod -aG docker vagrant
  SHELL

  config.vm.provision "shell", privileged: false, inline:
<<-SHELL
  echo "Cloning Oh My Zsh from the git repo" 
  git clone https://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
  echo "Copying the default .zshrc config file"
  cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
  echo "Changing the Oh_My_zsh theme"
  sed -i 's/robbyrussell/agnoster/g' ~/.zshrc
SHELL

config.vm.provision "shell", inline:
  <<-SHELL
    echo "Changing the vagrant user's shell to use zsh"
    chsh -s /bin/zsh vagrant
  SHELL
############################################################
end