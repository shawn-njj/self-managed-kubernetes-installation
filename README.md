# k8s-ubuntu-installation

## Step 1: Update

- Upgrade all packages to latest and reboot
```
sudo apt update && sudo apt -y full-upgrade [ -f /var/run/reboot-required ] && sudo reboot -f
```

## Step 2: Install kubelet, kubeadm, kubectl

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

## Step 3: Disable swap

- Turn off swap
```
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

- Ensure there is no lines with "swap", otherwise comment them with "#"
```
sudo vi /etc/fstab
```

- Check swap is not used (0GB)
```
sudo swapoff -a
```
```
sudo mount -a
```
```
free -h
```

## Step 4: Configure kernel modules and sysctl

- Set kernel modules
```
sudo modprobe overlay
```
```
sudo modprobe br_netfilter
```

- Set sysctl
```
sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
```
```
net.bridge.bridge-nf-call-ip6tables = 1
```
```
net.bridge.bridge-nf-call-iptables = 1
```
```
net.ipv4.ip_forward = 1
```
```
EOF
```
```
sudo sysctl --system
```

## Step 5: Install container runtime (docker)
- Add docker repository
```
sudo apt update
```
```
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
```
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/docker-archive-keyring.gpg
```
```
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

- Install docker
```
sudo apt update
```
```
sudo apt install -y containerd.io docker-ce docker-ce-cli
```
```
sudo mkdir -p /etc/systemd/system/docker.service.d
```
```
sudo tee /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
```
```
sudo systemctl daemon-reload
```
```
sudo systemctl restart docker
```
```
sudo systemctl enable docker
```

## Step 6: Install Mirantis cri-dockerd as docker engine shim for kubernetes
- Ensure docker is running
```
systemctl status docker
```

- Download latest binary of cri-dockerd
```
VER=$(curl -s https://api.github.com/repos/Mirantis/cri-dockerd/releases/latest|grep tag_name | cut -d '"' -f 4|sed 's/v//g')
```
```
echo $VER
```
```
wget https://github.com/Mirantis/cri-dockerd/releases/download/v${VER}/cri-dockerd-${VER}.amd64.tgz
```
```
tar xvf cri-dockerd-${VER}.amd64.tgz
```

- Move cri-dockerd binary to /usr/local/bin
```
sudo mv cri-dockerd/cri-dockerd /usr/local/bin/
```

- Validate cri-dockerd is installed
```
cri-dockerd --version
```

- Configure systemd units for cri-dockerd
```
wget https://raw.githubusercontent.com/Mirantis/cri-dockerd/master/packaging/systemd/cri-docker.service
```
```
wget https://raw.githubusercontent.com/Mirantis/cri-dockerd/master/packaging/systemd/cri-docker.socket
```
```
sudo mv cri-docker.socket cri-docker.service /etc/systemd/system/
```
```
sudo sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
```
```
sudo systemctl daemon-reload
```
```
sudo systemctl enable cri-docker.service
```
```
sudo systemctl enable --now cri-docker.socket
```
```
systemctl status cri-docker.socket
```

- Ensure docker is still running
```
systemctl status docker
```

- Pull 7 containers (api-server / controller / scheduler / proxy / pause / etcd / coredns) into cri-dockerd (to be pulled into master node later)
```
sudo kubeadm config images pull --cri-socket /run/cri-dockerd.sock
```

- Create k8s cluster (control plane + worker nodes) and set CIDR range. Result: "Your Kubernetes control-plane has initialized successfully"
```
sudo kubeadm init \
  --pod-network-cidr=10.244.0.0/16 \
  --cri-socket /run/cri-dockerd.sock
```

- Configure container runtime endpoint
```
sudo cat /var/lib/kubelet/kubeadm-flags.env
```

## Step 7: Initialize master node
- Ensure "br_netfilter" is loaded
```
lsmod | grep br_netfilter
```

- Enable kubelet service
```
sudo systemctl enable kubelet
```

- Pull 7 containers (api-server / controller / scheduler / proxy / pause / etcd / coredns) into master node (from cri-dockerd)
```
sudo kubeadm config images pull --cri-socket unix:///run/cri-dockerd.sock
```

- Option 1: Create cluster WITHOUT DNS endpoint
```
sudo sysctl -p
```
```
sudo kubeadm init \
  --pod-network-cidr=172.24.0.0/16 \
  --cri-socket unix:///run/cri-dockerd.sock
```

- Option 2: Create cluster WITH DNS endpoint
```
sudo vi /etc/hosts
```
```
172.29.20.5 yourDomain.com
```
```
sudo sysctl -p
```
```
sudo kubeadm init \
  --pod-network-cidr=172.24.0.0/16 \
  --upload-certs \
  --control-plane-endpoint=yourDomain.com
```



```

```
```

```
```

```
```

```
```

```
```

```
```

```
```

```
