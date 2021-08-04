# EKSCTL


Install AWS-CLI if not present:  
```shell
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
export PATH=$PATH:/usr/local/bin/
aws configure
```

## Install EKSCTL:  
```shell
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
. <(eksctl completion bash)
```

## Install Kubectl

```
curl -o kubectl https://amazon-eks.s3-us-west-2.amazonaws.com/1.20.4/2021-04-12/bin/linux/amd64/kubectl
chmod +x ./kubectl
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
kubectl version --short --client
```

### Create an EKS cluster

```shell
eksctl create cluster -f /vagrant/eks-config.yml
eksctl utils update-cluster-logging --enable-types all
eksctl utils write-kubeconfig --cluster=eks-lab-cluster --kubeconfig=/vagrant/eks-kube-config --set-kubeconfig-context=true
```

### Install Helm

```
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 > get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh
helm help
```

### Deploy Prometheus

```
kubectl create namespace prometheus
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm upgrade -i prometheus prometheus-community/prometheus \
    --namespace prometheus \
    --set alertmanager.persistentVolume.storageClass="gp2",server.persistentVolume.storageClass="gp2"
kubectl get pods -n prometheus
```

### Installing Metrics Server

```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl get deployment metrics-server -n kube-system
```

-----
References

- [EKSCTL - managing clusters](https://https://eksctl.io/usage/creating-and-managing-clusters/)
- [EKS Prometheus](https://docs.aws.amazon.com/eks/latest/userguide/prometheus.html)
