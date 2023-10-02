# k8s-ubuntu-installation

## Step 1: Update packages & reboot

```
sudo apt update && sudo apt -y full-upgrade [ -f /var/run/reboot-required ] && sudo reboot -f
```

## Step 2: Install Kubelet, Kubeadm, kubectl

- Add Kubernetes repository
```
sudo apt -y install curl apt-transport-https
```
```
curl  -fsSL  https://packages.cloud.google.com/apt/doc/apt-key.gpg|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/kubernetes.gpg
```
```
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

- Install vim, git, curl, wget, kubelet, kubeadm, kubectl
```
sudo apt update
```
```
sudo apt -y install vim git curl wget kubelet kubeadm kubectl
```
```
sudo apt-mark hold kubelet kubeadm kubectl
```

- Check version of kubectl to confirm installation
```
kubectl version --client && kubeadm version
```

## Step 2: Install Kubelet, Kubeadm, kubectl

- Show infrastructure to provision/change
```
terraform plan
```

- Provision resources
```
terraform apply -auto-approve
```

- Tear down resources
```
terraform destroy
```


