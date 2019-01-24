# -*- mode: ruby -*-
# vi: set ft=ruby :

############################################################
### DETAILED INSTRUCTIONS ON THE BOTTOM OF THIS DOCUMENT ###
############################################################

$script = <<-SCRIPT 
echo "INSTALLING BASE PACKAGES"
apt-get update
apt-get install -y \
git \
zsh \
apt-transport-https \
ca-certificates \
curl \
gnupg2 \
software-properties-common
echo "DOCKER - ADDING DOCKER’S OFFICIAL GPG KEY"
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
echo "DOCKER - VERIFY FINGERPRINT 9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88"
apt-key fingerprint 0EBFCD88
echo "DOCKER - SETUP STABLE REPO"
add-apt-repository \
"deb [arch=amd64] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) \
stable"
apt-get update
echo "DOCKER - INSTALL DOCKER CE VERSION 18.06.1 (KUBERNETES COMPATIBLE)"
apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu
apt-mark hold docker-ce
echo "DOCKER - ADDING VAGRANT USER TO THE DOCKER GROUP"
usermod -aG docker vagrant
echo "KUBERNETES - ADDING KUBERNETES GPG KEY"
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
echo "KUBERNETES - REQUIRED CONFIG"
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
echo "KUBERNETES - REQUIRED PACKAGES INSTALLATION AND HOLD MARK TO AVOID UPGRADES"
apt-get update
apt-get install -y \
kubelet \
kubeadm \
kubectl
apt-mark hold kubelet kubeadm kubectl
echo "KUBERNETES - KUBELET REQUIRES SWAP OFF"
swapoff -a
echo "KUBERNETES - KEEP SWAP OFF AFTER REBOOT"
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
SCRIPT

# Basic Config
Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-18.04"
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
 config.vm.provision "shell", inline: $script

 config.vm.provision "shell", privileged: false, inline:
 <<-SHELL
 echo "CLONING OH MY ZSH FROM THE GIT REPO" 
 git clone https://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
 echo "COPYING THE DEFAULT .ZSHRC CONFIG FILE"
 cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
 echo "CHANGING THE OH_MY_ZSH THEME"
 sed -i 's/ZSH_THEME="robbyrussell"/ZSH_THEME="agnoster"/' ~/.zshrc  
 SHELL

config.vm.provision "shell", inline:
 <<-SHELL
 echo "CHANGING THE VAGRANT USER'S SHELL TO USE ZSH"
 chsh -s /bin/zsh vagrant
 SHELL
############################################################
end

# Useful documents:
# https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/

# # Prerequisite (as root)
# apt-get update && apt-get install -y apt-transport-https curl

# NOT NEEDED # Update hosts file
# 	1) vi /etc/hosts
# 	2) 10.0.0.4 kube1

# NOT NEEDED # Install openssh-server
# apt-get install openssh-server

# # Installation Key (as root)
# curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

# # Required config (as root)
# cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
# deb https://apt.kubernetes.io/ kubernetes-xenial main
# EOF

# # Main packages installation (as root)
# 	• apt-get update
# 	• apt-get install -y kubelet kubeadm kubectl
# 	• apt-mark hold kubelet kubeadm kubectl (mark those packages for disabling upgrades via apt)

# # kubelet requires swap off (as root)
# swapoff -a

# # Keep swap off after reboot (as root)
# sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# NOT NEEDED # Update kubeadm config (as root)
# sed -i '0,/ExecStart=/s//Environment="KUBELET_EXTRA_ARGS=--cgroup-driver=cgroupfs"\n&/' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

# **** END OF STEPS REQUIRED FOR KUBERNETES SLAVE NODES ****

# # Initializing Master node using Calico as POD Network add on (as root)
# kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=10.0.0.4

# # To make kubectl work for your non-root use (non-root account)
# mkdir -p $HOME/.kube
# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
# sudo chown $(id -u):$(id -g) $HOME/.kube/config

# # Take note of the kubeadm join output. It'll be required to add nodes into the cluster
# kubeadm join --token <token> <master-ip>:<master-port>…
# 	Tokens in Master node can be retrieved using:
# 		kubeadm token list
# 	In a non-prod env, slave nodes can join using below command to skip the 
# 		kubeadm join --discovery-token-unsafe-skip-ca-verification --token=<token> <master-ip>:<master-port>

# # Installing a pod network add-on (Calico in this case)
# kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml

# kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml

# # Confirm pod network is up and running
# kubectl get pods -o wide --all-namespaces

# # Enable scheduling pods in master node
# kubectl taint nodes --all node-role.kubernetes.io/master-

# # Deploy Kubernetes Dashboard
# https://github.com/kubernetes/dashboard
# kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml

# # Expose dashboard to external clients
# https://github.com/kubernetes/dashboard/wiki/Accessing-Dashboard---1.7.X-and-above
# 	1) Check dashboard before exposing it externally:
# 		a. kubectl -n kube-system get service kubernetes-dashboard
# 	2) kubectl -n kube-system edit service kubernetes-dashboard
# 	3) Change type: ClusterIP to type: NodePort
# 	4) Check what port the dashboad was exposed: 
# 		a. kubectl -n kube-system get service kubernetes-dashboard
# 	5) Access it from another machine like: https://10.0.0.4/<port>

# # Check dashboard locally with curl enabling proxy
# 	1) From a separate terminal window: 
# 		a. kubectl proxy
# 	2) curl 10.0.0.4/<proxy_exposed_port>

# # Dashboard UI
# http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/

# # Generate token to access dashboard
# 	1) kubectl create serviceaccount dashboard -n default
# 	2) kubectl create clusterrolebinding dashboard-admin -n default \
# 	--clusterrole=cluster-admin \
# 	--serviceaccount=default:dashboard
# 	3) kubectl get secret $(kubectl get serviceaccount dashboard -o jsonpath="{.secrets[0].name}") -o jsonpath="{.data.token}" | base64 --decode
# 	4) Copy resulting token and paste it in the kubernetes dashboard prompt to sign in

# # Adding a slave node into the cluster
# 	1) From the note taken before, use:
# 		a. kubeadm join --token <token> <master-ip>:<master-port>…
# 	2) Check with:
# 		a. kubectl get nodes
# 	3) Check the new node in the dashboard