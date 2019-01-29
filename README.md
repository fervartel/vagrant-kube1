# Kubernetes cluster setup:
https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/

## Prerequisite (as root)
``` bash
apt-get update && apt-get install -y
apt-transport-https curl
```

## Optional -  Update hosts file
``` bash
vi /etc/hosts
10.0.0.4 kube1
```

## Installation Key (as root)
``` bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
```

## Required config (as root)
``` bash
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list  
deb https://apt.kubernetes.io/ kubernetes-xenial main  
EOF
```

## Main packages installation (as root)
```bash
apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
```
> "apt-mark hold" marks those packages for disabling upgrades via apt

## kubelet requires swap off (as root)
``` bash
swapoff -a
```

## Keep swap off after reboot (as root)
``` bash
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

---
<font size ="4"> **End of common steps for both MASTER and SLAVE nodes**</font>
___  

# MASTER Node setup

## Initializing Master node using Calico as POD Network add on (as root)
``` bash
kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=10.0.0.4
```

## To make kubectl work for your non-root use (non-root account)
``` bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Take note of the kubeadm join output. It'll be required to add nodes into the cluster
``` bash
kubeadm join --token "token" "master-ip":"master-port"…
```

## Tokens in Master node can be retrieved using:
``` bash
kubeadm token list
```

In a non-prod env, slave nodes can join using below command to skip the token verification:
``` bash
kubeadm join --discovery-token-unsafe-skip-ca-verification --token="token" "master-ip":"master-port"
```

## Installing a pod network add-on (Calico in this case)
``` bash
kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml

kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml
``` 

## Confirm pod network is up and running
``` bash
kubectl get pods -o wide --all-namespaces
```

## Enable scheduling pods in master node
``` bash
kubectl taint nodes --all node-role.kubernetes.io/master-
```

## Deploy Kubernetes Dashboard
https://github.com/kubernetes/dashboard
``` bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
```

## Expose dashboard to external clients
https://github.com/kubernetes/dashboard/wiki/Accessing-Dashboard---1.7.X-and-above

1) Check dashboard before exposing it externally:
	``` bash
	kubectl -n kube-system get service kubernetes-dashboard
	```
2) Edit dashboard config
	``` bash
	kubectl -n kube-system edit service kubernetes-dashboard
	```
3) Change type: ClusterIP to type: NodePort
4) Check what port the dashboad was exposed: 
 	``` bash
	kubectl -n kube-system get service kubernetes-dashboard
	```
5) Access it from another machine like:  
	https://10.0.0.4/"port"

## Dashboard UI
http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/

## Generate token to access dashboard
```bash
kubectl create serviceaccount dashboard -n default

kubectl create clusterrolebinding dashboard-admin -n default \
--clusterrole=cluster-admin \
--serviceaccount=default:dashboard

kubectl get secret $(kubectl get serviceaccount dashboard -o jsonpath="{.secrets[0].name}") -o jsonpath="{.data.token}" | base64 --decode
```
Copy resulting token and paste it in the kubernetes dashboard prompt to sign in.

# SLAVE Node setup

## Adding a slave node into the cluster
1) From the note taken before, use:
	```bash
	kubeadm join "master-ip":"master-port" --token "token" --discovery-token-ca-cert-hash "hash"
	```
	If token expired, use below command in the master node:
	``` bash
	kubeadm token create
	```
	If you don’t have the value for --discovery-token-ca-cert-hash :
	``` bash
	openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
   openssl dgst -sha256 -hex | sed 's/^.* //'
	```
2) Check with the new node using below command in the master node:
	``` bash
	kubectl get nodes
	```
3) Check the new node in the dashboard