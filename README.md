# Kubernetes Cluster Installation

## ARCHITECTURE
![alt text](https://github.com/shawn-njj/k8s-cluster-installation/blob/main/kube.drawio.png?raw=true)

### Bill of Materials (BOM)
- Minimum two Ubuntu nodes [One master and one worker node]. You can have more worker nodes as per your requirement.
- The master node should have a minimum of 2 vCPU and 2GB RAM.
- For the worker nodes, a minimum of 1vCPU and 2 GB RAM is recommended.
- 10.X.X.X/X network range with static IPs for master and worker nodes. We will be using the 192.x.x.x series as the pod network range that will be used by the Calico network plugin. Make sure the Node IP range and pod IP range don’t overlap.


## MASTER NODE / CONTROL PLANE
Ensure that inbound TCP ports (6443, 2379-2380, 10250-10252) are open

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


