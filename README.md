# Self-Managed Kubernetes Installation



## ARCHITECTURE
![alt text](https://github.com/shawn-njj/k8s-cluster-installation/blob/main/kube.drawio.png?raw=true)

### Bill of Materials (BOM)
- Minimum two Ubuntu nodes [1 master and 1 worker node]. You can have more worker nodes as per your requirement.
- The master node should have a minimum of 2 vCPU and 2GB RAM.
- For the worker nodes, a minimum of 1vCPU and 2 GB RAM is recommended.
- 10.X.X.X/X network range with static IPs for master and worker nodes. We will be using the 192.x.x.x series as the pod network range that will be used by the Calico network plugin. Make sure the Node IP range and pod IP range DO NOT overlap.



## MASTER-NODE/CONTROL-PLANE
IMPORTANT: Ensure inbound TCP ports (6443, 2379-2380, 10250-10252) are open

### Step 1: [Option 1] Update on Ubuntu

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


### Step 1: [Option 2] Update on RHEL/CentOS

- Pull packages
```
sudo dnf makecache -y
```

- Update packages
```
sudo dnf update -y
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


### Step 3: [Option 1] Set up container runtime cri-o on Ubuntu

- Set variable "OS" referenced "$OS"
```
OS="xUbuntu_22.04"
```

- Set variable "KUBERNETES_SHORT_VERSION" referenced "$KUBERNETES_SHORT_VERSION"
```
KUBERNETES_SHORT_VERSION="1.28"
```

- Write line(s) "overlay" + "br_netfilter" into file "/etc/modules-load.d/crio.conf"
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

- Write line(s) "net.bridge.bridge-nf-call-iptables = 1" + "net.ipv4.ip_forward = 1" + "net.bridge.bridge-nf-call-ip6tables = 1" into file "/etc/sysctl.d/99-crio.conf"
```
cat <<EOF | sudo tee /etc/sysctl.d/99-crio.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```

- Reload sysctl
```
sudo sysctl --system
```

- Write line(s) "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /" into file "/etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list"
```
cat <<EOF | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /
EOF
```

- Download file "https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key" into file "/etc/apt/trusted.gpg.d/libcontainers.gpg"
```
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -
```

- Write line(s) "deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$KUBERNETES_SHORT_VERSION/$OS/ /" into file "/etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$KUBERNETES_SHORT_VERSION.list"
```
cat <<EOF | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$KUBERNETES_SHORT_VERSION.list
deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$KUBERNETES_SHORT_VERSION/$OS/ /
EOF
```

- Download file "https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$KUBERNETES_SHORT_VERSION/$OS/Release.key" into file "/etc/apt/trusted.gpg.d/libcontainers.gpg"
```
curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$KUBERNETES_SHORT_VERSION/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -
```

- Install "cri-o" + "cri-o-runc"
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


### Step 3: [Option 2] Set up container runtime containerd on Ubuntu

- To be updated
```

```

### Step 3: [Option 3] Set up container runtime cri-o on RHEL/CentOS

- To be updated
```

```


### Step 3: [Option 4] Set up container runtime containerd on RHEL/CentOS

- To be updated
```

```


### Step 4: [Option 1] Set up kubeadm and kubelet and kubectl on Ubuntu

- Set variable "KUBERNETES_LONG_VERSION" referenced "$KUBERNETES_LONG_VERSION"
```
KUBERNETES_LONG_VERSION="1.28.1-00"
```

- Set variable "PRIVATE_IP" referenced "$PRIVATE_IP"
```
PRIVATE_IP="$(ip --json addr show eth0 | jq -r '.[0].addr_info[] | select(.family == "inet") | .local')"
```

- Install "apt-transport-https" + "ca-certificates" + "curl" + "jq"
```
sudo apt-get update -y && sudo apt-get install apt-transport-https ca-certificates curl jq -y
```

- Download file "https://dl.k8s.io/apt/doc/apt-key.gpg" into file "/usr/share/keyrings/kubernetes-archive-keyring.gpg"
```
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://dl.k8s.io/apt/doc/apt-key.gpg
```

- Write line(s) "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" into file "/etc/apt/sources.list.d/kubernetes.list"
```
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

- Install "kubeadm" + "kubelet" + "kubectl"
```
sudo apt-get update -y && sudo apt-get install -y kubelet="$KUBERNETES_LONG_VERSION" kubectl="$KUBERNETES_LONG_VERSION" kubeadm="$KUBERNETES_LONG_VERSION"
```

- Write line(s) "KUBELET_EXTRA_ARGS=--node-ip=$PRIVATE_IP" into file "/etc/default/kubelet"
```
cat <<EOF | sudo tee /etc/default/kubelet
KUBELET_EXTRA_ARGS=--node-ip=$PRIVATE_IP
EOF
```


### Step 4: [Option 2] Set up kubeadm and kubelet and kubectl on RHEL/CentOS

- To be updated
```

```


### Step 5: [Option 1] Set up Master-Node/Control-Plane using Public IP address

- Set variable "NODENAME" referenced "$NODENAME"
```
NODENAME=$(hostname -s)
```

- Set variable "POD_CIDR" referenced "$POD_CIDR"
```
POD_CIDR="192.168.0.0/16"
```

- Set variable "MASTER_PUBLIC_IP" referenced "$MASTER_PUBLIC_IP"
```
MASTER_PUBLIC_IP=$(curl ifconfig.me && echo "")
```

- Pull 7 container-image(s) "kube-api-server" + "kube-controller-manager" + "kube-scheduler" + "kube-proxy" + "pause" + "etcd" + "coredns"
```
sudo kubeadm config images pull
```

- Run Master-Node/Control-Plane
```
sudo kubeadm init --control-plane-endpoint="$MASTER_PUBLIC_IP" --apiserver-cert-extra-sans="$MASTER_PUBLIC_IP" --pod-network-cidr="$POD_CIDR" --node-name "$NODENAME" --ignore-preflight-errors Swap
```


### Step 5: [Option 2] Set up Master-Node/Control-Plane using Private IP address

- Set variable "NODENAME" referenced "$NODENAME"
```
NODENAME=$(hostname -s)
```

- Set variable "POD_CIDR" referenced "$POD_CIDR"
```
POD_CIDR="192.168.0.0/16"
```

- Set variable "MASTER_PUBLIC_IP" referenced "$MASTER_PUBLIC_IP"
```
MASTER_PRIVATE_IP=$(ip addr show eth0 | awk '/inet / {print $2}' | cut -d/ -f1)
```

- Pull 7 container-image(s) "kube-api-server" + "kube-controller-manager" + "kube-scheduler" + "kube-proxy" + "pause" + "etcd" + "coredns"
```
sudo kubeadm config images pull
```

- Run Master-Node/Control-Plane
```
sudo kubeadm init --apiserver-advertise-address="$MASTER_PRIVATE_IP" --apiserver-cert-extra-sans="$MASTER_PRIVATE_IP" --pod-network-cidr="$POD_CIDR" --node-name "$NODENAME" --ignore-preflight-errors Swap
```


### Step 6: Configure kubectl in Master-Node/Control-Plane to have access to the cluster

- Create directory "$HOME/.kube"
```
mkdir -p $HOME/.kube
```

- Copy file "/etc/kubernetes/admin.conf" into file "$HOME/.kube/config"
```
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```

- Change file "$HOME/.kube/config" ownership "$(id -u):$(id -g)"
```
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


### Step 7: [Option 1] Set up CNI Calico

- Create kubernetes-resource "kind: Namespace" + "kind: CustomResourceDefinition"
```
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml
```

- Create kubernetes-resource "kind: Installation"
```
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/custom-resources.yaml
```

- Verify pods "calico" is running
```
kubectl get pods --all-namespaces
```


### Step 7: [Option 2] Set up CNI Flannel

- Create kubernetes-resource "kind: Namespace" + "kind: ClusterRole" + "kind: ClusterRoleBinding" + "kind: ServiceAccount" + "kind: ConfigMap" + "kind: DaemonSet"
```
kubectl apply -f https://github.com/coreos/flannel/raw/master/Documentation/kube-flannel.yml
```

- Verify pods "flannel" is running
```
kubectl get pods --all-namespaces
```


### Step 8: Assign role to Master-Node/Control-Plane

- Verify Master-Node/Control-Plane is up
```
kubectl get nodes
```

- Assign role to Master-Node/Control-Plane
```
kubectl label node <MASTER_NODE_NAME> node-role.kubernetes.io/master=true
```



## WORKER-NODE
IMPORTANT: Ensure inbound TCP ports (10250, 30000-32767) are open

### Step 1: [Option 1] Update on Ubuntu

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


### Step 1: [Option 2] Update on RHEL/CentOS

- Pull packages
```
sudo dnf makecache -y
```

- Update packages
```
sudo dnf update -y
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


### Step 3: [Option 1] Set up container runtime cri-o on Ubuntu

- Set variable "OS" referenced "$OS"
```
OS="xUbuntu_22.04"
```

- Set variable "KUBERNETES_SHORT_VERSION" referenced "$KUBERNETES_SHORT_VERSION"
```
KUBERNETES_SHORT_VERSION="1.28"
```

- Write line(s) "overlay" + "br_netfilter" into file "/etc/modules-load.d/crio.conf"
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

- Write line(s) "net.bridge.bridge-nf-call-iptables = 1" + "net.ipv4.ip_forward = 1" + "net.bridge.bridge-nf-call-ip6tables = 1" into file "/etc/sysctl.d/99-crio.conf"
```
cat <<EOF | sudo tee /etc/sysctl.d/99-crio.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```

- Reload sysctl
```
sudo sysctl --system
```

- Write line(s) "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /" into file "/etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list"
```
cat <<EOF | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /
EOF
```

- Download file "https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key" into file "/etc/apt/trusted.gpg.d/libcontainers.gpg"
```
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -
```

- Write line(s) "deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$KUBERNETES_SHORT_VERSION/$OS/ /" into file "/etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$KUBERNETES_SHORT_VERSION.list"
```
cat <<EOF | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$KUBERNETES_SHORT_VERSION.list
deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$KUBERNETES_SHORT_VERSION/$OS/ /
EOF
```

- Download file "https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$KUBERNETES_SHORT_VERSION/$OS/Release.key" into file "/etc/apt/trusted.gpg.d/libcontainers.gpg"
```
curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$KUBERNETES_SHORT_VERSION/$OS/Release.key | sudo apt-key --keyring /etc/apt/trusted.gpg.d/libcontainers.gpg add -
```

- Install "cri-o" + "cri-o-runc"
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


### Step 3: [Option 2] Set up container runtime containerd on Ubuntu

- To be updated
```

```


### Step 3: [Option 3] Set up container runtime cri-o on RHEL/CentOS

- To be updated
```

```


### Step 3: [Option 4] Set up container runtime containerd on RHEL/CentOS

- To be updated
```

```


### Step 4: [Option 1] Set up kubeadm and kubelet and kubectl on Ubuntu

- Set variable "KUBERNETES_LONG_VERSION" referenced "$KUBERNETES_LONG_VERSION"
```
KUBERNETES_LONG_VERSION="1.28.1-00"
```

- Set variable "PRIVATE_IP" referenced "$PRIVATE_IP"
```
PRIVATE_IP="$(ip --json addr show eth0 | jq -r '.[0].addr_info[] | select(.family == "inet") | .local')"
```

- Install "apt-transport-https" + "ca-certificates" + "curl" + "jq"
```
sudo apt-get update -y && sudo apt-get install apt-transport-https ca-certificates curl jq -y
```

- Download file "https://dl.k8s.io/apt/doc/apt-key.gpg" into file "/usr/share/keyrings/kubernetes-archive-keyring.gpg"
```
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://dl.k8s.io/apt/doc/apt-key.gpg
```

- Write line(s) "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" into file "/etc/apt/sources.list.d/kubernetes.list"
```
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

- Install "kubeadm" + "kubelet" + "kubectl"
```
sudo apt-get update -y && sudo apt-get install -y kubelet="$KUBERNETES_LONG_VERSION" kubectl="$KUBERNETES_LONG_VERSION" kubeadm="$KUBERNETES_LONG_VERSION"
```

- Write line(s) "KUBELET_EXTRA_ARGS=--node-ip=$PRIVATE_IP" into file "/etc/default/kubelet"
```
cat <<EOF | sudo tee /etc/default/kubelet
KUBELET_EXTRA_ARGS=--node-ip=$PRIVATE_IP
EOF
```


### Step 4: [Option 2] Set up kubeadm and kubelet and kubectl on RHEL/CentOS

- To be updated
```

```


### Step 5: Register Worker-Node to Master-Node/Control-Plane

- Generate token for Worker-Node to join
> At Master-Node/Control-Plane:
```
kubeadm token create --print-join-command
```

- Join Master-Node/Control-Plane using token
> At Worker-Node:
```
sudo kubeadm join <MASTER_NODE_IP>:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<TOKEN_HASH>
```


### Step 6: Configure kubectl in Worker-Node to have access to the cluster

- Run python3-HTTP-server on directory "/etc/kubernetes"
> At Master-Node/Control-Plane:
```
cd /etc/kubernetes && sudo python3 -m http.server 9001
```

- Download file "http://<MASTER_NODE_IP>:9001/admin.conf" into file "/tmp/admin.conf"
> At Worker-Node:
```
sudo curl http://<MASTER_NODE_IP>:9001/admin.conf > /tmp/admin.conf
```

- Create directory "$HOME/.kube"
> At Worker-Node:
```
mkdir -p $HOME/.kube
```

- Copy file "/tmp/admin.conf" into file "$HOME/.kube/config"
> At Worker-Node:
```
sudo cp -i /tmp/admin.conf $HOME/.kube/config
```

- Change file "$HOME/.kube/config" ownership "$(id -u):$(id -g)"
> At Worker-Node:
```
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


### Step 7: Assign role to Worker-Node

- Verify Worker Node is up
```
kubectl get nodes
```

- Assign role to Worker Node
```
kubectl label node <WORKER_NODE_NAME> node-role.kubernetes.io/worker=worker
```
