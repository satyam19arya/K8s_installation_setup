### Pre-requisites
- Ubuntu OS
- sudo privileges
- t2.medium Instance type (or higher)

```
sudo apt update
sudo apt install -y curl wget apt-transport-https
sudo apt install docker.io -y
sudo systemctl enable --now docker
sudo usermod -aG docker $USER && newgrp docker
```

### Install Minikube
```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x minikube
sudo mv minikube /usr/local/bin/
```

### Install Kubectl
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

### Minikube commands
```
minikube start --driver=docker
minikube status
minikube stop
minikube delete
```

### Verify
```
kubectl get nodes
kubectl run hello-world-pod --image=busybox --restart=Never --command -- sh -c "echo 'Hello World' && sleep 3600"
kubectl get pods
```
