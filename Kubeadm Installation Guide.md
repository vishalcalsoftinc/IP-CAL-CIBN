
---
```bash
# to add user(username) to sudo
su -

usermod -aG sudo user

su user
```
---
```bash
$ sudo apt update
$ sudo apt install apt-transport-https curl -y

# Installing Containerd (on all nodes):

$ sudo mkdir -p /etc/apt/keyrings

$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

$ echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

$ sudo apt-get update

$ sudo apt-get install [containerd.io](http://containerd.io/) -y

# Picture

# Create containerd configuration (on all nodes):

$ sudo mkdir -p /etc/containerd
$ sudo containerd config default | sudo tee /etc/containerd/config.toml

# Picture

# Edit containerd configuration file (on all nodes):

$ sudo vi /etc/containerd/config.toml

# Set SystemdCgroup to true:
# SystemdCgroup = true

# Picture

$ sudo systemctl restart containerd

# Install Kubernetes (on all nodes):

$ curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

$ echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

$ sudo apt-get update
$ sudo apt-get install -y kubelet kubeadm kubectl
$ sudo apt-mark hold kubelet kubeadm kubectl
$ sudo systemctl enable --now kubelet

# Picture

# Disable swap (on all nodes):

$ sudo swapoff –a
$ sudo modprobe br_netfilter
$ sudo sysctl -w net.ipv4.ip_forward=1

$ sudo kubeadm init --pod-network-cidr=10.244.0.0/16 

kubectl taint nodes --all [node-role.kubernetes.io/control-plane-](http://node-role.kubernetes.io/control-plane-)

```