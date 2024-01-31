> [!NOTE]
> Expose port 6443 in the Security Group for the Worker Node to connect to Master Node

### Installing a container runtime (containerd) (Both Master and Worker)
```
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
```
```
modprobe overlay
modprobe br_netfilter
```
```
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```
```
sysctl --system
```
```
apt-get install -y containerd
mkdir -p /etc/containerd
containerd config default > /etc/containerd/config.toml
```
```
vim /etc/containerd/config.toml
```
> SystemdCgroup = true
```
systemctl restart containerd
```
```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```
```
sudo sysctl --system
```

Configuring Repo and Installation (Both Master and Worker)
```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
Installing kubeadm, kubelet and kubectl
```
sudo apt-get update
apt-cache madison kubeadm
sudo apt-get install -y kubelet=1.28.0-00 kubeadm=1.28.0-00 kubectl=1.28.0-00
sudo apt-mark hold kubelet kubeadm kubectl
```

### Initialize Cluster with kubeadm (Only Master)
```
kubeadm init --pod-network-cidr=10.244.0.0/16 --kubernetes-version=1.28.0
```
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
To Install Flannel
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

Connect Worker Node (Only Worker nodes)
```
kubeadm join 172.31.94.169:6443 --token ubabxy.owfrp9cqhm76dmlw \
        --discovery-token-ca-cert-hash sha256:3d9d6e347dd114bbe4a07801ced0189170bca1d35108f101fc27f26b4b764a92
```

Verify
```
kubectl get nodes
kubectl run nginx --image=nginx
kubectl get pods -o wide
```
Ref: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

To Upgrade Kubeadm Cluster
```
apt-cache madison kubeadm
sudo apt-mark unhold kubelet kubectl kubeadm
sudo apt-get update && apt-get install -y kubelet=1.28.2-00 kubectl=1.28.2-00 kubeadm=1.28.2-00
sudo apt-mark hold kubelet kubectl
kubeadm version
kubeadm upgrade plan
kubeadm upgrade apply 1.28.2
systemctl daemon-reload
systemctl restart kubelet
```

To Install Calico
```
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/tigera-operator.yaml
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/custom-resources.yaml
```
> [!NOTE]
> Before creating this manifest, read its contents and make sure its settings are correct for your environment. For example, you may need to change the default IP pool CIDR to match your pod network CIDR.

Verify Calico installation in your cluster
```
kubectl get pods -n calico-system
```

Ref: https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart
