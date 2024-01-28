### Pre-requisites
- Ubuntu OS
- sudo privileges
- t2.medium Instance type (or higher)

### Both Master & Worker Node
```
sudo apt update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo apt install docker.io -y
sudo systemctl enable --now docker

curl -fsSL "https://packages.cloud.google.com/apt/doc/apt-key.gpg" | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/kubernetes-archive-keyring.gpg
echo 'deb https://packages.cloud.google.com/apt kubernetes-xenial main' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update 
sudo apt install kubeadm=1.20.0-00 kubectl=1.20.0-00 kubelet=1.20.0-00 -y
```

### Master Node
```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --kubernetes-version=1.20.0
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Install Flannel network
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml```
```

Note: Expose port 6443 in the Security group for the Worker to connect to Master Node

### Worker Node
```
sudo <join_token> --v=5
```

### Verify
```
kubectl get nodes
kubectl run hello-world-pod --image=nginx
kubectl get pods
```
