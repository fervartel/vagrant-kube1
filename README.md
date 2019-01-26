# Useful documents:
https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/

## Prerequisite (as root)
- apt-get update && apt-get install -y
- apt-transport-https curl

## Optional -  Update hosts file
1) vi /etc/hosts
2) 10.0.0.4 kube1

## Installation Key (as root)
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

## Required config (as root)
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list  
deb https://apt.kubernetes.io/ kubernetes-xenial main  
EOF

## Main packages installation (as root)
- apt-get update
- apt-get install -y kubelet kubeadm kubectl
- apt-mark hold kubelet kubeadm kubectl (mark those packages for disabling upgrades via apt)

## kubelet requires swap off (as root)
- swapoff -a

## Keep swap off after reboot (as root)
- sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# **** END OF STEPS REQUIRED FOR KUBERNETES SLAVE NODES ****

## Initializing Master node using Calico as POD Network add on (as root)
- kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=10.0.0.4

## To make kubectl work for your non-root use (non-root account)
- mkdir -p $HOME/.kube
- sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
- sudo chown $(id -u):$(id -g) $HOME/.kube/config

## Take note of the kubeadm join output. It'll be required to add nodes into the cluster
- kubeadm join --token "token" "master-ip":"master-port"…

## Tokens in Master node can be retrieved using:
 - kubeadm token list

    In a non-prod env, slave nodes can join using below command to skip the token verification
- kubeadm join --discovery-token-unsafe-skip-ca-verification --token="token" "master-ip":"master-port"

## Installing a pod network add-on (Calico in this case)
- kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
- kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml

## Confirm pod network is up and running
- kubectl get pods -o wide --all-namespaces

## Enable scheduling pods in master node
- kubectl taint nodes --all node-role.kubernetes.io/master-

## Deploy Kubernetes Dashboard
- https://github.com/kubernetes/dashboard
- kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml

## Expose dashboard to external clients
https://github.com/kubernetes/dashboard/wiki/Accessing-Dashboard---1.7.X-and-above
1) Check dashboard before exposing it externally:
 	a. kubectl -n kube-system get service kubernetes-dashboard
2) kubectl -n kube-system edit service kubernetes-dashboard
3) Change type: ClusterIP to type: NodePort
4) Check what port the dashboad was exposed: 
 	a. kubectl -n kube-system get service kubernetes-dashboard
5) Access it from another machine like: https://10.0.0.4/"port"

## Check dashboard locally with curl enabling proxy
1) From a separate terminal window: 
	a. kubectl proxy
2) curl 10.0.0.4/"proxy_exposed_port"

## Dashboard UI
http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/

## Generate token to access dashboard
1) kubectl create serviceaccount dashboard -n default
2) kubectl create clusterrolebinding dashboard-admin -n default \
--clusterrole=cluster-admin \
--serviceaccount=default:dashboard
3) kubectl get secret $(kubectl get serviceaccount dashboard -o jsonpath="{.secrets[0].name}") -o jsonpath="{.data.token}" | base64 --decode
4) Copy resulting token and paste it in the kubernetes dashboard prompt to sign in

## Adding a slave node into the cluster
1) From the note taken before, use:
	a. kubeadm join --token "token" "master-ip":"master-port"…
2) Check with:
	a. kubectl get nodes
3) Check the new node in the dashboard