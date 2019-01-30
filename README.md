# Kubernetes cluster setup:
https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/

# Objectives
The objective of this vagrant file is to spin up and provision a VirtualBox VM with the following especifications:

- OS: ubuntu-18.04
- CPU: 2
- Memory: 2 GB
- Private IP: 10.0.0.4
- Shell: ZShell + Oh My SZh (only for fun)
- Basic Packages: GIT, Docker 18.06, Kubelet, Kubeadm and Kubectl

# Requirements

- [VirtualBox](https://www.virtualbox.org/)
- [Vagrant](https://www.vagrantup.com/downloads.html)

# Advantages
The main advantage of using vagrant for this kind of testing environment is that it enables us to us a completely isolated VM which can be built, destroyed and re-built in a matter of minutes.

# Scope of provisioning
This vagrant script is mostly intended for the OS build, base applications and core components installation required for rolling out a Kubernetes node. This node can later become a **Master node** or a **Worker node** using instructions provided below.

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
kubeadm join "master-ip":"master-port" --token "token" --discovery-token-ca-cert-hash "hash"
```

## Tokens in Master node can be retrieved using:
``` bash
kubeadm token list
```

In a non-prod environment, slave nodes can join using below command to skip the token verification:
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

## Deploy Kubernetes Dashboard (Optional - Recommended)
https://github.com/kubernetes/dashboard
``` bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
```

## Expose dashboard for external access
https://github.com/kubernetes/dashboard/wiki/Accessing-Dashboard---1.7.X-and-above

1) Check dashboard before exposing it externally:
	``` bash
	kubectl -n kube-system get service kubernetes-dashboard
	```
2) Edit dashboard config
	``` bash
	kubectl -n kube-system edit service kubernetes-dashboard
	```
3) Change `type: ClusterIP` to `type: NodePort`

4) Check what port the dashboad was exposed at: 
 	``` bash
	kubectl -n kube-system get service kubernetes-dashboard
	```

5) Generate token to access dashboard
	```bash
	kubectl create serviceaccount dashboard -n default

	kubectl create clusterrolebinding dashboard-admin -n default \
	--clusterrole=cluster-admin \
	--serviceaccount=default:dashboard

	kubectl get secret $(kubectl get serviceaccount dashboard -o jsonpath="{.secrets[0].name}") -o jsonpath="{.data.token}" | base64 --decode
	```
	Copy resulting token and paste it in the kubernetes dashboard prompt to sign in.

6) Access the dashboard from another machine like (i.e. Host machine):  
	https://10.0.0.4/"dashboard-port"

# SLAVE Node setup

## Adding a slave node into the cluster
1) From the note taken before, use:
	```bash
	kubeadm join "master-ip":"master-port" --token "token" --discovery-token-ca-cert-hash "hash"
	```
	If token expired, use below command in the master node to create a new one:
	``` bash
	kubeadm token create
	```
	If you don’t have the value for --discovery-token-ca-cert-hash :
	``` bash
	openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
   openssl dgst -sha256 -hex | sed 's/^.* //'
	```
2) Check the new node using below command in the master node:
	``` bash
	kubectl get nodes
	```
