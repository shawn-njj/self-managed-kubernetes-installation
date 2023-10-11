# Kubernetes Cluster Installation



## ARCHITECTURE
![alt text](https://github.com/shawn-njj/k8s-cluster-installation/blob/main/kube.drawio.png?raw=true)

### Bill of Materials (BOM)
- Minimum two Ubuntu nodes [1 master and 1 worker node]. You can have more worker nodes as per your requirement.
- The master node should have a minimum of 2 vCPU and 2GB RAM.
- For the worker nodes, a minimum of 1vCPU and 2 GB RAM is recommended.
- 10.X.X.X/X network range with static IPs for master and worker nodes. We will be using the 192.x.x.x series as the pod network range that will be used by the Calico network plugin. Make sure the Node IP range and pod IP range DO NOT overlap.



## MASTER NODE / CONTROL PLANE
IMPORTANT: Ensure inbound TCP ports (6443, 2379-2380, 10250-10252) are open

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


### Step 2: Disable swap with persists after reboot

- Turn off swap
```
sudo swapoff -a
```

- Turn off swap automatically during reboot
```
(crontab -l 2>/dev/null; echo "@reboot /sbin/swapoff -a") | crontab - || true
```


### Step 3: [Option 1] Install container runtime cri-o

- Set OS version variable to be referenced as $OS later
```
OS="xUbuntu_22.04"
```

- Set kubernetes version variable to be referenced as $VERSION later
```
VERSION="1.28"
```

- Write/Append configuration to enable modules "overlay" & "br_netfilter" into file "/etc/modules-load.d/crio.conf" 
```
cat <<EOF | sudo tee /etc/modules-load.d/crio.conf
overlay
br_netfilter
EOF
```

- Load module "overlay" into kernel
```
sudo modprobe overlay
```

- Load module "br_netfilter" into kernel
```
sudo modprobe br_netfilter
```

- Write/Append configuration to enable iptables bridged traffic into file "/etc/sysctl.d/99-kubernetes-cri.conf"
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

- Write/Append configuration to add package source "https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/" into file "/etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list" 
```
cat <<EOF | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /
EOF
```

- Download .gpg private-public signing key for package source "https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/"
```
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -
```

- Write/Append configuration to add package source "http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/" into file "/etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list"
```
cat <<EOF | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list
deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/ /
EOF
```

- Download .gpg private-public signing key for package source "http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/"
```
curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -
```

- Install "cri-o" and "cri-o-runc"
```
sudo apt-get update -y && sudo apt-get install cri-o cri-o-runc -y
```

- Reload systemd
```
sudo systemctl daemon-reload
```

- Enable "cri-o"
```
sudo systemctl enable crio --now
```

- Verify "cri-o" is running
```
systemctl status -l crio
```


### Step 3: [Option 2] Install container runtime containerd
- To be updated
```

```

### Step 3: [Option 3] Install container runtime cri-dockerd
- To be updated
```

```

### Step 4: Install kubeadm and kubelet and kubectl

- Set kubernetes long version variable to be referenced as $KUBERNETES_VERSION later
```
KUBERNETES_VERSION="1.28.1-00"
```

- Install "apt-transport-https" and "ca-certificates" and "curl" and "jq"
```
sudo apt-get update -y && sudo apt-get install apt-transport-https ca-certificates curl jq -y
```

- Download .gpg private-public signing key for package source "https://apt.kubernetes.io/"
```
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://dl.k8s.io/apt/doc/apt-key.gpg
```

- Write/Append configuration to add package source "https://apt.kubernetes.io/" with the .gpg private-public signing key into file "/etc/apt/sources.list.d/kubernetes.list"
```
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

- Install kubeadm and kubelet and kubectl
```
sudo apt-get update -y && sudo apt-get install -y kubelet="$KUBERNETES_VERSION" kubectl="$KUBERNETES_VERSION" kubeadm="$KUBERNETES_VERSION"
```

- Retrieves the local IPv4 address for "eth0" network and stores it as variable to be referenced as $PRIVATE_IP later
```
PRIVATE_IP="$(ip --json addr show eth0 | jq -r '.[0].addr_info[] | select(.family == "inet") | .local')"
```

- Write/Append configuration to use the local machine's private IP address as node-IP of the kubelet system daemon into file "/etc/default/kubelet"
```
cat <<EOF | sudo tee /etc/default/kubelet
KUBELET_EXTRA_ARGS=--node-ip=$PRIVATE_IP
EOF
```


### Step 5: Pull required container images to set up Master Node / Control Plane components

- Pull 7 container images comprising kube-api-server / kube-controller-manager / kube-scheduler / kube-proxy / pause / etcd / coredns
```
sudo kubeadm config images pull
```


### Step 6: [Option 1] Set up Master Node / Control Plane using Public IP address

- Set Master Node / Control Plane hostname variable to be referenced as $NODENAME later
```
NODENAME=$(hostname -s)
```

- Set Master Node / Control Plane CIDR range variable to be referenced as $POD_CIDR later
```
POD_CIDR="192.168.0.0/16"
```

- Set Master Node / Control Plane Public IP address variable to be referenced as $MASTER_PUBLIC_IP later
```
MASTER_PUBLIC_IP=$(curl ifconfig.me && echo "")
```

- Run Master Node / Control Plane with Public IP address and Pod CIDR range and Node name
```
sudo kubeadm init --control-plane-endpoint="$MASTER_PUBLIC_IP" --apiserver-cert-extra-sans="$MASTER_PUBLIC_IP" --pod-network-cidr="$POD_CIDR" --node-name "$NODENAME" --ignore-preflight-errors Swap
```


### Step 6: [Option 2] Set up Master Node / Control Plane using Private IP address

- Set Master Node / Control Plane hostname variable to be referenced as $NODENAME later
```
NODENAME=$(hostname -s)
```

- Set Master Node / Control Plane CIDR range variable to be referenced as $POD_CIDR later
```
POD_CIDR="192.168.0.0/16"
```

- Set Master Node / Control Plane Private IP address variable to be referenced as $MASTER_PRIVATE_IP later
```
MASTER_PRIVATE_IP=$(ip addr show eth0 | awk '/inet / {print $2}' | cut -d/ -f1)
```

- Run Master Node / Control Plane with Private IP address and Pod CIDR range and Node name
```
sudo kubeadm init --apiserver-advertise-address="$MASTER_PRIVATE_IP" --apiserver-cert-extra-sans="$MASTER_PRIVATE_IP" --pod-network-cidr="$POD_CIDR" --node-name "$NODENAME" --ignore-preflight-errors Swap
```


### Step 7: Create Kubernetes configuration file

- Create hidden directory "$HOME/.kube"
```
mkdir -p $HOME/.kube
```

- Copy file "/etc/kubernetes/admin.conf" into file "$HOME/.kube/config"
```
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```

- Change file "$HOME/.kube/config" ownership to root user and group
```
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


### Step 8: [Option 1] Install CNI Calico

- Create Kubernetes resource "kind: CustomResourceDefinition" for Calico CNI
```
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml
```

- Create Kubernetes resource "kind: Installation" for Calico CNI
```
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/custom-resources.yaml
```

- Check Calico pods is running
```
kubectl get pod -n kube-system
```


### Step 8: [Option 2] Install CNI Flannel

- To be Updated
```

```


### Step 9: Assign master role to Master Node / Control Plane

- Verify Master Node / Control Plane is up
```
kubectl get nodes
```

- Assign role to Master Node / Control Plane
```
kubectl label node <MASTER_NODE_NAME> node-role.kubernetes.io/control-plane=control-plane
```



## WORKER NODE
IMPORTANT: Ensure inbound TCP ports (10250, 30000-32767) are open

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


### Step 2: Disable swap with persists after reboot

- Turn off swap
```
sudo swapoff -a
```

- Turn off swap automatically during reboot
```
(crontab -l 2>/dev/null; echo "@reboot /sbin/swapoff -a") | crontab - || true
```


### Step 3: [Option 1] Install container runtime cri-o

- Set OS version variable to be referenced as $OS later
```
OS="xUbuntu_22.04"
```

- Set kubernetes version variable to be referenced as $VERSION later
```
VERSION="1.28"
```

- Write/Append configuration to enable modules "overlay" & "br_netfilter" into file "/etc/modules-load.d/crio.conf" 
```
cat <<EOF | sudo tee /etc/modules-load.d/crio.conf
overlay
br_netfilter
EOF
```

- Load module "overlay" into kernel
```
sudo modprobe overlay
```

- Load module "br_netfilter" into kernel
```
sudo modprobe br_netfilter
```

- Write/Append configuration to enable iptables bridged traffic into file "/etc/sysctl.d/99-kubernetes-cri.conf"
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

- Write/Append configuration to add package source "https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/" into file "/etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list" 
```
cat <<EOF | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /
EOF
```

- Download .gpg private-public signing key for package source "https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/"
```
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -
```

- Write/Append configuration to add package source "http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/" into file "/etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list"
```
cat <<EOF | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list
deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/ /
EOF
```

- Download .gpg private-public signing key for package source "http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/"
```
curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -
```

- Install "cri-o" and "cri-o-runc"
```
sudo apt-get update -y && sudo apt-get install cri-o cri-o-runc -y
```

- Reload systemd
```
sudo systemctl daemon-reload
```

- Enable "cri-o"
```
sudo systemctl enable crio --now
```


### Step 3: [Option 2] Install container runtime containerd
- To be updated


### Step 3: [Option 3] Install container runtime cri-dockerd
- To be updated


### Step 4: Install kubeadm and kubelet and kubectl

- Set kubernetes long version variable to be referenced as $KUBERNETES_VERSION later
```
KUBERNETES_VERSION="1.28.1-00"
```

- Install "apt-transport-https" and "ca-certificates" and "curl" and "jq"
```
sudo apt-get update -y && sudo apt-get install apt-transport-https ca-certificates curl jq -y
```

- Download .gpg private-public signing key for package source "https://apt.kubernetes.io/"
```
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://dl.k8s.io/apt/doc/apt-key.gpg
```

- Write/Append configuration to add package source "https://apt.kubernetes.io/" with the .gpg private-public signing key into file "/etc/apt/sources.list.d/kubernetes.list"
```
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

- Install kubeadm and kubelet and kubectl
```
sudo apt-get update -y && sudo apt-get install -y kubelet="$KUBERNETES_VERSION" kubectl="$KUBERNETES_VERSION" kubeadm="$KUBERNETES_VERSION"
```

- Retrieves the local IPv4 address for "eth0" network and stores it as variable to be referenced as $PRIVATE_IP later
```
PRIVATE_IP="$(ip --json addr show eth0 | jq -r '.[0].addr_info[] | select(.family == "inet") | .local')"
```

- Write/Append configuration to use the local machine's private IP address as node-IP of the kubelet system daemon into file "/etc/default/kubelet"
```
cat <<EOF | sudo tee /etc/default/kubelet
KUBELET_EXTRA_ARGS=--node-ip=$PRIVATE_IP
EOF
```


### Step 5: Register Worker Node to Master Node / Control Plane

- Go to Master Node / Control Plane, generate token for Worker Node to join
```
kubeadm token create --print-join-command
```

- Go to Worker Node, join Master Node / Control Plane using token
```
sudo kubeadm join <MASTER_NODE_IP>:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<TOKEN_HASH>
```


### Step 6: Assign worker role to Worker Node

- Go to Master Node / Control Plane, verify Worker Node is up
```
kubectl get nodes
```

- Go to Master Node / Control Plane, Assign role to Worker Node
```
kubectl label node <WORKER_NODE_NAME> node-role.kubernetes.io/worker=worker
```