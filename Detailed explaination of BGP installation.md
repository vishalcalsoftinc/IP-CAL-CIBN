
---
Of course. Here is a comprehensive, deeply detailed documentation created from the final corrected steps. This guide breaks down every command and parameter to explain not just *what* to do, but *why* you are doing it and how each piece connects to form the final solution.

---

## **Comprehensive Guide: Building a Multi-Cluster Kubernetes Lab with Calico BGP Peering**

### **1. High-Level Goal & Architecture**

The objective of this guide is to construct two independent Kubernetes clusters, each running on its own VirtualBox virtual machine. The key architectural choice is to forgo common overlay networks (like VXLAN or IPIP), which encapsulate traffic. Instead, we will configure the Calico CNI (Container Network Interface) to use the **Border Gateway Protocol (BGP)**.

This approach achieves **direct, non-encapsulated L3 routing** between the pods of the two clusters.

**How it Works:**
1.  Each Kubernetes cluster will have its own unique IP address range for its pods (e.g., `10.10.0.0/16` and `10.0.0.0/16`).
2.  The Calico agent on each cluster's node will act as a BGP router.
3.  We will establish a BGP "peering" session between the two clusters over a shared private network.
4.  Each cluster will "advertise" its pod network range to the other.
5.  The result: The Linux kernel on each VM will have explicit routes installed, knowing exactly how to send traffic directly to pods in the other cluster. This is highly efficient and mimics how large-scale networks operate.

**Our Lab Environment:**

*   `k8s-master-1`: Hosts Cluster 1. Node IP `172.17.42.27`. Pod CIDR `10.10.0.0/16`.
*   `k8s-master-2`: Hosts Cluster 2. Node IP `172.17.42.15`. Pod CIDR `10.0.0.0/16`.
*   `vboxnet0`: A VirtualBox Host-Only network acting as the shared "physical" network connecting our two nodes.



---

### **Part 0: The Foundation - Virtual Environment Setup**

This section prepares the virtual "data center" for our Kubernetes clusters.

#### **Section 0.1: Configure VirtualBox Network**

We need a shared network where our VMs can communicate directly. The standard `NAT` adapter is isolated per VM and will not work for this.

1.  **Open VirtualBox** -> `File` -> `Host Network Manager`.
2.  **Click `Create`**. This creates a virtual switch, `vboxnet0`.
3.  **Configure Adapter**: Ensure it has an IP like `192.168.56.1`. We won't use this IP, but it confirms the network exists for the host.
4.  **Configure DHCP Server**: **Uncheck "Enable Server"**. We need predictable, static IPs for our BGP peers, so we will assign them manually within the VMs.

#### **Section 0.2: Create and Configure Ubuntu VMs**

1.  **Create `k8s-master-1`**:
    *   **Name:** `k8s-master-1`
    *   **OS:** Ubuntu Server 20.04.6 LTS
    *   **Resources:** Minimum 2 vCPUs, 4GB RAM, 30GB Disk. Kubernetes is resource-intensive.
2.  **Configure Network Adapters for `k8s-master-1`**:
    *   **Adapter 1:** Attached to `NAT`. This is the "outbound" adapter, allowing the VM to download packages and container images from the internet.
    *   **Adapter 2:** Attached to `Host-only Adapter`, Name: `vboxnet0`. This is the "internal peering" adapter, providing the direct L2 connectivity between our VMs.
3.  **Clone `k8s-master-1` to create `k8s-master-2`**:
    *   Right-click `k8s-master-1` -> `Clone`.
    *   **Important:** Select "Reinitialize the MAC address of all network cards" to prevent network conflicts.

#### **Section 0.3: Common Configuration on BOTH VMs**

These steps prepare the Ubuntu OS on both `k8s-master-1` and `k8s-master-2`.

1.  **Grant Sudo Privileges (Optional but Recommended)**:
    ```bash
    su -                    # Switch to root user
    usermod -aG sudo user   # Add your user to the sudo group
    su user                 # Switch back to your user
    ```

2.  **Set Static IP Addresses**:
    *   We use Netplan, Ubuntu's standard network configuration tool.
    *   `sudo nano /etc/netplan/00-installer-config.yaml`

    **On `k8s-master-1`**:
    ```yaml
    network:
      ethernets:
        enp0s3: # The NAT Adapter
          dhcp4: true
        enp0s8: # The Host-Only Adapter
          dhcp4: no
          addresses: [172.17.42.27/24] # Static IP for BGP peering
      version: 2
    ```

    **On `k8s-master-2`**:
    ```yaml
    network:
      ethernets:
        enp0s3:
          dhcp4: true
        enp0s8:
          dhcp4: no
          addresses: [172.17.42.15/24] # Static IP for BGP peering
      version: 2
    ```
    *   Apply the configuration: `sudo netplan apply`.
    *   **Verify**: From `k8s-master-1`, run `ping 172.17.42.15`. A successful reply confirms our shared network is working.

3.  **Prepare for Kubernetes Installation**:
    *   **Disable Swap**: The `kubelet` (Kubernetes node agent) requires swap to be disabled for performance predictability and correct resource accounting.
        ```bash
        sudo swapoff -a
        sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
        ```
    *   **Enable Kernel Modules & Sysctl**: We need to configure the Linux kernel to handle container networking traffic correctly.
        ```bash
        # Load necessary kernel modules on boot
        cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
        overlay
        br_netfilter
        EOF

        # Load them now
        sudo modprobe overlay
        sudo modprobe br_netfilter

        # Set sysctl parameters for Kubernetes networking
        cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
        net.bridge.bridge-nf-call-ip6tables = 1
        net.bridge.bridge-nf-call-iptables = 1
        net.ipv4.ip_forward = 1
        EOF

        # Apply sysctl settings without reboot
        sudo sysctl --system
        ```
        *   `br_netfilter`: Allows `iptables` rules (used by Kubernetes for network policies and services) to see traffic that passes through a Linux bridge.
        *   `net.ipv4.ip_forward = 1`: The master switch that turns the Linux machine into a router, which is essential for Calico in BGP mode.

    *   **Install `containerd` (Container Runtime)**: This is the engine that actually runs the containers.
        ```bash
        # Install dependencies
        sudo apt-get update
        sudo apt install apt-transport-https curl -y
        # Add Docker's official GPG key and repository for containerd
        sudo mkdir -p /etc/apt/keyrings
        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
        echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
        sudo apt-get update
        sudo apt-get install containerd.io -y

        # Generate default containerd config
        sudo mkdir -p /etc/containerd
        sudo containerd config default | sudo tee /etc/containerd/config.toml
        # Set the Cgroup driver to systemd to match what kubelet expects
        sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
        # Restart containerd to apply changes
        sudo systemctl restart containerd
        ```

    *   **Install Kubernetes Components**:
        ```bash
        # Add Kubernetes GPG key and repository
        sudo apt-get update
        sudo apt-get install -y apt-transport-https ca-certificates curl gpg
        curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
        echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
        sudo apt-get update
        sudo apt-get install -y kubelet kubeadm kubectl
        # Prevent these packages from being accidentally upgraded
        sudo apt-mark hold kubelet kubeadm kubectl
        # Enable the kubelet service to start on boot
        sudo systemctl enable --now kubelet
        ```
        *   `kubeadm`: The tool to bootstrap a Kubernetes cluster.
        *   `kubelet`: The agent that runs on each node, manages pods, and communicates with the control plane.
        *   `kubectl`: The command-line tool to interact with any Kubernetes cluster.

---

### **Part 1: Initialize Kubernetes Clusters**

Now we create the two distinct clusters.

#### **On `k8s-master-1`**:
```bash
sudo kubeadm init \
--pod-network-cidr 10.10.0.0/16 \
--service-cidr 10.30.0.0/16 \
--service-dns-domain open5gs.local \
--apiserver-advertise-address 172.17.42.27
```
*   `--pod-network-cidr`: **Crucial.** Defines the unique IP range for all Pods in this cluster. Must not overlap with the other cluster.
*   `--service-cidr`: Defines the unique IP range for Kubernetes Services (like `ClusterIP`).
*   `--service-dns-domain`: Sets a custom internal DNS name.
*   `--apiserver-advertise-address`: **Crucial.** The IP address the API server will be reachable on. We use the Host-Only adapter's IP so the other cluster can reach it.

#### **On `k8s-master-2`**:
```bash
sudo kubeadm init \
--pod-network-cidr 10.0.0.0/16 \
--service-cidr 10.20.0.0/16 \
--service-dns-domain oran.local \
--apiserver-advertise-address 172.17.42.15
```
*   Notice the different CIDRs and advertise address, making this a separate, unique cluster.

#### **Post-Initialization Steps (on BOTH VMs)**

These commands configure `kubectl` for your user account.
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### **Untaint the Control-Plane Nodes (on BOTH VMs)**

By default, Kubernetes doesn't schedule user workloads on control-plane (master) nodes. Since we are running a simple single-node cluster lab, we remove this restriction (called a "taint").
```bash
# On k8s-master-1, run this:
kubectl taint nodes k8s-master-1 node-role.kubernetes.io/control-plane-

# On k8s-master-2, run this:
kubectl taint nodes k8s-master-2 node-role.kubernetes.io/control-plane-
```
*Note the corrected taint key and the trailing `-` which signifies removal.*

---

### **Part 2: Install and Configure Calico**

This section installs the network layer.

#### **Section 2.1: Install Tigera Operator (on BOTH VMs)**

The Tigera Operator is a "robot administrator" that manages the Calico installation. We install it first, and then give it a custom configuration to apply.
```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml
```

#### **Section 2.2: Download `calicoctl` (on BOTH VMs)**

This is a specialized command-line tool for managing Calico-specific resources like BGP configurations.
```bash
curl -L https://github.com/projectcalico/calico/releases/download/v3.26.1/calicoctl-linux-amd64 -o calicoctl
chmod +x ./calicoctl
sudo mv ./calicoctl /usr/local/bin/
```

#### **Section 2.3: Install Calico in BGP Mode (on BOTH VMs)**

Now we provide the custom configuration to the operator we just installed.

**On `k8s-master-1`**, create `calico-custom-c1.yaml`:
```yaml
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  calicoNetwork:
    ipPools:
    - blockSize: 26
      cidr: 10.10.0.0/16      # Must match kubeadm's --pod-network-cidr
      encapsulation: None      # CRITICAL: Disables overlays, forces BGP mode
      natOutgoing: Enabled     # Allows pods to reach the internet
      nodeSelector: all()
---
apiVersion: operator.tigera.io/v1
kind: APIServer
metadata:
  name: default
spec: {} # Enables the Calico API for calicoctl
```
Apply it: `kubectl apply -f calico-custom-c1.yaml`

**On `k8s-master-2`**, create `calico-custom-c2.yaml` with its unique CIDR:
```yaml
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  calicoNetwork:
    ipPools:
    - blockSize: 26
      cidr: 10.0.0.0/16       # Must match kubeadm's --pod-network-cidr
      encapsulation: None
      natOutgoing: Enabled
      nodeSelector: all()
---
apiVersion: operator.tigera.io/v1
kind: APIServer
metadata:
  name: default
spec: {}
```
Apply it: `kubectl apply -f calico-custom-c2.yaml`

---

### **Part 3: Configure BGP Peering**

This is the final step where we explicitly connect the two clusters at the network level.

***Correction Note:*** *The original BGP configurations had an inconsistent API version. The standard is `projectcalico.org/v3`. We will use this consistently.*

#### **On `k8s-master-1`**, create `bgp-config-c1.yaml`:
```yaml
# This is the global BGP configuration for this cluster
apiVersion: projectcalico.org/v3
kind: BGPConfiguration
metadata:
  name: default
spec:
  asNumber: 64512             # A unique ID for Cluster 1
  nodeToNodeMeshEnabled: false  # CRITICAL: We define peers manually
---
# This defines the connection to the OTHER cluster
apiVersion: projectcalico.org/v3
kind: BGPPeer
metadata:
  name: peer-to-cluster2
spec:
  peerIP: 172.17.42.15          # The IP of k8s-master-2
  asNumber: 64513               # The ASN of Cluster 2
```
Apply it: `calicoctl apply -f bgp-config-c1.yaml`

#### **On `k8s-master-2`**, create `bgp-config-c2.yaml`:
```yaml
apiVersion: projectcalico.org/v3
kind: BGPConfiguration
metadata:
  name: default
spec:
  asNumber: 64513             # A unique ID for Cluster 2
  nodeToNodeMeshEnabled: false
---
apiVersion: projectcalico.org/v3
kind: BGPPeer
metadata:
  name: peer-to-cluster1
spec:
  peerIP: 172.17.42.27          # The IP of k8s-master-1
  asNumber: 64512               # The ASN of Cluster 1
```
Apply it: `calicoctl apply -f bgp-config-c2.yaml`

---

### **Part 4: Verification and Testing**

This is the moment of truth.

#### **Section 4.1: Verify BGP Peering Status**

Run this on either VM. It checks if the BGP session is active.
```bash
sudo calicoctl node status
```
**Expected Output:**
```
Calico process is running.

IPv4 BGP status
+----------------+-------------------+-------+------------+-------------+
|  PEER ADDRESS  |     PEER TYPE     | STATE |   SINCE    |    INFO     |
+----------------+-------------------+-------+------------+-------------+
| 172.17.42.15   | global            | up    | <some_time>| Established |
+----------------+-------------------+-------+------------+-------------+
```
*   **STATE: up** and **INFO: Established** means success! The two clusters are talking BGP and have exchanged routes.
*   If `Idle` or `Active`, double-check the IPs and ASNs in your `bgp-config` files and ensure there is no firewall blocking TCP port 179 between the VMs.

#### **Section 4.2: Test Pod-to-Pod Communication**

The ultimate test.

1.  **On `k8s-master-1`**, deploy a test server pod:
    ```bash
    kubectl run nginx-c1 --image=nginx
    ```
    Get its IP address (it will be in the `10.10.x.x` range):
    ```bash
    kubectl get pod nginx-c1 -o wide
    # Example Output: NAME       ... IP            NODE
    #               nginx-c1   ... 10.10.180.193 k8s-master-1
    ```

2.  **On `k8s-master-2`**, deploy a temporary client pod to test from:
    ```bash
    kubectl run busybox-c2 --image=busybox -it --rm -- /bin/sh
    ```

3.  **From inside the `busybox-c2` shell**, ping the pod in the other cluster using its direct IP:
    ```sh
    # ping <IP_of_nginx-c1>
    ping 10.10.180.193
    ```

**If you see `PING` replies, congratulations!** You have successfully built two Kubernetes clusters that can communicate directly at the network layer, without any encapsulation, thanks to Calico and BGP.