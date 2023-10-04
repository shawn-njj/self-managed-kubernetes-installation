# Kubernetes Cluster Installation

## ARCHITECTURE


## MASTER NODE / CONTROL PLANE






## WORKER NODE




### Step 1: Update

- Upgrade all packages to latest and reboot
```
sudo apt update && sudo apt -y full-upgrade [ -f /var/run/reboot-required ] && sudo reboot -f
```

### Step 1: Update

- Upgrade all packages to latest and reboot
```
sudo apt update && sudo apt -y full-upgrade [ -f /var/run/reboot-required ] && sudo reboot -f
```

### Step 1: Update

- Upgrade all packages to latest and reboot
```
sudo apt update && sudo apt -y full-upgrade [ -f /var/run/reboot-required ] && sudo reboot -f
```

### Step 1: Update

- Upgrade all packages to latest and reboot
```
sudo apt update && sudo apt -y full-upgrade [ -f /var/run/reboot-required ] && sudo reboot -f
```














































## Step 2: Install kind (local kubernetes) via binary

- Option 1: For AMD64 / x86_64
```
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
```

- Option 2: For ARM64
```
[ $(uname -m) = aarch64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-arm64
```

- Set executable for kind binary
```
chmod +x ./kind
```

- Move kind binary to /usr/local/bin
```
sudo mv ./kind /usr/local/bin/kind
```

- Validate installation
```
kind --version
```

## Step 3: Install docker
- Validate installation
```
sudo apt-get remove docker docker-engine docker.io
```

- Validate installation
```
kind --version
```

- Validate installation
```
kind --version
```

- Validate installation
```
kind --version
```

- Validate installation
```
kind --version
```

## Step 3: Create kubernetes cluster

- Create 1 cluster
```
kind create cluster
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
