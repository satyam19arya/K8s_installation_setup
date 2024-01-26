### Install dependencies (Python3, AWS CLI, kubectl)
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y python3-pip apt-transport-https kubectl
pip3 install awscli --upgrade
export PATH="$PATH:/home/ubuntu/.local/bin/"
```

### Install KOPS
```
curl -Lo kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x kops
sudo mv kops /usr/local/bin/kops
```

Create S3 bucket to store the state of your cluster
```
aws s3api create-bucket --bucket kops-storage --region us-east-1
```
To implement versioning your S3 bucket in case you ever need to revert or recover a previous state store (Optional)
```
aws s3api put-bucket-versioning --bucket kops-storage --versioning-configuration Status=Enabled
```

Create the cluster
```
kops create cluster --name=demok8scluster1.k8s.local --state=s3://kops-storage --zones=us-east-1a --node-count=2 --node-size=t2.medium --master-size=t2.medium  --master-volume-size=15 --node-volume-size=15
```

Build the cluster
```
kops update cluster demok8scluster1.k8s.local --yes --state=s3://kops-storage
```

Verify it
```
kops validate cluster demok8scluster1.k8s.local
```

To delete the cluster
```
kops delete cluster --name ${NAME} --yes
```

Ref: https://kops.sigs.k8s.io/getting_started/aws/
