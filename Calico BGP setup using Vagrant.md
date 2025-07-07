
---
Of course. This is the perfect use case for Vagrant. It will automate the creation of your two lightweight, CLI-only VMs, making your entire lab setup reproducible with just a few commands. This guide will walk you through creating the infrastructure with Vagrant and then deploying your OAI and Open5GS Kubernetes clusters as per your new network diagram.

### **Architecture Overview**
[![](https://mermaid.ink/img/pako:eNqtlnFvokgYxr_Km9lkYxNBGECUvTSx9LZnbuuSuvaP1YuZwqCkyJBhcO11-91vQFRslU3vbkiUeed9nndkfs7wjHwWUOSgBSfpEr5dzxKQLcsftoE_WCbglvjLKKEwLXtfx6DAXZ5kcB9xkZP4im3gI9wTqUjEX1uDogURp76IWAJf7g7R9TZv7sdRq9KA-2V4MUsOOfvy65UO0_tb0B1gJFIee5myXtVKHJf5dnU8ItXzKJ0OPQd0G6u6rZry03ql3xeT7nM5HZj-mT9QnlBBM3DjPBOUF794MHolbC5etMpx7rNEcBZP3e03eDFJ6G8PvHM58IYwpnxNeeMc62YBFSSKs6nHAnCH13dSqKlYU-XV0bula-EY-bQ2rNeHXRJHPoOrGw8G45EDXdPS8YmK-ydTlE1ZkMH062AIXnGnVFU36qYYXtAT-vMQ1Jtc2HlOW4Xz5PeL8zlBXuZcTxpy_G2OO1FcrzPxTmTSJDgOHgXKzkkOcckhlhymNLEW2ftZxG9ZxHYTiz7j9AyM1g24cvRfAVnYvpvINzM9cjuJpNGMpPFLJI0mJMu6FZPFityMa1wa_weXZQWyCluD288XDSmZTBk3p-Rp2Jp4n_8rjrWtExQFZqgKQJYtZ0iGLost79M7BbgS1IrtdsJSU6yJRymPkkW5VC2PR2siKIyo-MH4Yx0TrYPNi8p5R0dlv99DFFW5_DnCnZEBTFIGI3kAeYwLqBDJfh4W99NuRpl4iunxaRRGcex8CPvF1c4kxY_U-WAYRnWv_IgCsXRwuqk7FAfKVki7oRGGe6Gm2Q_77jktPmjD0NhrcY_YptWo3T3RatK_KKynm10gINmScE6eHLDAem1ZbhFvPU9N6KwnasvDPwqQE5I4o220onxFij56LsrNkFjSFZ0hR94GNCR5LGZolrxIXUqS74ytkCN4LpWc5Yvl3idPA0nJdVQgd0iRlFHusjwRyMH9fumBnGe0QU7PUG0Ty3--3evbuma00RNy9K5qmjJq9zWsdTXTsF7a6O-yqKb2TMPsa5ap9Xs67mO7jWgQCcZvt-805avNyz96b4Uw?type=png)](https://mermaid.live/edit#pako:eNqtlnFvokgYxr_Km9lkYxNBGECUvTSx9LZnbuuSuvaP1YuZwqCkyJBhcO11-91vQFRslU3vbkiUeed9nndkfs7wjHwWUOSgBSfpEr5dzxKQLcsftoE_WCbglvjLKKEwLXtfx6DAXZ5kcB9xkZP4im3gI9wTqUjEX1uDogURp76IWAJf7g7R9TZv7sdRq9KA-2V4MUsOOfvy65UO0_tb0B1gJFIee5myXtVKHJf5dnU8ItXzKJ0OPQd0G6u6rZry03ql3xeT7nM5HZj-mT9QnlBBM3DjPBOUF794MHolbC5etMpx7rNEcBZP3e03eDFJ6G8PvHM58IYwpnxNeeMc62YBFSSKs6nHAnCH13dSqKlYU-XV0bula-EY-bQ2rNeHXRJHPoOrGw8G45EDXdPS8YmK-ydTlE1ZkMH062AIXnGnVFU36qYYXtAT-vMQ1Jtc2HlOW4Xz5PeL8zlBXuZcTxpy_G2OO1FcrzPxTmTSJDgOHgXKzkkOcckhlhymNLEW2ftZxG9ZxHYTiz7j9AyM1g24cvRfAVnYvpvINzM9cjuJpNGMpPFLJI0mJMu6FZPFityMa1wa_weXZQWyCluD288XDSmZTBk3p-Rp2Jp4n_8rjrWtExQFZqgKQJYtZ0iGLost79M7BbgS1IrtdsJSU6yJRymPkkW5VC2PR2siKIyo-MH4Yx0TrYPNi8p5R0dlv99DFFW5_DnCnZEBTFIGI3kAeYwLqBDJfh4W99NuRpl4iunxaRRGcex8CPvF1c4kxY_U-WAYRnWv_IgCsXRwuqk7FAfKVki7oRGGe6Gm2Q_77jktPmjD0NhrcY_YptWo3T3RatK_KKynm10gINmScE6eHLDAem1ZbhFvPU9N6KwnasvDPwqQE5I4o220onxFij56LsrNkFjSFZ0hR94GNCR5LGZolrxIXUqS74ytkCN4LpWc5Yvl3idPA0nJdVQgd0iRlFHusjwRyMH9fumBnGe0QU7PUG0Ty3--3evbuma00RNy9K5qmjJq9zWsdTXTsF7a6O-yqKb2TMPsa5ap9Xs67mO7jWgQCcZvt-805avNyz96b4Uw)

1.  **Host Machine:** Runs VirtualBox and Vagrant.
2.  **VM 1 (`oai-k8s-vm`):**
    *   **IP:** `172.17.42.15`
    *   **Purpose:** Hosts the RAN Kubernetes cluster.
    *   **Components:** OAI UE, DU, CU-CP, CU-UP pods.
3.  **VM 2 (`open5gs-k8s-vm`):**
    *   **IP:** `172.17.42.27`
    *   **Purpose:** Hosts the 5G Core Kubernetes cluster.
    *   **Components:** Open5GS AMF, UPF, SMF pods.
4.  **Networking:**
    *   **Inter-VM:** A private Vagrant network (`172.17.42.0/24`) for BGP peering.
    *   **Inter-Cluster:** Calico CNI with BGP will advertise pod routes between the clusters.
    *   **Application-Level:** OAI pods on the RAN cluster will connect to `NodePort` services on the Core cluster to reach the AMF (N2) and UPF (N3).

---

### **Part 0: Host Environment Setup**

If you haven't already, install the necessary tools on your main computer (Windows, macOS, or Linux).

1.  **Install VirtualBox:** Download and install from the [official VirtualBox website](https://www.virtualbox.org/wiki/Downloads).
2.  **Install Vagrant:** Download and install from the [official Vagrant website](https://www.vagrantup.com/downloads).

---

### **Part 1: Infrastructure Provisioning with Vagrant**

This replaces all manual VM creation steps.

1.  **Create a Project Directory:**
    ```bash
    mkdir 5g-k8s-lab
    cd 5g-k8s-lab
    ```

2.  Install disksize plugin
    ```bash
    vagrant plugin install vagrant-disksize
    ```

3. **Create the `Vagrantfile`:**
    This file defines your entire two-VM setup. Create a file named `Vagrantfile` and paste the following content.

```ruby
# Vagrantfile

# This line defines the required plugin. Vagrant will prompt the user to install it
# if it's not already present.
Vagrant.require_plugin "vagrant-disksize"

Vagrant.configure("2") do |config|

  # --- Global Settings ---

  # Use the official, minimal Ubuntu 20.04 server image.
  config.vm.box = "ubuntu/focal64"

  # Disable default folder sharing to improve performance and avoid permission issues.
  config.vm.synced_folder ".", "/vagrant", disabled: true

  # Set the default disk size for all VMs defined in this file.
  config.disksize.size = '50GB'

  # --- VM Definitions ---

  # Define the OAI RAN VM (oai-k8s-vm)
  config.vm.define "oai-k8s-vm" do |oai_vm|
    oai_vm.vm.hostname = "oai-k8s-vm"
    # Assign the static IP from your network diagram.
    oai_vm.vm.network "private_network", ip: "172.17.42.15"

    # Allocate resources
    oai_vm.vm.provider "virtualbox" do |vb|
      vb.memory = "6144" # 6GB RAM
      vb.cpus = "4"
      vb.name = "oai-k8s-vm" # Set a clear name in the VirtualBox GUI
    end
  end

  # Define the Open5GS Core VM (open5gs-k8s-vm)
  config.vm.define "open5gs-k8s-vm" do |core_vm|
    core_vm.vm.hostname = "open5gs-k8s-vm"
    # Assign the static IP from your network diagram.
    core_vm.vm.network "private_network", ip: "172.17.42.27"

    # Allocate resources
    core_vm.vm.provider "virtualbox" do |vb|
      vb.memory = "4096" # 4GB RAM
      vb.cpus = "2"
      vb.name = "open5gs-k8s-vm" # Set a clear name in the VirtualBox GUI
    end
  end
end
```

3.  **Launch the Environment:**
    From your terminal inside the `5g-k8s-lab` directory, run:
    ```bash
    vagrant up
    ```
    Vagrant will now create and configure both VMs automatically. Once it's done, you'll have two running, lightweight, CLI-only servers.

4.  **Access Your VMs:**
    You can now SSH into each machine with a simple command:
    ```bash
    # SSH into the OAI RAN machine
    vagrant ssh oai-k8s-vm

    # SSH into the Open5GS Core machine
    vagrant ssh open5gs-k8s-vm
    ```

5. **Ping test the connection:**
```bash
# From oai-k8s-vm
ping 172.17.42.27

# From open5gs-k8s-vm
ping 172.17.42.15

```
---

### **Part 2: Kubernetes and Calico BGP Setup**

The following steps must be performed **inside each VM** after you `vagrant ssh` into them.

#### **Section 2.1: Common Kubernetes Prerequisites (Run on BOTH VMs)**

```ruby
# Disable swap
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# Enable kernel modules and sysctl settings for K8s networking
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system

# Install containerd (container runtime)
sudo apt-get update
sudo apt-get install -y containerd
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
sudo systemctl restart containerd

containerd --version 
sudo systemctl status containerd

# Install Kubernetes components
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable --now kubelet

kubelet --version
kubeadm version
kubectl version --client

```

#### **Section 2.2: Initialize Kubernetes Clusters**

**On `oai-k8s-vm` (172.17.42.15):**
```bash
sudo kubeadm init \
--pod-network-cidr 10.20.0.0/16 \
--service-cidr 10.21.0.0/16 \
--apiserver-advertise-address 172.17.42.15
```

**On `open5gs-k8s-vm` (172.17.42.27):**
```bash
sudo kubeadm init \
--pod-network-cidr 10.30.0.0/16 \
--service-cidr 10.31.0.0/16 \
--apiserver-advertise-address 172.17.42.27
```

**After `kubeadm init` on BOTH VMs, run these commands:**
```bash
# Configure kubectl access for the vagrant user
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Untaint the control-plane node to allow scheduling pods on it
kubectl taint nodes $(kubectl get nodes -o jsonpath='{.items[0].metadata.name}') node-role.kubernetes.io/control-plane-
```

#### **Section 2.3: Install Calico with BGP (Run on BOTH VMs)**

```bash
# Install the Tigera Operator
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml

# Download the calicoctl CLI
curl -L https://github.com/projectcalico/calico/releases/download/v3.26.1/calicoctl-linux-amd64 -o calicoctl
chmod +x ./calicoctl
sudo mv ./calicoctl /usr/local/bin/
```

**On `oai-k8s-vm` (172.17.42.15):**
Create `calico-install.yaml` with the correct Pod CIDR.
```yaml
# calico-install.yaml
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  calicoNetwork:
    ipPools:
    - blockSize: 26
      cidr: 10.20.0.0/16  # CIDR for OAI RAN cluster
      encapsulation: None
      natOutgoing: Enabled
      nodeSelector: all()
```
Apply it: `kubectl apply -f calico-install.yaml`

**On `open5gs-k8s-vm` (172.17.42.27):**
Create `calico-install.yaml` with the correct Pod CIDR.
```yaml
# calico-install.yaml
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  calicoNetwork:
    ipPools:
    - blockSize: 26
      cidr: 10.30.0.0/16 # CIDR for Open5GS Core cluster
      encapsulation: None
      natOutgoing: Enabled
      nodeSelector: all()
```
Apply it: `kubectl apply -f calico-install.yaml`

#### **Section 2.4: Configure BGP Peering**

**On `oai-k8s-vm` (172.17.42.15):**
Create `bgp-peer.yaml` to peer with the core cluster.
```yaml
# bgp-peer.yaml
apiVersion: projectcalico.org/v3
kind: BGPConfiguration
metadata:
  name: default
spec:
  asNumber: 64512 # OAI RAN Cluster ASN
  nodeToNodeMeshEnabled: false
---
apiVersion: projectcalico.org/v3
kind: BGPPeer
metadata:
  name: peer-to-core-cluster
spec:
  peerIP: 172.17.42.27   # IP of open5gs-k8s-vm
  asNumber: 64513         # ASN of Core Cluster
```
Apply it:  
```
# sudo calicoctl apply -f bgp-peer.yaml
# giving error  below is the sol.

export KUBECONFIG=/etc/kubernetes/admin.conf
sudo -E calicoctl apply -f bgp-peer.yaml

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
unset KUBECONFIG

```

**On `open5gs-k8s-vm` (172.17.42.27):**
Create `bgp-peer.yaml` to peer with the RAN cluster.
```yaml
# bgp-peer.yaml
apiVersion: projectcalico.org/v3
kind: BGPConfiguration
metadata:
  name: default
spec:
  asNumber: 64513 # Open5GS Core Cluster ASN
  nodeToNodeMeshEnabled: false
---
apiVersion: projectcalico.org/v3
kind: BGPPeer
metadata:
  name: peer-to-ran-cluster
spec:
  peerIP: 172.17.42.15   # IP of oai-k8s-vm
  asNumber: 64512         # ASN of RAN Cluster
```
Apply it: 
```
# sudo calicoctl apply -f bgp-peer.yaml
# giving error  below is the sol.

export KUBECONFIG=/etc/kubernetes/admin.conf
sudo -E calicoctl apply -f bgp-peer.yaml

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
unset KUBECONFIG
```


#### **Section 2.5: Verify BGP Peering**

On either VM, check the status.
```bash
sudo calicoctl node status
```
The output should show the peer as **`Established`**. If not, check your IPs, ASNs, and that there are no firewalls (`sudo ufw status`).

Your lightweight, fully provisioned Kubernetes infrastructure is now ready! You can now proceed with deploying the 5G applications. The next steps will be identical to the "Part 1: Deploy Open5GS" and "Part 2: Deploy OAI" sections from the previous guide, as they deal with Kubernetes manifests, which are independent of how the underlying VMs were created.


#### **Section 2.6: Test Pod-to-Pod Communication**

This is the ultimate test.

1.  **On `k8s-master-1`,** create a test pod:
    ```bash
    kubectl run nginx-c1 --image=nginx
    ```
    Wait for it to be running and get its IP:
    ```bash
    kubectl get pod nginx-c1 -o wide
    # Note the IP address, it will be something like 10.20.x.x
    ```
    Let's say the IP is `10.20.127.6`.

2.  **On `k8s-master-2`,** create a temporary pod to use for testing:
    ```bash
    kubectl run busybox-c2 --image=busybox -it --rm -- /bin/sh
    ```
    This will drop you into a shell inside the pod on Cluster 2.

3.  **From inside the `busybox-c2` pod,** ping the `nginx-c1` pod on the other cluster:
    ```sh
    # Inside the busybox shell
    ping 10.20.127.6
    ```

If you see ping replies, you have successfully configured cross-cluster networking with Calico and BGP! You have direct, non-encapsulated routing between pods in separate Kubernetes clusters.

```bash
vagrant@oai-k8s-vm:~$ kubectl get pods -o wide
NAME       READY   STATUS    RESTARTS   AGE     IP            NODE         NOMINATED NODE   READINESS GATES   
nginx-c1   1/1     Running   0          2m42s   10.20.127.6   oai-k8s-vm   <none>           <none>
```

```bash
vagrant@open5gs-k8s-vm:~$ kubectl get pod -o wide 
NAME       READY   STATUS    RESTARTS   AGE     IP             NODE             NOMINATED NODE   READINESS GATES
nginx-c1   1/1     Running   0          2m21s   10.30.33.134   open5gs-k8s-vm   <none>           <none>
vagrant@open5gs-k8s-vm:~$ kubectl run busybox-c2 --image=busybox -it --rm -- /bin/sh
If you dont see a command prompt, try pressing enter.
/ # ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: seq=0 ttl=254 time=66.350 ms
64 bytes from 8.8.8.8: seq=1 ttl=254 time=51.251 ms
64 bytes from 8.8.8.8: seq=2 ttl=254 time=49.880 ms
^C
--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 49.880/55.827/66.350 ms
/ # 
/ # ping 10.30.33.134
PING 10.30.33.134 (10.30.33.134): 56 data bytes
64 bytes from 10.30.33.134: seq=0 ttl=63 time=2.634 ms
64 bytes from 10.30.33.134: seq=1 ttl=63 time=27.587 ms
^C
--- 10.30.33.134 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 2.634/15.110/27.587 ms
/ #
/ # ping 10.30.33.134
PING 10.30.33.134 (10.30.33.134): 56 data bytes
64 bytes from 10.30.33.134: seq=0 ttl=63 time=0.158 ms
64 bytes from 10.30.33.134: seq=1 ttl=63 time=0.151 ms
64 bytes from 10.30.33.134: seq=2 ttl=63 time=0.115 ms
^C
--- 10.30.33.134 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.115/0.141/0.158 ms
/ #
/ #
/ # ping 10.20.127.6
PING 10.20.127.6 (10.20.127.6): 56 data bytes
64 bytes from 10.20.127.6: seq=0 ttl=62 time=18.653 ms
64 bytes from 10.20.127.6: seq=1 ttl=62 time=3.965 ms       
64 bytes from 10.20.127.6: seq=2 ttl=62 time=6.511 ms       
64 bytes from 10.20.127.6: seq=3 ttl=62 time=3.715 ms       
^C
--- 10.20.127.6 ping statistics ---
64 bytes from 10.20.127.6: seq=1 ttl=62 time=3.965 ms      
64 bytes from 10.20.127.6: seq=2 ttl=62 time=6.511 ms      
64 bytes from 10.20.127.6: seq=3 ttl=62 time=3.715 ms      
^C
--- 10.20.127.6 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss  
round-trip min/avg/max = 3.715/8.211/18.653 ms
/ # exit
pod "busybox-c2" deleted
```

---
## Logs:

### Vagrant setup logs :

```ruby
PS D:\5g-k8s-lab> vagrant up
require_plugin is deprecated and has no effect any longer.
Use `vagrant plugin` commands to manage plugins. This warning will
be removed in the next version of Vagrant.
Bringing machine 'oai-k8s-vm' up with 'virtualbox' provider...
Bringing machine 'open5gs-k8s-vm' up with 'virtualbox' provider...
==> oai-k8s-vm: Box 'ubuntu/focal64' could not be found. Attempting to find and install...
    oai-k8s-vm: Box Provider: virtualbox
    oai-k8s-vm: Box Version: >= 0
==> oai-k8s-vm: Loading metadata for box 'ubuntu/focal64'
    oai-k8s-vm: URL: https://vagrantcloud.com/api/v2/vagrant/ubuntu/focal64
==> oai-k8s-vm: Adding box 'ubuntu/focal64' (v20240821.0.1) for provider: virtualbox
    oai-k8s-vm: Downloading: https://vagrantcloud.com/ubuntu/boxes/focal64/versions/20240821.0.1/providers/virtualbox/unknown/vagrant.box
    oai-k8s-vm:
==> oai-k8s-vm: Successfully added box 'ubuntu/focal64' (v20240821.0.1) for 'virtualbox'!
==> oai-k8s-vm: Importing base box 'ubuntu/focal64'...
==> oai-k8s-vm: Matching MAC address for NAT networking...
==> oai-k8s-vm: Checking if box 'ubuntu/focal64' version '20240821.0.1' is up to date...
A VirtualBox machine with the name 'oai-k8s-vm' already exists.
Please use another name or delete the machine with the existing
name, and try again.
PS D:\5g-k8s-lab> vagrant up
require_plugin is deprecated and has no effect any longer.
Use `vagrant plugin` commands to manage plugins. This warning will
be removed in the next version of Vagrant.
Bringing machine 'oai-k8s-vm' up with 'virtualbox' provider...
Bringing machine 'open5gs-k8s-vm' up with 'virtualbox' provider...
==> oai-k8s-vm: Checking if box 'ubuntu/focal64' version '20240821.0.1' is up to date...
==> oai-k8s-vm: Setting the name of the VM: oai-k8s-vm
==> oai-k8s-vm: Clearing any previously set network interfaces...
==> oai-k8s-vm: Preparing network interfaces based on configuration...
    oai-k8s-vm: Adapter 1: nat
    oai-k8s-vm: Adapter 2: hostonly
==> oai-k8s-vm: Forwarding ports...
    oai-k8s-vm: 22 (guest) => 2222 (host) (adapter 1)
==> oai-k8s-vm: Running 'pre-boot' VM customizations...
There was an error while executing `VBoxManage`, a CLI used by Vagrant
for controlling VirtualBox. The command and stderr is shown below.

Command: ["clonemedium", "C:\\\\Users\\\\vishalkumar.shaw\\\\VirtualBox VMs\\\\oai-k8s-vm\\\\ubuntu-focal-20.04-cloudimg.vmdk", "C:\\Users\\\\vishalkumar.shaw\\\\VirtualBox VMs\\\\oai-k8s-vm/ubuntu-focal-20.04-cloudimg.vdi", "--format", "VDI"]

Stderr: 0%...VBOX_E_FILE_ERROR
VBoxManage.exe: error: Failed to clone medium
VBoxManage.exe: error: Could not create the clone medium 'C:\Users\vishalkumar.shaw\VirtualBox VMs\oai-k8s-vm\ubuntu-focal-20.04-cloudimg.vdi' (VERR_DISK_FULL)
VBoxManage.exe: error: Details: code VBOX_E_FILE_ERROR (0x80bb0004), component MediumWrap, interface IMedium
VBoxManage.exe: error: Context: "enum RTEXITCODE __cdecl handleCloneMedium(struct HandlerArg *)" at line 1208 of file VBoxManageDisk.cpp    
PS D:\5g-k8s-lab> vagrant destroy
require_plugin is deprecated and has no effect any longer.
Use `vagrant plugin` commands to manage plugins. This warning will
be removed in the next version of Vagrant.
==> open5gs-k8s-vm: VM not created. Moving on...
==> oai-k8s-vm: VM not created. Moving on...
PS D:\5g-k8s-lab> 
PS D:\5g-k8s-lab> vagrant up     
require_plugin is deprecated and has no effect any longer.
Use `vagrant plugin` commands to manage plugins. This warning will
be removed in the next version of Vagrant.
Bringing machine 'oai-k8s-vm' up with 'virtualbox' provider...
Bringing machine 'open5gs-k8s-vm' up with 'virtualbox' provider...
==> oai-k8s-vm: Importing base box 'ubuntu/focal64'...
==> oai-k8s-vm: Matching MAC address for NAT networking...
==> oai-k8s-vm: Checking if box 'ubuntu/focal64' version '20240821.0.1' is up to date...
==> oai-k8s-vm: Setting the name of the VM: oai-k8s-vm
The name of your virtual machine couldn't be set because VirtualBox
is reporting another VM with that name already exists. Most of the
time, this is because of an error with VirtualBox not cleaning up
properly. To fix this, verify that no VMs with that name do exist
(by opening the VirtualBox GUI). If they don't, then look at the
folder in the error message from VirtualBox below and remove it
if there isn't any information you need in there.

VirtualBox error:

VBoxManage.exe: error: Could not rename the directory 'D:\VirtualBox VMs\ubuntu-focal-20.04-cloudimg-20250625_1751875708920_77624' to 'D:\VirtualBox VMs\oai-k8s-vm' to save the settings file (VERR_ALREADY_EXISTS)
VBoxManage.exe: error: Details: code E_FAIL (0x80004005), component SessionMachine, interface IMachine, callee IUnknown
VBoxManage.exe: error: Context: "SaveSettings()" at line 3772 of file VBoxManageModifyVM.cpp
PS D:\5g-k8s-lab> 
PS D:\5g-k8s-lab>
PS D:\5g-k8s-lab> vagrant destroy -f
require_plugin is deprecated and has no effect any longer.
Use `vagrant plugin` commands to manage plugins. This warning will
be removed in the next version of Vagrant.
==> open5gs-k8s-vm: VM not created. Moving on...
==> oai-k8s-vm: Destroying VM and associated drives...
PS D:\5g-k8s-lab> vagrant global-status --prune
require_plugin is deprecated and has no effect any longer.
Use `vagrant plugin` commands to manage plugins. This warning will
be removed in the next version of Vagrant.
id       name   provider state  directory
--------------------------------------------------------------------
There are no active Vagrant environments on this computer! Or,
you haven't destroyed and recreated Vagrant environments that were
started with an older version of Vagrant.
PS D:\5g-k8s-lab> Remove-Item -Recurse -Force .vagrant
PS D:\5g-k8s-lab> 
PS D:\5g-k8s-lab>
PS D:\5g-k8s-lab> vagrant up
require_plugin is deprecated and has no effect any longer.
Use `vagrant plugin` commands to manage plugins. This warning will
be removed in the next version of Vagrant.
Bringing machine 'oai-k8s-vm' up with 'virtualbox' provider...
Bringing machine 'open5gs-k8s-vm' up with 'virtualbox' provider...
==> oai-k8s-vm: Importing base box 'ubuntu/focal64'...
==> oai-k8s-vm: Matching MAC address for NAT networking...
==> oai-k8s-vm: Checking if box 'ubuntu/focal64' version '20240821.0.1' is up to date...
==> oai-k8s-vm: Setting the name of the VM: oai-k8s-vm
==> oai-k8s-vm: Clearing any previously set network interfaces...
==> oai-k8s-vm: Preparing network interfaces based on configuration...
    oai-k8s-vm: Adapter 1: nat
    oai-k8s-vm: Adapter 2: hostonly
==> oai-k8s-vm: Forwarding ports...
    oai-k8s-vm: 22 (guest) => 2222 (host) (adapter 1)
==> oai-k8s-vm: Running 'pre-boot' VM customizations...
==> oai-k8s-vm: Resized disk: old 40960 MB, req 51200 MB, new 51200 MB
==> oai-k8s-vm: You may need to resize the filesystem from within the guest.
==> oai-k8s-vm: Booting VM...
==> oai-k8s-vm: Waiting for machine to boot. This may take a few minutes...
    oai-k8s-vm: SSH address: 127.0.0.1:2222
    oai-k8s-vm: SSH username: vagrant
    oai-k8s-vm: SSH auth method: private key
    oai-k8s-vm: Warning: Connection reset. Retrying...
    oai-k8s-vm: Warning: Connection aborted. Retrying...
    oai-k8s-vm: 
    oai-k8s-vm: Vagrant insecure key detected. Vagrant will automatically replace
    oai-k8s-vm: this with a newly generated keypair for better security.
    oai-k8s-vm: 
    oai-k8s-vm: Inserting generated public key within guest...
    oai-k8s-vm: Removing insecure key from the guest if it's present...
    oai-k8s-vm: Key inserted! Disconnecting and reconnecting using new SSH key...
==> oai-k8s-vm: Machine booted and ready!
==> oai-k8s-vm: Checking for guest additions in VM...
    oai-k8s-vm: The guest additions on this VM do not match the installed version of
    oai-k8s-vm: VirtualBox! In most cases this is fine, but in rare cases it can
    oai-k8s-vm: prevent things such as shared folders from working properly. If you see
    oai-k8s-vm: shared folder errors, please make sure the guest additions within the
    oai-k8s-vm: virtual machine match the version of VirtualBox you have installed on
    oai-k8s-vm: your host and reload your VM.
    oai-k8s-vm:
    oai-k8s-vm: Guest Additions Version: 6.1.50
    oai-k8s-vm: VirtualBox Version: 7.1
==> oai-k8s-vm: Setting hostname...
==> oai-k8s-vm: Configuring and enabling network interfaces...
==> open5gs-k8s-vm: Importing base box 'ubuntu/focal64'...
==> open5gs-k8s-vm: Matching MAC address for NAT networking...
==> open5gs-k8s-vm: Checking if box 'ubuntu/focal64' version '20240821.0.1' is up to date...
==> open5gs-k8s-vm: Setting the name of the VM: open5gs-k8s-vm
==> open5gs-k8s-vm: Fixed port collision for 22 => 2222. Now on port 2200.
==> open5gs-k8s-vm: Clearing any previously set network interfaces...
==> open5gs-k8s-vm: Preparing network interfaces based on configuration...
    open5gs-k8s-vm: Adapter 1: nat
    open5gs-k8s-vm: Adapter 2: hostonly
==> open5gs-k8s-vm: Forwarding ports...
    open5gs-k8s-vm: 22 (guest) => 2200 (host) (adapter 1)
==> open5gs-k8s-vm: Running 'pre-boot' VM customizations...
==> open5gs-k8s-vm: Resized disk: old 40960 MB, req 51200 MB, new 51200 MB
==> open5gs-k8s-vm: You may need to resize the filesystem from within the guest.
==> open5gs-k8s-vm: Booting VM...
==> open5gs-k8s-vm: Waiting for machine to boot. This may take a few minutes...
    open5gs-k8s-vm: SSH address: 127.0.0.1:2200
    open5gs-k8s-vm: SSH username: vagrant
    open5gs-k8s-vm: SSH auth method: private key
    open5gs-k8s-vm: Warning: Connection reset. Retrying...
    open5gs-k8s-vm: Warning: Connection aborted. Retrying...
    open5gs-k8s-vm: 
    open5gs-k8s-vm: Vagrant insecure key detected. Vagrant will automatically replace
    open5gs-k8s-vm: this with a newly generated keypair for better security.
    open5gs-k8s-vm: 
    open5gs-k8s-vm: Inserting generated public key within guest...
    open5gs-k8s-vm: Removing insecure key from the guest if it's present...
    open5gs-k8s-vm: Key inserted! Disconnecting and reconnecting using new SSH key...
==> open5gs-k8s-vm: Machine booted and ready!
==> open5gs-k8s-vm: Checking for guest additions in VM...
    open5gs-k8s-vm: The guest additions on this VM do not match the installed version of
    open5gs-k8s-vm: VirtualBox! In most cases this is fine, but in rare cases it can
    open5gs-k8s-vm: prevent things such as shared folders from working properly. If you see
    open5gs-k8s-vm: shared folder errors, please make sure the guest additions within the
    open5gs-k8s-vm: virtual machine match the version of VirtualBox you have installed on
    open5gs-k8s-vm: your host and reload your VM.
    open5gs-k8s-vm:
    open5gs-k8s-vm: Guest Additions Version: 6.1.50
    open5gs-k8s-vm: VirtualBox Version: 7.1
==> open5gs-k8s-vm: Setting hostname...
==> open5gs-k8s-vm: Configuring and enabling network interfaces...
PS D:\5g-k8s-lab> 
    open5gs-k8s-vm:
    open5gs-k8s-vm: Vagrant insecure key detected. Vagrant will automatically replace
    open5gs-k8s-vm: this with a newly generated keypair for better security.
    open5gs-k8s-vm:
    open5gs-k8s-vm: Inserting generated public key within guest...
    open5gs-k8s-vm: Removing insecure key from the guest if it's present...
    open5gs-k8s-vm: Key inserted! Disconnecting and reconnecting using new SSH key...
==> open5gs-k8s-vm: Machine booted and ready!
==> open5gs-k8s-vm: Checking for guest additions in VM...
    open5gs-k8s-vm: The guest additions on this VM do not match the installed version of
    open5gs-k8s-vm: VirtualBox! In most cases this is fine, but in rare cases it can
    open5gs-k8s-vm: prevent things such as shared folders from working properly. If you see
    open5gs-k8s-vm: shared folder errors, please make sure the guest additions within the
    open5gs-k8s-vm: virtual machine match the version of VirtualBox you have installed on
    open5gs-k8s-vm: your host and reload your VM.
    open5gs-k8s-vm:
    open5gs-k8s-vm: Guest Additions Version: 6.1.50
    open5gs-k8s-vm: VirtualBox Version: 7.1
==> open5gs-k8s-vm: Setting hostname...
==> open5gs-k8s-vm: Configuring and enabling network interfaces...
PS D:\5g-k8s-lab>
    open5gs-k8s-vm: virtual machine match the version of VirtualBox you have installed on
    open5gs-k8s-vm: your host and reload your VM.
    open5gs-k8s-vm:
    open5gs-k8s-vm: Guest Additions Version: 6.1.50
    open5gs-k8s-vm: VirtualBox Version: 7.1
==> open5gs-k8s-vm: Setting hostname...
==> open5gs-k8s-vm: Configuring and enabling network interfaces...
PS D:\5g-k8s-lab>
PS D:\5g-k8s-lab>
PS D:\5g-k8s-lab>
```

### oai-k8s-vm calico bgp setup logs
```ruby
 [104 kB]
Get:16 http://archive.ubuntu.com/ubuntu focal/multiverse amd64 c-n-f Metadata [9136 B]
Get:17 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 Packages [3954 kB]
Get:18 http://archive.ubuntu.com/ubuntu focal-updates/universe amd64 Packages [1262 kB]
Get:19 http://archive.ubuntu.com/ubuntu focal-updates/universe Translation-en [303 kB]
Get:20 http://archive.ubuntu.com/ubuntu focal-updates/universe amd64 c-n-f Metadata [29.3 kB]
Get:21 http://archive.ubuntu.com/ubuntu focal-updates/multiverse amd64 Packages [29.7 kB]
Get:22 http://archive.ubuntu.com/ubuntu focal-updates/multiverse Translation-en [8316 B]
Get:23 http://archive.ubuntu.com/ubuntu focal-updates/multiverse amd64 c-n-f Metadata [688 B]
Get:24 http://archive.ubuntu.com/ubuntu focal-backports/main amd64 Packages [45.7 kB]
Get:25 http://archive.ubuntu.com/ubuntu focal-backports/main Translation-en [16.3 kB]
Get:26 http://archive.ubuntu.com/ubuntu focal-backports/main amd64 c-n-f Metadata [1420 B]
Get:27 http://archive.ubuntu.com/ubuntu focal-backports/restricted amd64 c-n-f Metadata [116 B]
Get:28 http://archive.ubuntu.com/ubuntu focal-backports/universe amd64 Packages [25.0 kB]
Get:29 http://archive.ubuntu.com/ubuntu focal-backports/universe Translation-en [16.3 kB]
Get:30 http://archive.ubuntu.com/ubuntu focal-backports/universe amd64 c-n-f Metadata [880 B]
Get:31 http://archive.ubuntu.com/ubuntu focal-backports/multiverse amd64 c-n-f Metadata [116 B]
Fetched 21.7 MB in 12s (1808 kB/s)                                     
Reading package lists... Done
vagrant@oai-k8s-vm:~$ sudo apt-get install -y containerd
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  runc
The following NEW packages will be installed:
  containerd runc
0 upgraded, 2 newly installed, 0 to remove and 0 not upgraded.
Need to get 41.7 MB of archives.
After this operation, 176 MB of additional disk space will be used.    
Get:1 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 runc amd64 1.1.12-0ubuntu2~20.04.1 [8066 kB]
Get:2 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 containerd amd64 1.7.24-0ubuntu1~20.04.2 [33.6 MB]
Fetched 41.7 MB in 11s (3784 kB/s)                                     
Selecting previously unselected package runc.
(Reading database ... 63917 files and directories currently installed.)
Preparing to unpack .../runc_1.1.12-0ubuntu2~20.04.1_amd64.deb ...
Unpacking runc (1.1.12-0ubuntu2~20.04.1) ...
Selecting previously unselected package containerd.
Preparing to unpack .../containerd_1.7.24-0ubuntu1~20.04.2_amd64.deb ...
Unpacking containerd (1.7.24-0ubuntu1~20.04.2) ...
Setting up runc (1.1.12-0ubuntu2~20.04.1) ...
Setting up containerd (1.7.24-0ubuntu1~20.04.2) ...
Created symlink /etc/systemd/system/multi-user.target.wants/containerd.service → /lib/systemd/system/containerd.service.
Processing triggers for man-db (2.9.1-1) ...
vagrant@oai-k8s-vm:~$ sudo mkdir -p /etc/containerd
 /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
sudo systemctl restartvagrant@oai-k8s-vm:~$ sudo containerd config default | sudo tee /etc/containerd/config.toml
 containerddisabled_plugins = []
imports = []
oom_score = 0
plugin_dir = ""
required_plugins = []
root = "/var/lib/containerd"
state = "/run/containerd"
temp = ""
version = 2

[cgroup]
  path = ""

[debug]
  address = ""
  format = ""
  gid = 0
  level = ""
  uid = 0

[grpc]
  address = "/run/containerd/containerd.sock"
  gid = 0
  max_recv_message_size = 16777216
  max_send_message_size = 16777216
  tcp_address = ""
  tcp_tls_ca = ""
  tcp_tls_cert = ""
  tcp_tls_key = ""
  uid = 0

[metrics]
  address = ""
  grpc_histogram = false

[plugins]

  [plugins."io.containerd.gc.v1.scheduler"]
    deletion_threshold = 0
    mutation_threshold = 100
    pause_threshold = 0.02
    schedule_delay = "0s"
    startup_delay = "100ms"

  [plugins."io.containerd.grpc.v1.cri"]
    cdi_spec_dirs = ["/etc/cdi", "/var/run/cdi"]
    device_ownership_from_security_context = false
    disable_apparmor = false
    disable_cgroup = false
    disable_hugetlb_controller = true
    disable_proc_mount = false
    disable_tcp_service = true
    drain_exec_sync_io_timeout = "0s"
    enable_cdi = false
    enable_selinux = false
    enable_tls_streaming = false
    enable_unprivileged_icmp = false
    enable_unprivileged_ports = false
    ignore_deprecation_warnings = []
    ignore_image_defined_volumes = false
    image_pull_progress_timeout = "5m0s"
    image_pull_with_sync_fs = false
    max_concurrent_downloads = 3
    max_container_log_line_size = 16384
    netns_mounts_under_state_dir = false
    restrict_oom_score_adj = false
    sandbox_image = "registry.k8s.io/pause:3.8"
    selinux_category_range = 1024
    stats_collect_period = 10
    stream_idle_timeout = "4h0m0s"
    stream_server_address = "127.0.0.1"
    stream_server_port = "0"
    systemd_cgroup = false
    tolerate_missing_hugetlb_controller = true
    unset_seccomp_profile = ""

    [plugins."io.containerd.grpc.v1.cri".cni]
      bin_dir = "/opt/cni/bin"
      conf_dir = "/etc/cni/net.d"
      conf_template = ""
      ip_pref = ""
      max_conf_num = 1
      setup_serially = false

    [plugins."io.containerd.grpc.v1.cri".containerd]
      default_runtime_name = "runc"
      disable_snapshot_annotations = true
      discard_unpacked_layers = false
      ignore_blockio_not_enabled_errors = false
      ignore_rdt_not_enabled_errors = false
      no_pivot = false
      snapshotter = "overlayfs"

      [plugins."io.containerd.grpc.v1.cri".containerd.default_runtime] 
        base_runtime_spec = ""
        cni_conf_dir = ""
        cni_max_conf_num = 0
        container_annotations = []
        pod_annotations = []
        privileged_without_host_devices = false
        privileged_without_host_devices_all_devices_allowed = false    
        runtime_engine = ""
        runtime_path = ""
        runtime_root = ""
        runtime_type = ""
        sandbox_mode = ""
        snapshotter = ""

        [plugins."io.containerd.grpc.v1.cri".containerd.default_runtime.options]

      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]        

        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc] 
          base_runtime_spec = ""
          cni_conf_dir = ""
          cni_max_conf_num = 0
          container_annotations = []
          pod_annotations = []
          privileged_without_host_devices = false
          privileged_without_host_devices_all_devices_allowed = false  
          runtime_engine = ""
          runtime_path = ""
          runtime_root = ""
          runtime_type = "io.containerd.runc.v2"
          sandbox_mode = "podsandbox"
          snapshotter = ""

          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            BinaryName = ""
            CriuImagePath = ""
            CriuPath = ""
            CriuWorkPath = ""
            IoGid = 0
            IoUid = 0
            NoNewKeyring = false
            NoPivotRoot = false
            Root = ""
            ShimCgroup = ""
            SystemdCgroup = false

      [plugins."io.containerd.grpc.v1.cri".containerd.untrusted_workload_runtime]
        base_runtime_spec = ""
        cni_conf_dir = ""
        cni_max_conf_num = 0
        container_annotations = []
        pod_annotations = []
        privileged_without_host_devices = false
        privileged_without_host_devices_all_devices_allowed = false    
        runtime_engine = ""
        runtime_path = ""
        runtime_root = ""
        runtime_type = ""
        sandbox_mode = ""
        snapshotter = ""

        [plugins."io.containerd.grpc.v1.cri".containerd.untrusted_workload_runtime.options]

    [plugins."io.containerd.grpc.v1.cri".image_decryption]
      key_model = "node"

    [plugins."io.containerd.grpc.v1.cri".registry]
      config_path = ""

      [plugins."io.containerd.grpc.v1.cri".registry.auths]

      [plugins."io.containerd.grpc.v1.cri".registry.configs]

      [plugins."io.containerd.grpc.v1.cri".registry.headers]

      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]

    [plugins."io.containerd.grpc.v1.cri".x509_key_pair_streaming]      
      tls_cert_file = ""
      tls_key_file = ""

  [plugins."io.containerd.internal.v1.opt"]
    path = "/opt/containerd"

  [plugins."io.containerd.internal.v1.restart"]
    interval = "10s"

  [plugins."io.containerd.internal.v1.tracing"]

  [plugins."io.containerd.metadata.v1.bolt"]
    content_sharing_policy = "shared"

  [plugins."io.containerd.monitor.v1.cgroups"]
    no_prometheus = false

  [plugins."io.containerd.nri.v1.nri"]
    disable = true
    disable_connections = false
    plugin_config_path = "/etc/nri/conf.d"
    plugin_path = "/opt/nri/plugins"
    plugin_registration_timeout = "5s"
    plugin_request_timeout = "2s"
    socket_path = "/var/run/nri/nri.sock"

  [plugins."io.containerd.runtime.v1.linux"]
    no_shim = false
    runtime = "runc"
    runtime_root = ""
    shim = "containerd-shim"
    shim_debug = false

  [plugins."io.containerd.runtime.v2.task"]
    platforms = ["linux/amd64"]
    sched_core = false

  [plugins."io.containerd.service.v1.diff-service"]
    default = ["walking"]

  [plugins."io.containerd.service.v1.tasks-service"]
    blockio_config_file = ""
    rdt_config_file = ""

  [plugins."io.containerd.snapshotter.v1.aufs"]
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.blockfile"]
    fs_type = ""
    mount_options = []
    root_path = ""
    scratch_file = ""

  [plugins."io.containerd.snapshotter.v1.btrfs"]
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.devmapper"]
    async_remove = false
    base_image_size = ""
    discard_blocks = false
    fs_options = ""
    fs_type = ""
    pool_name = ""
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.native"]
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.overlayfs"]
    mount_options = []
    root_path = ""
    sync_remove = false
    upperdir_label = false

  [plugins."io.containerd.snapshotter.v1.zfs"]
    root_path = ""

  [plugins."io.containerd.tracing.processor.v1.otlp"]

  [plugins."io.containerd.transfer.v1.local"]
    config_path = ""
    max_concurrent_downloads = 3
    max_concurrent_uploaded_layers = 3

    [[plugins."io.containerd.transfer.v1.local".unpack_config]]        
      differ = ""
      platform = "linux/amd64"
      snapshotter = "overlayfs"

[proxy_plugins]

[stream_processors]

  [stream_processors."io.containerd.ocicrypt.decoder.v1.tar"]
    accepts = ["application/vnd.oci.image.layer.v1.tar+encrypted"]     
    args = ["--decryption-keys-path", "/etc/containerd/ocicrypt/keys"] 
    env = ["OCICRYPT_KEYPROVIDER_CONFIG=/etc/containerd/ocicrypt/ocicrypt_keyprovider.conf"]
    path = "ctd-decoder"
    returns = "application/vnd.oci.image.layer.v1.tar"

  [stream_processors."io.containerd.ocicrypt.decoder.v1.tar.gzip"]     
    accepts = ["application/vnd.oci.image.layer.v1.tar+gzip+encrypted"]
    args = ["--decryption-keys-path", "/etc/containerd/ocicrypt/keys"] 
    env = ["OCICRYPT_KEYPROVIDER_CONFIG=/etc/containerd/ocicrypt/ocicrypt_keyprovider.conf"]
    path = "ctd-decoder"
    returns = "application/vnd.oci.image.layer.v1.tar+gzip"

[timeouts]
  "io.containerd.timeout.bolt.open" = "0s"
  "io.containerd.timeout.metrics.shimstats" = "2s"
  "io.containerd.timeout.shim.cleanup" = "5s"
  "io.containerd.timeout.shim.load" = "5s"
  "io.containerd.timeout.shim.shutdown" = "3s"
  "io.containerd.timeout.task.state" = "2s"

[ttrpc]
  address = ""
  gid = 0
  uid = 0
vagrant@oai-k8s-vm:~$ sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
vagrant@oai-k8s-vm:~$ sudo systemctl restart containerd
vagrant@oai-k8s-vm:~$ 
vagrant@oai-k8s-vm:~$
vagrant@oai-k8s-vm:~$ # Install Kubernetes components
vagrant@oai-k8s-vm:~$ sudo apt-get install -y apt-transport-https ca-certificates curl
Reading package lists... Done
Building dependency tree       
Reading state information... Done
ca-certificates is already the newest version (20240203~20.04.1).
ca-certificates set to manually installed.
curl is already the newest version (7.68.0-1ubuntu2.25).
curl set to manually installed.
The following NEW packages will be installed:
  apt-transport-https
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 1704 B of archives.
After this operation, 162 kB of additional disk space will be used.    
Get:1 http://archive.ubuntu.com/ubuntu focal-updates/universe amd64 apt-transport-https all 2.0.11 [1704 B]
Fetched 1704 B in 1s (2336 B/s)
Selecting previously unselected package apt-transport-https.
(Reading database ... 63981 files and directories currently installed.)
Preparing to unpack .../apt-transport-https_2.0.11_all.deb ...
Unpacking apt-transport-https (2.0.11) ...
Setting up apt-transport-https (2.0.11) ...
vagrant@oai-k8s-vm:~$ curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
tes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.listgpg: can't create '/etc/apt/keyrings/kubernetes-apt-keyring.gpg': No such file or directory   
gpg: no valid OpenPGP data found.
gpg: dearmoring failed: No such file or directory
(23) Failed writing body
vagrant@oai-k8s-vm:~$ 
vagrant@oai-k8s-vm:~$ 
vagrant@oai-k8s-vm:~$ curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
 [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.dgpg: can't create '/etc/apt/keyrings/kubernetes-apt-keyring.gpg': No such file or directory
gpg: no valid OpenPGP data found.
gpg: dearmoring failed: No such file or directory
/kubernetes.list(23) Failed writing body
vagrant@oai-k8s-vm:~$ echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /
vagrant@oai-k8s-vm:~$ 
vagrant@oai-k8s-vm:~$ 
vagrant@oai-k8s-vm:~$
vagrant@oai-k8s-vm:~$ sudo mkdir -p /etc/apt/keyrings
le:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/kevagrant@oai-k8s-vm:~$ curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
yrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.listvagrant@oai-k8s-vm:~$ echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /
vagrant@oai-k8s-vm:~$ 
vagrant@oai-k8s-vm:~$
vagrant@oai-k8s-vm:~$ sudo apt-get update
Hit:2 http://archive.ubuntu.com/ubuntu focal InRelease
Hit:3 http://security.ubuntu.com/ubuntu focal-security InRelease
Hit:4 http://archive.ubuntu.com/ubuntu focal-updates InRelease         
Get:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  InRelease [1192 B]
Hit:5 http://archive.ubuntu.com/ubuntu focal-backports InRelease
Get:6 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  Packages [20.3 kB]
Fetched 21.5 kB in 2s (10.2 kB/s)
Reading package lists... Done
vagrant@oai-k8s-vm:~$ sudo apt-get install -y kubelet kubeadm kubectl
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  conntrack cri-tools kubernetes-cni
Suggested packages:
  nftables
The following NEW packages will be installed:
  conntrack cri-tools kubeadm kubectl kubelet kubernetes-cni
0 upgraded, 6 newly installed, 0 to remove and 0 not upgraded.
Need to get 93.8 MB of archives.
After this operation, 343 MB of additional disk space will be used.    
Get:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  cri-tools 1.30.1-1.1 [21.3 MB]
Get:3 http://archive.ubuntu.com/ubuntu focal/main amd64 conntrack amd64 1:1.4.5-2 [30.3 kB]
Get:2 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  kubeadm 1.30.14-1.1 [10.5 MB]
Get:4 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  kubectl 1.30.14-1.1 [10.9 MB]
Get:5 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  kubernetes-cni 1.4.0-1.1 [32.9 MB]
Get:6 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  kubelet 1.30.14-1.1 [18.2 MB]
Fetched 93.8 MB in 9s (10.3 MB/s)                                      
Selecting previously unselected package conntrack.
(Reading database ... 63985 files and directories currently installed.)
Preparing to unpack .../0-conntrack_1%3a1.4.5-2_amd64.deb ...
Unpacking conntrack (1:1.4.5-2) ...
Selecting previously unselected package cri-tools.
Preparing to unpack .../1-cri-tools_1.30.1-1.1_amd64.deb ...
Unpacking cri-tools (1.30.1-1.1) ...
Selecting previously unselected package kubeadm.
Preparing to unpack .../2-kubeadm_1.30.14-1.1_amd64.deb ...
Unpacking kubeadm (1.30.14-1.1) ...
Selecting previously unselected package kubectl.
Preparing to unpack .../3-kubectl_1.30.14-1.1_amd64.deb ...
Unpacking kubectl (1.30.14-1.1) ...
Selecting previously unselected package kubernetes-cni.
Preparing to unpack .../4-kubernetes-cni_1.4.0-1.1_amd64.deb ...
Unpacking kubernetes-cni (1.4.0-1.1) ...
Selecting previously unselected package kubelet.
Preparing to unpack .../5-kubelet_1.30.14-1.1_amd64.deb ...
Unpacking kubelet (1.30.14-1.1) ...
Setting up conntrack (1:1.4.5-2) ...
Setting up kubectl (1.30.14-1.1) ...
Setting up cri-tools (1.30.1-1.1) ...
Setting up kubernetes-cni (1.4.0-1.1) ...
Setting up kubeadm (1.30.14-1.1) ...
Setting up kubelet (1.30.14-1.1) ...
Processing triggers for man-db (2.9.1-1) ...
vagrant@oai-k8s-vm:~$ 
vagrant@oai-k8s-vm:~$
vagrant@oai-k8s-vm:~$ sudo apt-mark hold kubelet kubeadm kubectl
l enable --now kubeletkubelet set on hold.
kubeadm set on hold.
kubectl set on hold.
vagrant@oai-k8s-vm:~$ sudo systemctl enable --now kubelet
vagrant@oai-k8s-vm:~$ 
vagrant@oai-k8s-vm:~$
vagrant@oai-k8s-vm:~$ containerd --version 
containerd github.com/containerd/containerd 1.7.24 
vagrant@oai-k8s-vm:~$ sudo systemctl status containerd
● containerd.service - containerd container runtime
     Loaded: loaded (/lib/systemd/system/containerd.service; enabled; >
     Active: active (running) since Mon 2025-07-07 09:07:28 UTC; 10min>
       Docs: https://containerd.io
   Main PID: 16290 (containerd)
      Tasks: 10
     Memory: 14.4M
     CGroup: /system.slice/containerd.service
             └─16290 /usr/bin/containerd

Jul 07 09:07:28 oai-k8s-vm containerd[16290]: time="2025-07-07T09:07:2>
Jul 07 09:07:28 oai-k8s-vm containerd[16290]: time="2025-07-07T09:07:2>
Jul 07 09:07:28 oai-k8s-vm containerd[16290]: time="2025-07-07T09:07:2>
Jul 07 09:07:28 oai-k8s-vm containerd[16290]: time="2025-07-07T09:07:2>
Jul 07 09:07:28 oai-k8s-vm containerd[16290]: time="2025-07-07T09:07:2>
Jul 07 09:07:28 oai-k8s-vm containerd[16290]: time="2025-07-07T09:07:2>
Jul 07 09:07:28 oai-k8s-vm containerd[16290]: time="2025-07-07T09:07:2>
Jul 07 09:07:28 oai-k8s-vm containerd[16290]: time="2025-07-07T09:07:2>
Jul 07 09:07:28 oai-k8s-vm systemd[1]: Started containerd container ru>
Jul 07 09:15:10 oai-k8s-vm containerd[16290]: time="2025-07-07T09:15:1>

vagrant@oai-k8s-vm:~$ kubelet --version
Kubernetes v1.30.14
vagrant@oai-k8s-vm:~$ kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"30", GitVersion:"v1.30.14", GitCommit:"9e18483918821121abdf9aa82bc14d66df5d68cd", GitTreeState:"clean", BuildDate:"2025-06-17T18:34:53Z", GoVersion:"go1.23.10", Compiler:"gc", Platform:"linux/amd64"}
vagrant@oai-k8s-vm:~$ kubectl version --client
Client Version: v1.30.14
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
vagrant@oai-k8s-vm:~$ 
vagrant@oai-k8s-vm:~$
vagrant@oai-k8s-vm:~$
vagrant@oai-k8s-vm:~$
vagrant@oai-k8s-vm:~$
vagrant@oai-k8s-vm:~$ sudo kubeadm init \
> --pod-network-cidr 10.20.0.0/16 \
> --service-cidr 10.21.0.0/16 \
> --apiserver-advertise-address 172.17.42.15
I0707 09:21:08.484754   18363 version.go:256] remote version is much newer: v1.33.2; falling back to: stable-1.30
[init] Using Kubernetes version: v1.30.14
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
W0707 09:21:09.729326   18363 checks.go:844] detected that the sandbox image "registry.k8s.io/pause:3.8" of the container runtime is inconsistent with that used by kubeadm.It is recommended to use "registry.k8s.io/pause:3.9" as the CRI sandbox image.
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local oai-k8s-vm] and IPs [10.21.0.1 172.17.42.15]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [localhost oai-k8s-vm] and IPs [172.17.42.15 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [localhost oai-k8s-vm] and IPs [172.17.42.15 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "super-admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"      
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"      
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests"
[kubelet-check] Waiting for a healthy kubelet at http://127.0.0.1:10248/healthz. This can take up to 4m0s
[kubelet-check] The kubelet is healthy after 1.005906175s
[api-check] Waiting for a healthy API server. This can take up to 4m0s 
[api-check] The API server is healthy after 14.004843723s
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node oai-k8s-vm as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node oai-k8s-vm as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]     
[bootstrap-token] Using token: 5b2bov.pvmyr1o3uzxqyfjr
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/   

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.17.42.15:6443 --token 5b2bov.pvmyr1o3uzxqyfjr \       
        --discovery-token-ca-cert-hash sha256:25c3be3cbbbee0bb78c4eedc93024c42da34109a2a4746840e5c245fcb1d5364
vagrant@oai-k8s-vm:~$ 
vagrant@oai-k8s-vm:~$
vagrant@oai-k8s-vm:~$
vagrant@oai-k8s-vm:~$ # Configure kubectl access for the vagrant user
vagrant@oai-k8s-vm:~$ mkdir -p $HOME/.kube
config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Untaivagrant@oai-k8s-vm:~$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
nt the control-plane node to allow scheduling pods on it
kubectl taint nodes $(kubectl get nodes -o jsonpath='{.items[0].metadata.name}') node-role.kubernetes.io/controvagrant@oai-k8s-vm:~$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
l-plane-vagrant@oai-k8s-vm:~$ 
vagrant@oai-k8s-vm:~$ # Untaint the control-plane node to allow scheduling pods on it
vagrant@oai-k8s-vm:~$ kubectl get nodes
NAME         STATUS     ROLES           AGE     VERSION
oai-k8s-vm   NotReady   control-plane   2m59s   v1.30.14
vagrant@oai-k8s-vm:~$ kubectl describe node oai-k8s-vm
Name:               oai-k8s-vm
Roles:              control-plane
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=oai-k8s-vm
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/control-plane=
                    node.kubernetes.io/exclude-from-external-load-balancers=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: unix:///var/run/containerd/containerd.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Mon, 07 Jul 2025 09:23:12 +0000
Taints:             node-role.kubernetes.io/control-plane:NoSchedule   
                    node.kubernetes.io/not-ready:NoSchedule
Unschedulable:      false
Lease:
  HolderIdentity:  oai-k8s-vm
  AcquireTime:     <unset>
  RenewTime:       Mon, 07 Jul 2025 09:26:32 +0000
Conditions:
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----             ------  -----------------                 ------------------                ------                       -------
  MemoryPressure   False   Mon, 07 Jul 2025 09:23:28 +0000   Mon, 07 Jul 2025 09:23:08 +0000   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Mon, 07 Jul 2025 09:23:28 +0000   Mon, 07 Jul 2025 09:23:08 +0000   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure      False   Mon, 07 Jul 2025 09:23:28 +0000   Mon, 07 Jul 2025 09:23:08 +0000   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready            False   Mon, 07 Jul 2025 09:23:28 +0000   Mon, 07 Jul 2025 09:23:08 +0000   KubeletNotReady              container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized
Addresses:
  InternalIP:  10.0.2.15
  Hostname:    oai-k8s-vm
Capacity:
  cpu:                4
  ephemeral-storage:  50745656Ki
  hugepages-2Mi:      0
  memory:             6071776Ki
  pods:               110
Allocatable:
  cpu:                4
  ephemeral-storage:  46767196493
  hugepages-2Mi:      0
  memory:             5969376Ki
  pods:               110
System Info:
  Machine ID:                 2c81b08720f34fab817da0595434f81d
  System UUID:                0d30e8d1-371f-fd41-8b77-fbb20173a0d9     
  Boot ID:                    6592f70d-8ccb-49a0-9963-03cb2cfc124d     
  Kernel Version:             5.4.0-216-generic
  OS Image:                   Ubuntu 20.04.6 LTS
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  containerd://1.7.24
  Kubelet Version:            v1.30.14
  Kube-Proxy Version:         v1.30.14
PodCIDR:                      10.20.0.0/24
PodCIDRs:                     10.20.0.0/24
Non-terminated Pods:          (5 in total)
  Namespace                   Name                                  CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                                  ------------  ----------  ---------------  -------------  ---
  kube-system                 etcd-oai-k8s-vm                       100m (2%)     0 (0%)      100Mi (1%)       0 (0%)         3m24s
  kube-system                 kube-apiserver-oai-k8s-vm             250m (6%)     0 (0%)      0 (0%)           0 (0%)         3m24s
  kube-system                 kube-controller-manager-oai-k8s-vm    200m (5%)     0 (0%)      0 (0%)           0 (0%)         3m26s
  kube-system                 kube-proxy-l76v4                      0 (0%)        0 (0%)      0 (0%)           0 (0%)         3m11s
  kube-system                 kube-scheduler-oai-k8s-vm             100m (2%)     0 (0%)      0 (0%)           0 (0%)         3m27s
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests    Limits
  --------           --------    ------
  cpu                650m (16%)  0 (0%)
  memory             100Mi (1%)  0 (0%)
  ephemeral-storage  0 (0%)      0 (0%)
  hugepages-2Mi      0 (0%)      0 (0%)
Events:
  Type     Reason                   Age                    From             Message
  ----     ------                   ----                   ----             -------
  Normal   Starting                 3m9s                   kube-proxy  

  Normal   NodeAllocatableEnforced  3m40s                  kubelet          Updated Node Allocatable limit across pods
  Warning  InvalidDiskCapacity      3m40s                  kubelet          invalid capacity 0 on image filesystem
  Normal   NodeHasSufficientMemory  3m40s (x6 over 3m40s)  kubelet          Node oai-k8s-vm status is now: NodeHasSufficientMemory
  Normal   NodeHasNoDiskPressure    3m40s (x6 over 3m40s)  kubelet          Node oai-k8s-vm status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     3m40s (x6 over 3m40s)  kubelet          Node oai-k8s-vm status is now: NodeHasSufficientPID
  Normal   Starting                 3m40s                  kubelet          Starting kubelet.
  Normal   Starting                 3m24s                  kubelet          Starting kubelet.
  Warning  InvalidDiskCapacity      3m24s                  kubelet          invalid capacity 0 on image filesystem
  Normal   NodeHasSufficientMemory  3m24s                  kubelet          Node oai-k8s-vm status is now: NodeHasSufficientMemory
  Normal   NodeHasNoDiskPressure    3m24s                  kubelet          Node oai-k8s-vm status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     3m24s                  kubelet          Node oai-k8s-vm status is now: NodeHasSufficientPID
  Normal   NodeAllocatableEnforced  3m24s                  kubelet          Updated Node Allocatable limit across pods
  Normal   RegisteredNode           3m11s                  node-controller  Node oai-k8s-vm event: Registered Node oai-k8s-vm in Controller   
vagrant@oai-k8s-vm:~$
vagrant@oai-k8s-vm:~$
vagrant@oai-k8s-vm:~$ 
vagrant@oai-k8s-vm:~$
vagrant@oai-k8s-vm:~$ # Install the Tigera Operator
vagrant@oai-k8s-vm:~$ kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml

namespace/tigera-operator created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgpfilters.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/caliconodestatuses.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipreservations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/kubecontrollersconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/apiservers.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/imagesets.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/installations.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/tigerastatuses.operator.tigera.io created
serviceaccount/tigera-operator created
clusterrole.rbac.authorization.k8s.io/tigera-operator created
clusterrolebinding.rbac.authorization.k8s.io/tigera-operator created
deployment.apps/tigera-operator created
vagrant@oai-k8s-vm:~$
vagrant@oai-k8s-vm:~$ # Download the calicoctl CLI
vagrant@oai-k8s-vm:~$ curl -L https://github.com/projectcalico/calico/releases/download/v3.26.1/calicoctl-linux-amd64 -o calicoctl
n/  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:-  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:-  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:01 --:--:-  4 62.7M    4 2959k    0     0  1280k      0  0:00:50  0:00:02  0:00:4 20 62.7M   20 13.1M    0     0  4031k      0  0:00:15  0:00:03  0:00:1 32 62.7M   32 20.2M    0     0  4802k      0  0:00:13  0:00:04  0:00:0 55 62.7M   55 34.8M    0     0  6712k      0  0:00:09  0:00:05  0:00:0 72 62.7M   72 45.6M    0     0  7402k      0  0:00:08  0:00:06  0:00:0 90 62.7M   90 56.7M    0     0  7941k      0  0:00:08  0:00:07  0:00:0 98 62.7M   98 61.6M    0     0  7593k      0  0:00:08  0:00:08 --:--:-100 62.7M  100 62.7M    0     0  7643k      0  0:00:08  0:00:08 --:--:-- 10.3M
vagrant@oai-k8s-vm:~$ chmod +x ./calicoctl
vagrant@oai-k8s-vm:~$ sudo mv ./calicoctl /usr/local/bin/
vagrant@oai-k8s-vm:~$
vagrant@oai-k8s-vm:~$
vagrant@oai-k8s-vm:~$ nano calico-install.yaml
vagrant@oai-k8s-vm:~$ kubectl apply -f calico-install.yaml
installation.operator.tigera.io/default created
vagrant@oai-k8s-vm:~$


vagrant@oai-k8s-vm:~$ ls
bgp-peer.yaml  calico-install.yaml
vagrant@oai-k8s-vm:~$ nano bgp-peer.yaml
vagrant@oai-k8s-vm:~$ nano bgp-peer.yaml
vagrant@oai-k8s-vm:~$ sudo calicoctl apply -f bgp-peer.yaml
Failed to create Calico API client: invalid configuration: no configuration has been provided, try setting KUBERNETES_MASTER environment variable
vagrant@oai-k8s-vm:~$ nano calico-install.yaml
vagrant@oai-k8s-vm:~$ kubectl apply -f calico-install.yaml
installation.operator.tigera.io/default configured
apiserver.operator.tigera.io/default created
vagrant@oai-k8s-vm:~$
vagrant@oai-k8s-vm:~$
vagrant@oai-k8s-vm:~$ sudo calicoctl apply -f bgp-peer.yaml
Failed to create Calico API client: invalid configuration: no configuration has been provided, try setting KUBERNETES_MASTER environment variable
vagrant@oai-k8s-vm:~$
vagrant@oai-k8s-vm:~$
vagrant@oai-k8s-vm:~$ sudo calicoctl --kubeconfig=/etc/kubernetes/admin.conf apply -f bgp-peer.yaml
Usage:
  calicoctl [options] <command> [<args>...]
Invalid option: '--kubeconfig=/etc/kubernetes/admin.conf apply -f bgp-peer.yaml'. Use flag '--help' to read about a specific subcommand.      
vagrant@oai-k8s-vm:~$ export KUBECONFIG=/etc/kubernetes/admin.conf     
vagrant@oai-k8s-vm:~$ sudo calicoctl apply -f bgp-peer.yaml
Failed to create Calico API client: invalid configuration: no configuration has been provided, try setting KUBERNETES_MASTER environment variable
vagrant@oai-k8s-vm:~$ sudo -E calicoctl apply -f bgp-peer.yaml
Successfully applied 2 resource(s)
vagrant@oai-k8s-vm:~$ nano calico-install.yaml
vagrant@oai-k8s-vm:~$ 
vagrant@oai-k8s-vm:~$
vagrant@oai-k8s-vm:~$ 
vagrant@oai-k8s-vm:~$ sudo calicoctl node status
Calico process is running.

IPv4 BGP status
+--------------+-----------+-------+----------+-------------+
| PEER ADDRESS | PEER TYPE | STATE |  SINCE   |    INFO     |
+--------------+-----------+-------+----------+-------------+
| 172.17.42.27 | global    | up    | 09:47:37 | Established |
+--------------+-----------+-------+----------+-------------+

IPv6 BGP status
No IPv6 peers found.

vagrant@oai-k8s-vm:~$ 
vagrant@oai-k8s-vm:~$
vagrant@oai-k8s-vm:~$ # testing ........................
vagrant@oai-k8s-vm:~$ kubectl run nginx-c1 --image=nginx
error: error loading config file "/etc/kubernetes/admin.conf": open /etc/kubernetes/admin.conf: permission denied
vagrant@oai-k8s-vm:~$ 
vagrant@oai-k8s-vm:~$ 
vagrant@oai-k8s-vm:~$ mkdir -p $HOME/.kube
E/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
vagrant@oai-k8s-vm:~$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
cp: overwrite '/home/vagrant/.kube/config'? vagrant@oai-k8s-vm:~$ 
vagrant@oai-k8s-vm:~$ unset KUBECONFIG
vagrant@oai-k8s-vm:~$
vagrant@oai-k8s-vm:~$
vagrant@oai-k8s-vm:~$ kubectl run nginx-c1 --image=nginx
pod/nginx-c1 created
vagrant@oai-k8s-vm:~$ kubectl get pods -o wide
NAME       READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
nginx-c1   0/1     Pending   0          45s   <none>   <none>   <none>           <none>
vagrant@oai-k8s-vm:~$ kubectl describe pod nginx-c1
Name:             nginx-c1
Namespace:        default
Priority:         0
Service Account:  default
Node:             <none>
Labels:           run=nginx-c1
Annotations:      <none>
Status:           Pending
IP:
IPs:              <none>
Containers:
  nginx-c1:
    Image:        nginx
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-rbkzk (ro)
Conditions:
  Type           Status
  PodScheduled   False
Volumes:
  kube-api-access-rbkzk:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  77s   default-scheduler  0/1 nodes are available: 1 node(s) had untolerated taint {node-role.kubernetes.io/control-plane: }. preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling.
vagrant@oai-k8s-vm:~$ kubectl taint nodes $(kubectl get nodes -o jsonpath='{.items[0].metadata.name}') node-role.kubernetes.io/control-plane- 
node/oai-k8s-vm untainted
vagrant@oai-k8s-vm:~$
vagrant@oai-k8s-vm:~$
vagrant@oai-k8s-vm:~$ watch kubectl get pods -o wide
vagrant@oai-k8s-vm:~$ kubectl get pods -o wide
NAME       READY   STATUS    RESTARTS   AGE     IP            NODE         NOMINATED NODE   READINESS GATES
nginx-c1   1/1     Running   0          2m42s   10.20.127.6   oai-k8s-vm   <none>           <none>
vagrant@oai-k8s-vm:~$ 
```

### open5gs-k8s-vm calico bgp setup logs
```ruby
       valid_lft 86349sec preferred_lft 14349sec
    inet6 fe80::1a:a2ff:fe1e:9db7/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:b7:7d:ec brd ff:ff:ff:ff:ff:ff
    inet 172.17.42.27/24 brd 172.17.42.255 scope global enp0s8
    inet6 fe80::1a:a2ff:fe1e:9db7/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:b7:7d:ec brd ff:ff:ff:ff:ff:ff
    inet 172.17.42.27/24 brd 172.17.42.255 scope global enp0s8
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:b7:7d:ec brd ff:ff:ff:ff:ff:ff
    inet 172.17.42.27/24 brd 172.17.42.255 scope global enp0s8
    link/ether 08:00:27:b7:7d:ec brd ff:ff:ff:ff:ff:ff
    inet 172.17.42.27/24 brd 172.17.42.255 scope global enp0s8
    inet 172.17.42.27/24 brd 172.17.42.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feb7:7dec/64 scope link
       valid_lft forever preferred_lft forever
vagrant@open5gs-k8s-vm:~$ s
s: command not found
vagrant@open5gs-k8s-vm:~$ ls
vagrant@open5gs-k8s-vm:~$
vagrant@open5gs-k8s-vm:~$ 
vagrant@open5gs-k8s-vm:~$ # Disable swap
vagrant@open5gs-k8s-vm:~$ sudo swapoff -a
\)$/#\1/g' /etc/fstabvagrant@open5gs-k8s-vm:~$ sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
vagrant@open5gs-k8s-vm:~$ 
vagrant@open5gs-k8s-vm:~$ 
vagrant@open5gs-k8s-vm:~$ # Enable kernel modules and sysctl settings for K8s networking
vagrant@open5gs-k8s-vm:~$ cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
> overlay
> br_netfilter
> EOF
etfilteroverlay
br_netfilter
vagrant@open5gs-k8s-vm:~$ sudo modprobe overlay
vagrant@open5gs-k8s-vm:~$ sudo modprobe br_netfilter
vagrant@open5gs-k8s-vm:~$ 
vagrant@open5gs-k8s-vm:~$ cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
> net.bridge.bridge-nf-call-ip6tables = 1
> net.bridge.bridge-nf-call-iptables = 1
> net.ipv4.ip_forward = 1
> EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
vagrant@open5gs-k8s-vm:~$ sudo sysctl --system
* Applying /etc/sysctl.d/10-console-messages.conf ...
kernel.printk = 4 4 1 7
* Applying /etc/sysctl.d/10-ipv6-privacy.conf ...
net.ipv6.conf.all.use_tempaddr = 2
net.ipv6.conf.default.use_tempaddr = 2
* Applying /etc/sysctl.d/10-kernel-hardening.conf ...
kernel.kptr_restrict = 1
* Applying /etc/sysctl.d/10-link-restrictions.conf ...
fs.protected_hardlinks = 1
fs.protected_symlinks = 1
* Applying /etc/sysctl.d/10-magic-sysrq.conf ...
kernel.sysrq = 176
* Applying /etc/sysctl.d/10-network-security.conf ...
net.ipv4.conf.default.rp_filter = 2
net.ipv4.conf.all.rp_filter = 2
* Applying /etc/sysctl.d/10-ptrace.conf ...
kernel.yama.ptrace_scope = 1
* Applying /etc/sysctl.d/10-zeropage.conf ...
vm.mmap_min_addr = 65536
* Applying /usr/lib/sysctl.d/50-default.conf ...
net.ipv4.conf.default.promote_secondaries = 1
sysctl: setting key "net.ipv4.conf.all.promote_secondaries": Invalid argument
net.ipv4.ping_group_range = 0 2147483647
net.core.default_qdisc = fq_codel
fs.protected_regular = 1
fs.protected_fifos = 1
* Applying /usr/lib/sysctl.d/50-pid-max.conf ...
kernel.pid_max = 4194304
* Applying /etc/sysctl.d/99-cloudimg-ipv6.conf ...
net.ipv6.conf.all.use_tempaddr = 0
net.ipv6.conf.default.use_tempaddr = 0
* Applying /etc/sysctl.d/99-sysctl.conf ...
* Applying /etc/sysctl.d/k8s.conf ...
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
* Applying /usr/lib/sysctl.d/protect-links.conf ...
fs.protected_fifos = 1
fs.protected_hardlinks = 1
fs.protected_regular = 2
fs.protected_symlinks = 1
* Applying /etc/sysctl.conf ...
vagrant@open5gs-k8s-vm:~$ 
vagrant@open5gs-k8s-vm:~$
vagrant@open5gs-k8s-vm:~$ # Install containerd (container runtime)
vagrant@open5gs-k8s-vm:~$ sudo apt-get update
Hit:1 http://archive.ubuntu.com/ubuntu focal InRelease
Get:2 http://security.ubuntu.com/ubuntu focal-security InRelease [128 kB]
Get:3 http://archive.ubuntu.com/ubuntu focal-updates InRelease [128 kB]
Get:4 http://archive.ubuntu.com/ubuntu focal-backports InRelease [128 kB]
Get:5 http://security.ubuntu.com/ubuntu focal-security/universe amd64 Packages [1040 kB]
Get:6 http://security.ubuntu.com/ubuntu focal-security/universe Translation-en [221 kB]
Get:7 http://security.ubuntu.com/ubuntu focal-security/universe amd64 c-n-f Metadata [22.4 kB]
Get:8 http://security.ubuntu.com/ubuntu focal-security/multiverse amd64 Packages [26.6 kB]
Get:9 http://security.ubuntu.com/ubuntu focal-security/multiverse Translation-en [6448 B]
Get:10 http://security.ubuntu.com/ubuntu focal-security/multiverse amd64 c-n-f Metadata [604 B]
Get:11 http://archive.ubuntu.com/ubuntu focal/universe amd64 Packages [8628 kB]
Get:12 http://archive.ubuntu.com/ubuntu focal/universe Translation-en [5124 kB]
Get:13 http://archive.ubuntu.com/ubuntu focal/universe amd64 c-n-f Metadata [265 kB]
Get:14 http://archive.ubuntu.com/ubuntu focal/multiverse amd64 Packages [144 kB]
Get:15 http://archive.ubuntu.com/ubuntu focal/multiverse Translation-en [104 kB]
Get:16 http://archive.ubuntu.com/ubuntu focal/multiverse amd64 c-n-f Metadata [9136 B]
Get:17 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 Packages [3954 kB]
Get:18 http://archive.ubuntu.com/ubuntu focal-updates/universe amd64 Packages [1262 kB]
Get:19 http://archive.ubuntu.com/ubuntu focal-updates/universe Translation-en [303 kB]
Get:20 http://archive.ubuntu.com/ubuntu focal-updates/universe amd64 c-n-f Metadata [29.3 kB]
Get:21 http://archive.ubuntu.com/ubuntu focal-updates/multiverse amd64 Packages [29.7 kB]
Get:22 http://archive.ubuntu.com/ubuntu focal-updates/multiverse Translation-en [8316 B]
Get:23 http://archive.ubuntu.com/ubuntu focal-updates/multiverse amd64 c-n-f Metadata [688 B]
Get:24 http://archive.ubuntu.com/ubuntu focal-backports/main amd64 Packages [45.7 kB]
Get:25 http://archive.ubuntu.com/ubuntu focal-backports/main Translation-en [16.3 kB]
Get:26 http://archive.ubuntu.com/ubuntu focal-backports/main amd64 c-n-f Metadata [1420 B]
Get:27 http://archive.ubuntu.com/ubuntu focal-backports/restricted amd64 c-n-f Metadata [116 B]
Get:28 http://archive.ubuntu.com/ubuntu focal-backports/universe amd64 Packages [25.0 kB]
Get:29 http://archive.ubuntu.com/ubuntu focal-backports/universe Translation-en [16.3 kB]
Get:30 http://archive.ubuntu.com/ubuntu focal-backports/universe amd64 c-n-f Metadata [880 B]
Get:31 http://archive.ubuntu.com/ubuntu focal-backports/multiverse amd64 c-n-f Metadata [116 B]
Fetched 21.7 MB in 14s (1527 kB/s)
Reading package lists... Done
vagrant@open5gs-k8s-vm:~$ sudo apt-get install -y containerd
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  runc
The following NEW packages will be installed:
  containerd runc
0 upgraded, 2 newly installed, 0 to remove and 0 not upgraded.
Need to get 41.7 MB of archives.
After this operation, 176 MB of additional disk space will be used.    
Get:1 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 runc amd64 1.1.12-0ubuntu2~20.04.1 [8066 kB]
Get:2 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 containerd amd64 1.7.24-0ubuntu1~20.04.2 [33.6 MB]
Fetched 41.7 MB in 8s (5420 kB/s)
Selecting previously unselected package runc.
(Reading database ... 63917 files and directories currently installed.)
Preparing to unpack .../runc_1.1.12-0ubuntu2~20.04.1_amd64.deb ...
Unpacking runc (1.1.12-0ubuntu2~20.04.1) ...
Selecting previously unselected package containerd.
Preparing to unpack .../containerd_1.7.24-0ubuntu1~20.04.2_amd64.deb ...
Unpacking containerd (1.7.24-0ubuntu1~20.04.2) ...
Setting up runc (1.1.12-0ubuntu2~20.04.1) ...
Setting up containerd (1.7.24-0ubuntu1~20.04.2) ...
Created symlink /etc/systemd/system/multi-user.target.wants/containerd.service → /lib/systemd/system/containerd.service.
Processing triggers for man-db (2.9.1-1) ...
vagrant@open5gs-k8s-vm:~$ sudo mkdir -p /etc/containerd
lse/SystemdCgroup = true/g' /etc/containerd/config.toml
sudo systemctl restartvagrant@open5gs-k8s-vm:~$ sudo containerd config default | sudo tee /etc/containerd/config.toml
 containerddisabled_plugins = []
imports = []
oom_score = 0
plugin_dir = ""
required_plugins = []
root = "/var/lib/containerd"
state = "/run/containerd"
temp = ""
version = 2

[cgroup]
  path = ""

[debug]
  address = ""
  format = ""
  gid = 0
  level = ""
  uid = 0

[grpc]
  address = "/run/containerd/containerd.sock"
  gid = 0
  max_recv_message_size = 16777216
  max_send_message_size = 16777216
  tcp_address = ""
  tcp_tls_ca = ""
  tcp_tls_cert = ""
  tcp_tls_key = ""
  uid = 0

[metrics]
  address = ""
  grpc_histogram = false

[plugins]

  [plugins."io.containerd.gc.v1.scheduler"]
    deletion_threshold = 0
    mutation_threshold = 100
    pause_threshold = 0.02
    schedule_delay = "0s"
    startup_delay = "100ms"

  [plugins."io.containerd.grpc.v1.cri"]
    cdi_spec_dirs = ["/etc/cdi", "/var/run/cdi"]
    device_ownership_from_security_context = false
    disable_apparmor = false
    disable_cgroup = false
    disable_hugetlb_controller = true
    disable_proc_mount = false
    disable_tcp_service = true
    drain_exec_sync_io_timeout = "0s"
    enable_cdi = false
    enable_selinux = false
    enable_tls_streaming = false
    enable_unprivileged_icmp = false
    enable_unprivileged_ports = false
    ignore_deprecation_warnings = []
    ignore_image_defined_volumes = false
    image_pull_progress_timeout = "5m0s"
    image_pull_with_sync_fs = false
    max_concurrent_downloads = 3
    max_container_log_line_size = 16384
    netns_mounts_under_state_dir = false
    restrict_oom_score_adj = false
    sandbox_image = "registry.k8s.io/pause:3.8"
    selinux_category_range = 1024
    stats_collect_period = 10
    stream_idle_timeout = "4h0m0s"
    stream_server_address = "127.0.0.1"
    stream_server_port = "0"
    systemd_cgroup = false
    tolerate_missing_hugetlb_controller = true
    unset_seccomp_profile = ""

    [plugins."io.containerd.grpc.v1.cri".cni]
      bin_dir = "/opt/cni/bin"
      conf_dir = "/etc/cni/net.d"
      conf_template = ""
      ip_pref = ""
      max_conf_num = 1
      setup_serially = false

    [plugins."io.containerd.grpc.v1.cri".containerd]
      default_runtime_name = "runc"
      disable_snapshot_annotations = true
      discard_unpacked_layers = false
      ignore_blockio_not_enabled_errors = false
      ignore_rdt_not_enabled_errors = false
      no_pivot = false
      snapshotter = "overlayfs"

      [plugins."io.containerd.grpc.v1.cri".containerd.default_runtime] 
        base_runtime_spec = ""
        cni_conf_dir = ""
        cni_max_conf_num = 0
        container_annotations = []
        pod_annotations = []
        privileged_without_host_devices = false
        privileged_without_host_devices_all_devices_allowed = false    
        runtime_engine = ""
        runtime_path = ""
        runtime_root = ""
        runtime_type = ""
        sandbox_mode = ""
        snapshotter = ""

        [plugins."io.containerd.grpc.v1.cri".containerd.default_runtime.options]

      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]        

        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc] 
          base_runtime_spec = ""
          cni_conf_dir = ""
          cni_max_conf_num = 0
          container_annotations = []
          pod_annotations = []
          privileged_without_host_devices = false
          privileged_without_host_devices_all_devices_allowed = false  
          runtime_engine = ""
          runtime_path = ""
          runtime_root = ""
          runtime_type = "io.containerd.runc.v2"
          sandbox_mode = "podsandbox"
          snapshotter = ""

          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            BinaryName = ""
            CriuImagePath = ""
            CriuPath = ""
            CriuWorkPath = ""
            IoGid = 0
            IoUid = 0
            NoNewKeyring = false
            NoPivotRoot = false
            Root = ""
            ShimCgroup = ""
            SystemdCgroup = false

      [plugins."io.containerd.grpc.v1.cri".containerd.untrusted_workload_runtime]
        base_runtime_spec = ""
        cni_conf_dir = ""
        cni_max_conf_num = 0
        container_annotations = []
        pod_annotations = []
        privileged_without_host_devices = false
        privileged_without_host_devices_all_devices_allowed = false    
        runtime_engine = ""
        runtime_path = ""
        runtime_root = ""
        runtime_type = ""
        sandbox_mode = ""
        snapshotter = ""

        [plugins."io.containerd.grpc.v1.cri".containerd.untrusted_workload_runtime.options]

    [plugins."io.containerd.grpc.v1.cri".image_decryption]
      key_model = "node"

    [plugins."io.containerd.grpc.v1.cri".registry]
      config_path = ""

      [plugins."io.containerd.grpc.v1.cri".registry.auths]

      [plugins."io.containerd.grpc.v1.cri".registry.configs]

      [plugins."io.containerd.grpc.v1.cri".registry.headers]

      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]

    [plugins."io.containerd.grpc.v1.cri".x509_key_pair_streaming]      
      tls_cert_file = ""
      tls_key_file = ""

  [plugins."io.containerd.internal.v1.opt"]
    path = "/opt/containerd"

  [plugins."io.containerd.internal.v1.restart"]
    interval = "10s"

  [plugins."io.containerd.internal.v1.tracing"]

  [plugins."io.containerd.metadata.v1.bolt"]
    content_sharing_policy = "shared"

  [plugins."io.containerd.monitor.v1.cgroups"]
    no_prometheus = false

  [plugins."io.containerd.nri.v1.nri"]
    disable = true
    disable_connections = false
    plugin_config_path = "/etc/nri/conf.d"
    plugin_path = "/opt/nri/plugins"
    plugin_registration_timeout = "5s"
    plugin_request_timeout = "2s"
    socket_path = "/var/run/nri/nri.sock"

  [plugins."io.containerd.runtime.v1.linux"]
    no_shim = false
    runtime = "runc"
    runtime_root = ""
    shim = "containerd-shim"
    shim_debug = false

  [plugins."io.containerd.runtime.v2.task"]
    platforms = ["linux/amd64"]
    sched_core = false

  [plugins."io.containerd.service.v1.diff-service"]
    default = ["walking"]

  [plugins."io.containerd.service.v1.tasks-service"]
    blockio_config_file = ""
    rdt_config_file = ""

  [plugins."io.containerd.snapshotter.v1.aufs"]
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.blockfile"]
    fs_type = ""
    mount_options = []
    root_path = ""
    scratch_file = ""

  [plugins."io.containerd.snapshotter.v1.btrfs"]
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.devmapper"]
    async_remove = false
    base_image_size = ""
    discard_blocks = false
    fs_options = ""
    fs_type = ""
    pool_name = ""
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.native"]
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.overlayfs"]
    mount_options = []
    root_path = ""
    sync_remove = false
    upperdir_label = false

  [plugins."io.containerd.snapshotter.v1.zfs"]
    root_path = ""

  [plugins."io.containerd.tracing.processor.v1.otlp"]

  [plugins."io.containerd.transfer.v1.local"]
    config_path = ""
    max_concurrent_downloads = 3
    max_concurrent_uploaded_layers = 3

    [[plugins."io.containerd.transfer.v1.local".unpack_config]]        
      differ = ""
      platform = "linux/amd64"
      snapshotter = "overlayfs"

[proxy_plugins]

[stream_processors]

  [stream_processors."io.containerd.ocicrypt.decoder.v1.tar"]
    accepts = ["application/vnd.oci.image.layer.v1.tar+encrypted"]     
    args = ["--decryption-keys-path", "/etc/containerd/ocicrypt/keys"] 
    env = ["OCICRYPT_KEYPROVIDER_CONFIG=/etc/containerd/ocicrypt/ocicrypt_keyprovider.conf"]
    path = "ctd-decoder"
    returns = "application/vnd.oci.image.layer.v1.tar"

  [stream_processors."io.containerd.ocicrypt.decoder.v1.tar.gzip"]     
    accepts = ["application/vnd.oci.image.layer.v1.tar+gzip+encrypted"]
    args = ["--decryption-keys-path", "/etc/containerd/ocicrypt/keys"] 
    env = ["OCICRYPT_KEYPROVIDER_CONFIG=/etc/containerd/ocicrypt/ocicrypt_keyprovider.conf"]
    path = "ctd-decoder"
    returns = "application/vnd.oci.image.layer.v1.tar+gzip"

[timeouts]
  "io.containerd.timeout.bolt.open" = "0s"
  "io.containerd.timeout.metrics.shimstats" = "2s"
  "io.containerd.timeout.shim.cleanup" = "5s"
  "io.containerd.timeout.shim.load" = "5s"
  "io.containerd.timeout.shim.shutdown" = "3s"
  "io.containerd.timeout.task.state" = "2s"

[ttrpc]
  address = ""
  gid = 0
  uid = 0
vagrant@open5gs-k8s-vm:~$ sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
vagrant@open5gs-k8s-vm:~$ sudo systemctl restart containerd
vagrant@open5gs-k8s-vm:~$ 
vagrant@open5gs-k8s-vm:~$
vagrant@open5gs-k8s-vm:~$ # Install Kubernetes components
vagrant@open5gs-k8s-vm:~$ sudo apt-get install -y apt-transport-https ca-certificates curl
Reading package lists... Done
Building dependency tree       
Reading state information... Done
ca-certificates is already the newest version (20240203~20.04.1).
ca-certificates set to manually installed.
curl is already the newest version (7.68.0-1ubuntu2.25).
curl set to manually installed.
The following NEW packages will be installed:
  apt-transport-https
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 1704 B of archives.
After this operation, 162 kB of additional disk space will be used.    
Get:1 http://archive.ubuntu.com/ubuntu focal-updates/universe amd64 apt-transport-https all 2.0.11 [1704 B]
Fetched 1704 B in 2s (859 B/s)
Selecting previously unselected package apt-transport-https.
(Reading database ... 63981 files and directories currently installed.)
Preparing to unpack .../apt-transport-https_2.0.11_all.deb ...
Unpacking apt-transport-https (2.0.11) ...
Setting up apt-transport-https (2.0.11) ...
vagrant@open5gs-k8s-vm:~$ 
vagrant@open5gs-k8s-vm:~$
vagrant@open5gs-k8s-vm:~$ sudo mkdir -p /etc/apt/keyrings
e:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-byvagrant@open5gs-k8s-vm:~$ curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.listvagrant@open5gs-k8s-vm:~$ echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /
vagrant@open5gs-k8s-vm:~$
vagrant@open5gs-k8s-vm:~$
vagrant@open5gs-k8s-vm:~$ sudo apt-get update
Hit:2 http://archive.ubuntu.com/ubuntu focal InRelease
Hit:3 http://security.ubuntu.com/ubuntu focal-security InRelease
Get:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  InRelease [1192 B]
Get:4 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  Packages [20.3 kB]
Get:5 http://archive.ubuntu.com/ubuntu focal-updates InRelease [128 kB]
Hit:6 http://archive.ubuntu.com/ubuntu focal-backports InRelease
Fetched 149 kB in 3s (59.5 kB/s)
Reading package lists... Done
vagrant@open5gs-k8s-vm:~$ sudo apt-get install -y kubelet kubeadm kubectl
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  conntrack cri-tools kubernetes-cni
Suggested packages:
  nftables
The following NEW packages will be installed:
  conntrack cri-tools kubeadm kubectl kubelet kubernetes-cni
0 upgraded, 6 newly installed, 0 to remove and 0 not upgraded.
Need to get 93.8 MB of archives.
After this operation, 343 MB of additional disk space will be used.    
Get:3 http://archive.ubuntu.com/ubuntu focal/main amd64 conntrack amd64 1:1.4.5-2 [30.3 kB]
Get:1 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  cri-tools 1.30.1-1.1 [21.3 MB]
Get:2 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  kubeadm 1.30.14-1.1 [10.5 MB]
Get:4 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  kubectl 1.30.14-1.1 [10.9 MB]
Get:5 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  kubernetes-cni 1.4.0-1.1 [32.9 MB]
Get:6 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  kubelet 1.30.14-1.1 [18.2 MB]
Fetched 93.8 MB in 15s (6305 kB/s)                                     
Selecting previously unselected package conntrack.
(Reading database ... 63985 files and directories currently installed.)
Preparing to unpack .../0-conntrack_1%3a1.4.5-2_amd64.deb ...
Unpacking conntrack (1:1.4.5-2) ...
Selecting previously unselected package cri-tools.
Preparing to unpack .../1-cri-tools_1.30.1-1.1_amd64.deb ...
Unpacking cri-tools (1.30.1-1.1) ...
Selecting previously unselected package kubeadm.
Preparing to unpack .../2-kubeadm_1.30.14-1.1_amd64.deb ...
Unpacking kubeadm (1.30.14-1.1) ...
Selecting previously unselected package kubectl.
Preparing to unpack .../3-kubectl_1.30.14-1.1_amd64.deb ...
Unpacking kubectl (1.30.14-1.1) ...
Selecting previously unselected package kubernetes-cni.
Preparing to unpack .../4-kubernetes-cni_1.4.0-1.1_amd64.deb ...
Unpacking kubernetes-cni (1.4.0-1.1) ...
Selecting previously unselected package kubelet.
Preparing to unpack .../5-kubelet_1.30.14-1.1_amd64.deb ...
Unpacking kubelet (1.30.14-1.1) ...
Setting up conntrack (1:1.4.5-2) ...
Setting up kubectl (1.30.14-1.1) ...
Setting up cri-tools (1.30.1-1.1) ...
Setting up kubernetes-cni (1.4.0-1.1) ...
Setting up kubeadm (1.30.14-1.1) ...
Setting up kubelet (1.30.14-1.1) ...
Processing triggers for man-db (2.9.1-1) ...
vagrant@open5gs-k8s-vm:~$ 
vagrant@open5gs-k8s-vm:~$
vagrant@open5gs-k8s-vm:~$ sudo apt-mark hold kubelet kubeadm kubectl
systemctl enable --now kubeletkubelet set on hold.
kubeadm set on hold.
kubectl set on hold.
vagrant@open5gs-k8s-vm:~$ sudo systemctl enable --now kubelet
vagrant@open5gs-k8s-vm:~$ 
vagrant@open5gs-k8s-vm:~$
vagrant@open5gs-k8s-vm:~$ containerd --version 
containerd github.com/containerd/containerd 1.7.24 
vagrant@open5gs-k8s-vm:~$ sudo systemctl status containerd
● containerd.service - containerd container runtime
     Loaded: loaded (/lib/systemd/system/containerd.service; enabled; >
     Active: active (running) since Mon 2025-07-07 09:07:33 UTC; 10min>
       Docs: https://containerd.io
   Main PID: 16048 (containerd)
      Tasks: 9
     Memory: 13.7M
     CGroup: /system.slice/containerd.service
             └─16048 /usr/bin/containerd

Jul 07 09:07:33 open5gs-k8s-vm containerd[16048]: time="2025-07-07T09:>
Jul 07 09:07:33 open5gs-k8s-vm containerd[16048]: time="2025-07-07T09:>
Jul 07 09:07:33 open5gs-k8s-vm containerd[16048]: time="2025-07-07T09:>
Jul 07 09:07:33 open5gs-k8s-vm containerd[16048]: time="2025-07-07T09:>
Jul 07 09:07:33 open5gs-k8s-vm containerd[16048]: time="2025-07-07T09:>
Jul 07 09:07:33 open5gs-k8s-vm containerd[16048]: time="2025-07-07T09:>
Jul 07 09:07:33 open5gs-k8s-vm containerd[16048]: time="2025-07-07T09:>
Jul 07 09:07:33 open5gs-k8s-vm containerd[16048]: time="2025-07-07T09:>
Jul 07 09:07:33 open5gs-k8s-vm systemd[1]: Started containerd containe>
Jul 07 09:15:20 open5gs-k8s-vm containerd[16048]: time="2025-07-07T09:>

vagrant@open5gs-k8s-vm:~$ kubelet --version
Kubernetes v1.30.14
vagrant@open5gs-k8s-vm:~$ kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"30", GitVersion:"v1.30.14", GitCommit:"9e18483918821121abdf9aa82bc14d66df5d68cd", GitTreeState:"clean", BuildDate:"2025-06-17T18:34:53Z", GoVersion:"go1.23.10", Compiler:"gc", Platform:"linux/amd64"}
vagrant@open5gs-k8s-vm:~$ kubectl version --client
Client Version: v1.30.14
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
vagrant@open5gs-k8s-vm:~$
vagrant@open5gs-k8s-vm:~$ 
vagrant@open5gs-k8s-vm:~$
vagrant@open5gs-k8s-vm:~$
vagrant@open5gs-k8s-vm:~$
vagrant@open5gs-k8s-vm:~$
vagrant@open5gs-k8s-vm:~$ sudo kubeadm init \
> --pod-network-cidr 10.30.0.0/16 \
> --service-cidr 10.31.0.0/16 \
> --apiserver-advertise-address 172.17.42.27
I0707 09:21:27.807590   18096 version.go:256] remote version is much newer: v1.33.2; falling back to: stable-1.30
[init] Using Kubernetes version: v1.30.14
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
W0707 09:21:29.107421   18096 checks.go:844] detected that the sandbox image "registry.k8s.io/pause:3.8" of the container runtime is inconsistent with that used by kubeadm.It is recommended to use "registry.k8s.io/pause:3.9" as the CRI sandbox image.
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local open5gs-k8s-vm] and IPs [10.31.0.1 172.17.42.27]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [localhost open5gs-k8s-vm] and IPs [172.17.42.27 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [localhost open5gs-k8s-vm] and IPs [172.17.42.27 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "super-admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"      
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"      
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests"
[kubelet-check] Waiting for a healthy kubelet at http://127.0.0.1:10248/healthz. This can take up to 4m0s
[kubelet-check] The kubelet is healthy after 1.013428442s
[api-check] Waiting for a healthy API server. This can take up to 4m0s 
[api-check] The API server is healthy after 17.00378595s
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node open5gs-k8s-vm as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node open5gs-k8s-vm as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule] 
[bootstrap-token] Using token: h7lo95.54r4q3qunzaknx5w
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/   

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.17.42.27:6443 --token h7lo95.54r4q3qunzaknx5w \       
        --discovery-token-ca-cert-hash sha256:30b4d389c5f602dc0d5f4d68a9500841749244e0e9942077947a2c3ad59d85b3
vagrant@open5gs-k8s-vm:~$ 
vagrant@open5gs-k8s-vm:~$
vagrant@open5gs-k8s-vm:~$ mkdir -p $HOME/.kube
e/config
sudo chown $(id -u):$(id -g) $HOME/.kube/configvagrant@open5gs-k8s-vm:~$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
vagrant@open5gs-k8s-vm:~$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
vagrant@open5gs-k8s-vm:~$ 
vagrant@open5gs-k8s-vm:~$
vagrant@open5gs-k8s-vm:~$ # Install the Tigera Operator
vagrant@open5gs-k8s-vm:~$ kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml
namespace/tigera-operator created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgpfilters.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/caliconodestatuses.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipreservations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/kubecontrollersconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/apiservers.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/imagesets.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/installations.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/tigerastatuses.operator.tigera.io created
serviceaccount/tigera-operator created
clusterrole.rbac.authorization.k8s.io/tigera-operator created
clusterrolebinding.rbac.authorization.k8s.io/tigera-operator created
deployment.apps/tigera-operator created
vagrant@open5gs-k8s-vm:~$ 
vagrant@open5gs-k8s-vm:~$
vagrant@open5gs-k8s-vm:~$ # Download the calicoctl CLI
vagrant@open5gs-k8s-vm:~$ curl -L https://github.com/projectcalico/calico/releases/download/v3.26.1/calicoctl-linux-amd64 -o calicoctl
r/local/bin/  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:-  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
  0     0    0     0    0     0      0      0 --:--:--  0:00:01 --:--:-  0 62.7M    0  207k    0     0   104k      0  0:10:16  0:00:01  0:10:15  479k^C
vagrant@open5gs-k8s-vm:~$ curl -L https://github.com/projectcalico/calico/releases/download/v3.26.1/calicoctl-linux-amd64 -o calicoctl        
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:-  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:-  0     0    0     0    0     0      0      0 --:--:--  0:00:01 --:--:-  3 62.7M    3 2432k    0     0   854k      0  0:01:15  0:00:02  0:01:1  9 62.7M    9 5919k    0     0  1538k      0  0:00:41  0:00:03  0:00:3 15 62.7M   15 10.0M    0     0  2062k      0  0:00:31  0:00:04  0:00:2 23 62.7M   23 14.6M    0     0  2564k      0  0:00:25  0:00:05  0:00:2 31 62.7M   31 20.0M    0     0  2940k      0  0:00:21  0:00:06  0:00:1 41 62.7M   41 26.1M    0     0  3411k      0  0:00:18  0:00:07  0:00:1 49 62.7M   49 31.2M    0     0  3621k      0  0:00:17  0:00:08  0:00:0 57 62.7M   57 35.9M    0     0  3736k      0  0:00:17  0:00:09  0:00:0 64 62.7M   64 40.7M    0     0  3843k      0  0:00:16  0:00:10  0:00:0 75 62.7M   75 47.0M    0     0  4067k      0  0:00:15  0:00:11  0:00:0 84 62.7M   84 53.1M    0     0  4240k      0  0:00:15  0:00:12  0:00:0 92 62.7M   92 58.2M    0     0  4309k      0  0:00:14  0:00:13  0:00:01 5526k^C
vagrant@open5gs-k8s-vm:~$ curl -L https://github.com/projectcalico/calico/releases/download/v3.26.1/calicoctl-linux-amd64 -o calicoctl        
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:-  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:-  0 62.7M    0     0    0     0      0      0 --:--:--  0:00:01 --:--:-  2 62.7M    2 1887k    0     0   806k      0  0:01:19  0:00:02  0:01:1  7 62.7M    7 4736k    0     0  1418k      0  0:00:45  0:00:03  0:00:4 13 62.7M   13 8367k    0     0  1928k      0  0:00:33  0:00:04  0:00:2 20 62.7M   20 12.5M    0     0  2406k      0  0:00:26  0:00:05  0:00:2 28 62.7M   28 17.7M    0     0  2859k      0  0:00:22  0:00:06  0:00:1 36 62.7M   36 23.1M    0     0  3230k      0  0:00:19  0:00:07  0:00:1 42 62.7M   42 26.5M    0     0  3263k      0  0:00:19  0:00:08  0:00:1 49 62.7M   49 30.8M    0     0  3378k      0  0:00:19  0:00:09  0:00:1 57 62.7M   57 35.8M    0     0  3553k      0  0:00:18  0:00:10  0:00:0 63 62.7M   63 39.7M    0     0  3591k      0  0:00:17  0:00:11  0:00:0 69 62.7M   69 43.4M    0     0  3604k      0  0:00:17  0:00:12  0:00:0 76 62.7M   76 47.9M    0     0  3677k      0  0:00:17  0:00:13  0:00:0 84 62.7M   84 53.1M    0     0  3794k      0  0:00:16  0:00:14  0:00:0 86 62.7M   86 54.3M    0     0  3625k      0  0:00:17  0:00:15  0:00:0 88 62.7M   88 55.5M    0     0  3483k      0  0:00:18  0:00:16  0:00:0 89 62.7M   89 56.2M    0     0  3306k      0  0:00:19  0:00:17  0:00:0 89 62.7M   89 56.4M    0     0  3150k      0  0:00:20  0:00:18  0:00:0 90 62.7M   90 56.7M    0     0  3005k      0  0:00:21  0:00:19  0:00:0 90 62.7M   90 57.0M    0     0  2870k      0  0:00:22  0:00:20  0:00:0 91 62.7M   91 57.2M    0     0  2747k      0  0:00:23  0:00:21  0:00:0 92 62.7M   92 58.0M    0     0  2661k      0  0:00:24  0:00:22  0:00:0 95 62.7M   95 60.0M    0     0  1091k      0  0:00:58  0:00:56  0:00:0100 62.7M  100 62.7M    0     0  1127k      0  0:00:56  0:00:56 --:--:--  161k
vagrant@open5gs-k8s-vm:~$ chmod +x ./calicoctl
vagrant@open5gs-k8s-vm:~$ sudo mv ./calicoctl /usr/local/bin/
vagrant@open5gs-k8s-vm:~$
vagrant@open5gs-k8s-vm:~$
vagrant@open5gs-k8s-vm:~$ calico-install.yaml
calico-install.yaml: command not found
vagrant@open5gs-k8s-vm:~$ nano calico-install.yaml
vagrant@open5gs-k8s-vm:~$ kubectl apply -f calico-install.yaml
installation.operator.tigera.io/default created
vagrant@open5gs-k8s-vm:~$
vagrant@open5gs-k8s-vm:~$
vagrant@open5gs-k8s-vm:~$ nano bgp-peer.yaml
vagrant@open5gs-k8s-vm:~$ sudo calicoctl apply -f bgp-peer.yaml        
Failed to create Calico API client: invalid configuration: no configuration has been provided, try setting KUBERNETES_MASTER environment variable
vagrant@open5gs-k8s-vm:~$
vagrant@open5gs-k8s-vm:~$
vagrant@open5gs-k8s-vm:~$ export KUBECONFIG=/etc/kubernetes/admin.conf 
vagrant@open5gs-k8s-vm:~$
vagrant@open5gs-k8s-vm:~$ sudo -E calicoctl apply -f bgp-peer.yaml     
Successfully applied 2 resource(s)
vagrant@open5gs-k8s-vm:~$ nano


Use "fg" to return to nano.

[1]+  Stopped                 nano
vagrant@open5gs-k8s-vm:~$ nano calico-install.yaml
vagrant@open5gs-k8s-vm:~$ kubectl apply -f calico-install.yaml
error: error loading config file "/etc/kubernetes/admin.conf": open /etc/kubernetes/admin.conf: permission denied
vagrant@open5gs-k8s-vm:~$ sudo kubectl apply -f calico-install.yaml    
error: error validating "calico-install.yaml": error validating data: failed to download openapi: Get "http://localhost:8080/openapi/v2?timeout=32s": dial tcp 127.0.0.1:8080: connect: connection refused; if you choose to ignore these errors, turn validation off with --validate=false 
vagrant@open5gs-k8s-vm:~$ sudo -E kubectl apply -f calico-install.yaml 
installation.operator.tigera.io/default configured
apiserver.operator.tigera.io/default created
vagrant@open5gs-k8s-vm:~$ nano calico-install.yaml
vagrant@open5gs-k8s-vm:~$
vagrant@open5gs-k8s-vm:~$
vagrant@open5gs-k8s-vm:~$
vagrant@open5gs-k8s-vm:~$ sudo calicoctl node status
Calico process is running.

IPv4 BGP status
+--------------+-----------+-------+----------+-------------+
| PEER ADDRESS | PEER TYPE | STATE |  SINCE   |    INFO     |
+--------------+-----------+-------+----------+-------------+
| 172.17.42.15 | global    | up    | 09:47:36 | Established |
+--------------+-----------+-------+----------+-------------+

IPv6 BGP status
No IPv6 peers found.

vagrant@open5gs-k8s-vm:~$
vagrant@open5gs-k8s-vm:~$
vagrant@open5gs-k8s-vm:~$ mkdir -p $HOME/.kube
c/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
vagrant@open5gs-k8s-vm:~$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
cp: overwrite '/home/vagrant/.kube/config'? vagrant@open5gs-k8s-vm:~$ unset KUBECONFIG
vagrant@open5gs-k8s-vm:~$
vagrant@open5gs-k8s-vm:~$
vagrant@open5gs-k8s-vm:~$ kubectl run nginx-c1 --image=nginx
pod/nginx-c1 created
vagrant@open5gs-k8s-vm:~$ kubectl taint nodes $(kubectl get nodes -o jsonpath='{.items[0].metadata.name}') node-role.kubernetes.io/control-plane-
node/open5gs-k8s-vm untainted
vagrant@open5gs-k8s-vm:~$ 
vagrant@open5gs-k8s-vm:~$ kubectl get pod -o wide 
NAME       READY   STATUS    RESTARTS   AGE     IP             NODE             NOMINATED NODE   READINESS GATES
nginx-c1   1/1     Running   0          2m21s   10.30.33.134   open5gs-k8s-vm   <none>           <none>
vagrant@open5gs-k8s-vm:~$ kubectl run busybox-c2 --image=busybox -it --rm -- /bin/sh
If you don't see a command prompt, try pressing enter.
/ # ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: seq=0 ttl=254 time=66.350 ms
64 bytes from 8.8.8.8: seq=1 ttl=254 time=51.251 ms
64 bytes from 8.8.8.8: seq=2 ttl=254 time=49.880 ms
^C
--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 49.880/55.827/66.350 ms
/ # 
/ # ping 10.30.33.134
PING 10.30.33.134 (10.30.33.134): 56 data bytes
64 bytes from 10.30.33.134: seq=0 ttl=63 time=2.634 ms
64 bytes from 10.30.33.134: seq=1 ttl=63 time=27.587 ms
^C
--- 10.30.33.134 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 2.634/15.110/27.587 ms
/ #
/ # ping 10.30.33.134
PING 10.30.33.134 (10.30.33.134): 56 data bytes
64 bytes from 10.30.33.134: seq=0 ttl=63 time=0.158 ms
64 bytes from 10.30.33.134: seq=1 ttl=63 time=0.151 ms
64 bytes from 10.30.33.134: seq=2 ttl=63 time=0.115 ms
^C
--- 10.30.33.134 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.115/0.141/0.158 ms
/ #
/ #
/ # ping 10.20.127.6
PING 10.20.127.6 (10.20.127.6): 56 data bytes
64 bytes from 10.20.127.6: seq=0 ttl=62 time=18.653 ms
64 bytes from 10.20.127.6: seq=1 ttl=62 time=3.965 ms
64 bytes from 10.20.127.6: seq=2 ttl=62 time=6.511 ms
64 bytes from 10.20.127.6: seq=3 ttl=62 time=3.715 ms
^C
--- 10.20.127.6 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 3.715/8.211/18.653 ms
/ # exit
Session ended, resume using 'kubectl attach busybox-c2 -c busybox-c2 -i -t' command when the pod is running
pod "busybox-c2" deleted
vagrant@open5gs-k8s-vm:~$ 
```



---

next -- [[open5gs setup in vagrant ]]
