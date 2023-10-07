# Kubernetes Cluster Installation

## ARCHITECTURE
![alt text](https://github.com/shawn-njj/k8s-cluster-installation/blob/main/kube.drawio.png?raw=true)

### Bill of Materials (BOM)
- Minimum two Ubuntu nodes [One master and one worker node]. You can have more worker nodes as per your requirement.
- The master node should have a minimum of 2 vCPU and 2GB RAM.
- For the worker nodes, a minimum of 1vCPU and 2 GB RAM is recommended.
- 10.X.X.X/X network range with static IPs for master and worker nodes. We will be using the 192.x.x.x series as the pod network range that will be used by the Calico network plugin. Make sure the Node IP range and pod IP range DO NOT overlap.

LINKS = https://devopscube.com/setup-kubernetes-cluster-kubeadm/

## MASTER NODE / CONTROL PLANE
Ensure that inbound TCP ports (6443, 2379-2380, 10250-10252) are open

### Step 1: Update

- Pull packages
```
sudo apt-get update -y
```

- Update packages
```
sudo apt-get upgrade -y
```

- Restart
```
sudo reboot
```


### Step 2: Disable swap (persists after reboot)

- Turn off swap
```
sudo swapoff -a
```

- Turn off swap automatically during reboot
```
(crontab -l 2>/dev/null; echo "@reboot /sbin/swapoff -a") | crontab - || true
```


### Step 3: [Option 1] Install container runtime (CRI-O)

- Set environment variable (OS version) to be referenced as $VariableName later
```
OS="xUbuntu_22.04"
```

- Set environment variable (kubernetes version) to be referenced as $VariableName later
```
VERSION="1.28"
```

- Create file (crio.conf under /etc/modules-load.d/) & write configuration (overlay and br_netfilter modules)
```
cat <<EOF | sudo tee /etc/modules-load.d/crio.conf
overlay
br_netfilter
EOF
```

- Add module (overlay) into kernel
```
sudo modprobe overlay
```

- Add module (br_netfilter) into kernel
```
sudo modprobe br_netfilter
```

- Create file (99-kubernetes-cri.conf under /etc/sysctl.d/) & write configuration (enable iptables bridged traffic)
```
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```

- Reload sysctl
```
sudo sysctl --system
```

STOP HERERE STOP HERERERARERERER
- Enable cri-o repositories and gpg keys
```
cat <<EOF | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /
EOF
```
```
cat <<EOF | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list
deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/ /
EOF
```
```
curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -
```
```
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -
```

- Install CRI-O and CRI-O tools
```
sudo apt-get update
```
```
sudo apt-get install cri-o cri-o-runc -y
```

- Reload systemd and enable CRI-O
```
sudo systemctl daemon-reload
```
```
sudo systemctl enable crio --now
```


### Step 4: [Option 2] Install container runtime (containerd)
- XXX


### Step 4: [Option 3] Install container runtime (cri-dockerd)
- XXX



### Step 5: Install kubeadm and kubelet and kubectl

- Install required dependencies and download signing key
```
sudo apt-get update -y
```
```
sudo apt-get install -y apt-transport-https ca-certificates curl
```
```
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://dl.k8s.io/apt/doc/apt-key.gpg
```

- Install required dependencies
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
```

```
```

```





## WORKER NODE
Ensure that inbound TCP ports (10250, 30000-32767) are open

### Step 1: Pre-requisites
- Update packages
```
sudo apt-get update
```
```
sudo apt-get upgrade
```
```
sudo reboot
```

- Enable iptables Bridged Traffic
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```
```
sudo modprobe overlay
```
```
sudo modprobe br_netfilter
```
```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
```
```
sudo sysctl --system
```

- Disable swap
```
sudo swapoff -a
```
```
(crontab -l 2>/dev/null; echo "@reboot /sbin/swapoff -a") | crontab - || true
```


