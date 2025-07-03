
---
Here is a comprehensive, step-by-step guide designed for implementation on VirtualBox Ubuntu VMs, including all the necessary prerequisites and detailed explanations.

### **Conceptual Overview**

*   **Goal:** ==To create two separate Kubernetes clusters in two separate VMs. These clusters will not use an overlay network (like VXLAN). Instead, they will use the Border Gateway Protocol (BGP) to advertise their Pod and Service IP address ranges directly to each other. This allows for direct, routable, and high-performance communication between pods across the clusters.==
*   **Environment:**
    *   **VirtualBox:** To host our VMs.
    *   **Host-only Network:** A private network in VirtualBox that allows VMs to communicate with each other and with the host machine.
    *   **VM 1 (k8s-master-1):** Will host Cluster 1 (the "Open5GS" cluster).
    *   **VM 2 (k8s-master-2):** Will host Cluster 2 (the "RAN" cluster).

---

### **Part 0: Prerequisites - Setting up the Virtual Environment**

This part is crucial and covers everything needed before you can even start with `kubeadm`.

#### **Section 0.1: Configure VirtualBox Network**

1.  Open VirtualBox.
2.  Go to `File` -> `Host Network Manager`.
3.  Click `Create`. A new network adapter, probably named `vboxnet0`, will appear.
4.  Select the new adapter and ensure the "Adapter" tab has an IPv4 address like `192.168.56.1`. We won't use this IP directly, but it confirms the network exists.
5.  Go to the "DHCP Server" tab and **uncheck** "Enable Server". We will assign static IPs to our VMs.

#### **Section 0.2: Create and Configure Ubuntu VMs**

Create two identical VMs.

1.  **Create VM:**
    *   Click `New`.
    *   **Name:** `k8s-master-1`
    *   **Type:** `Linux`
    *   **Version:** `Ubuntu (64-bit)`
    *   **Memory:** At least 4GB RAM.
    *   **CPU:** At least 2 vCPUs.
    *   **Hard Disk:** 30GB is sufficient.
    *   Install **Ubuntu Server 22.04 LTS**.

2.  **Configure VM Network Adapters:**
    *   Select the `k8s-master-1` VM and go to `Settings` -> `Network`.
    *   **Adapter 1:** Should be `Enabled`, Attached to: `NAT`. This provides internet access to download packages.
    *   **Adapter 2:** `Enable Network Adapter`, Attached to: `Host-only Adapter`, Name: `vboxnet0` (or whatever you created).

3.  **Clone the VM:**
    *   Right-click `k8s-master-1` and choose `Clone`.
    *   Name the clone `k8s-master-2`.
    *   Check "Reinitialize the MAC address of all network cards".

Now you have two VMs. Start both.

#### **Section 0.3: Common Configuration on BOTH VMs**

Perform these steps on **both** `k8s-master-1` and `k8s-master-2`.
1.  Add current user to sudo
	```bash
	su -
	usermod -aG sudo user
	su user
	```

2.  **Set Static IP Addresses:**
    *   Find the interface name for the host-only network. It's usually `enp0s8`. Run `ip a` to confirm.
    *   Edit the netplan configuration file:
        ```bash
        sudo nano /etc/netplan/00-installer-config.yaml
        ```
    *   Modify it to look like this. **Use the correct IP for each VM!**

    **On `k8s-master-1`:**
    ```yaml
    # This is the network config written by 'subiquity'
    network:
      ethernets:
        enp0s3: # The NAT interface
          dhcp4: true
        enp0s8: # The Host-only interface
          dhcp4: no
          addresses:
            - 172.17.42.27/24 # IP from your guide
      version: 2
    ```

    **On `k8s-master-2`:**
    ```yaml
    # This is the network config written by 'subiquity'
    network:
      ethernets:
        enp0s3: # The NAT interface
          dhcp4: true
        enp0s8: # The Host-only interface
          dhcp4: no
          addresses:
            - 172.17.42.15/24 # IP from your guide
      version: 2
    ```

    *   Apply the new network configuration:
        ```bash
        sudo netplan apply
        ```
    *   Verify by pinging between the VMs: From `k8s-master-1`, run `ping 172.17.42.15`. It should work.

3.  **Prepare for Kubernetes Installation (on both VMs):**
    *   Disable swap (required by kubelet).
        ```bash
        sudo swapoff -a
        sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
        ```
    * Enable kernel modules and configure sysctl for container networking.
        ```bash
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
        ```
    *   Install `containerd` (the container runtime).
        ```bash
        sudo apt-get update
        sudo apt install apt-transport-https curl -y
        sudo mkdir -p /etc/apt/keyrings
        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
        echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
        sudo apt-get update
        sudo apt-get install containerd.io -y
        
        sudo mkdir -p /etc/containerd
        sudo containerd config default | sudo tee /etc/containerd/config.toml
        sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
        sudo systemctl restart containerd
        
        containerd --version
        sudo systemctl status containerd
        ```
    *   Install Kubernetes components (`kubeadm`, `kubelet`, `kubectl`).
        ```bash
        sudo apt-get update
        sudo apt-get install -y apt-transport-https ca-certificates curl gpg
        curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
        echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
        sudo apt-get update
        sudo apt-get install -y kubelet kubeadm kubectl
        sudo apt-mark hold kubelet kubeadm kubectl
        sudo systemctl enable --now kubelet
        ```

---

### **Part 1: Initialize Kubernetes Clusters (Your Step 1)**

Now we follow your guide, executing the commands on the correct VM.

#### **On `k8s-master-1` (Open5GS VM):**

```bash
sudo kubeadm init \
--pod-network-cidr 10.10.0.0/16 \
--service-cidr 10.30.0.0/16 \
--service-dns-domain open5gs.local \
--apiserver-advertise-address 172.17.42.27
```

After `kubeadm init` finishes, it will print some important commands. **Run them.**
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### **On `k8s-master-2` (RAN VM):**

```bash
sudo kubeadm init \
--pod-network-cidr 10.0.0.0/16 \
--service-cidr 10.20.0.0/16 \
--service-dns-domain oran.local \
--apiserver-advertise-address 172.17.42.15
```

After it finishes, **run the same follow-up commands** to configure `kubectl` access:
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---

### **Part 2: Install Calico Operator & Custom Configuration (Your Steps 2, 3, 4)**

Perform these steps on **both** VMs.

#### **Section 2.1: Install Tigera Operator (Your Step 2)**

This operator will watch for our custom Calico configuration and apply it.
```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml
```
*Note: I've updated the URL to a specific stable version. It's generally better practice than using a branch name like `master` or `v3.30.1` (which doesn't exist).*

#### **Section 2.2: Download `calicoctl` (Your Step 3)**

This CLI tool is used to manage Calico-specific resources, like BGP status.
```bash
curl -L https://github.com/projectcalico/calico/releases/download/v3.26.1/calicoctl-linux-amd64 -o calicoctl
chmod +x ./calicoctl
sudo mv ./calicoctl /usr/local/bin/
```

#### **Section 2.3: Install Calico in BGP Mode (Your Step 4)**

Now we create the custom configuration that tells the operator to set up Calico **without** an overlay.

**On `k8s-master-1`:**
Create a file named `calico-custom-c1.yaml`:
```bash
nano calico-custom-c1.yaml
```
Paste the following content. Note the `cidr` matches the `--pod-network-cidr` from `kubeadm init`.
```yaml
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  calicoNetwork:
    ipPools:
    - blockSize: 26
      cidr: 10.10.0.0/16  # <-- For Cluster 1
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
Apply it:
```bash
kubectl apply -f calico-custom-c1.yaml
```

**On `k8s-master-2`:**
Create a file named `calico-custom-c2.yaml`:
```bash
nano calico-custom-c2.yaml
```
Paste the following content. Note the different `cidr`.
```yaml
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  calicoNetwork:
    ipPools:
    - blockSize: 26
      cidr: 10.0.0.0/16   # <-- For Cluster 2
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
Apply it:
```bash
kubectl apply -f calico-custom-c2.yaml
```

You can watch the Calico pods come up with `watch kubectl get pods -n calico-system`.

---

### **Part 3: Configure BGP Peering (Your Step 5)**

This is where we explicitly tell each cluster about the other.

#### **On `k8s-master-1`:**

Create a file named `bgp-config-c1.yaml`:
```bash
nano bgp-config-c1.yaml
```
Paste this configuration. It sets its own ASN to `64512` and defines a peer connection to `172.17.42.15` (Cluster 2) which has ASN `64513`.
```yaml
apiVersion: crd.projectcalico.org/v1
kind: BGPConfiguration
metadata:
  name: default
spec:
  asNumber: 64512
  nodeToNodeMeshEnabled: false # IMPORTANT: We are defining peers manually
---
apiVersion: crd.projectcalico.org/v1
kind: BGPPeer
metadata:
  name: peer-to-cluster2
spec:
  peerIP: 172.17.42.15  # IP of k8s-master-2
  asNumber: 64513        # ASN of Cluster 2
```
Apply it:
```bash
calicoctl apply -f bgp-config-c1.yaml
```

#### **On `k8s-master-2`:**

Create a file named `bgp-config-c2.yaml`:
```bash
nano bgp-config-c2.yaml
```
Paste this configuration. It sets its own ASN to `64513` and peers with `172.17.42.27` (Cluster 1) which has ASN `64512`.
```yaml
apiVersion: crd.projectcalico.org/v1
kind: BGPConfiguration
metadata:
  name: default
spec:
  asNumber: 64513
  nodeToNodeMeshEnabled: false # IMPORTANT: We are defining peers manually
---
apiVersion: crd.projectcalico.org/v1
kind: BGPPeer
metadata:
  name: peer-to-cluster1
spec:
  peerIP: 172.17.42.27  # IP of k8s-master-1
  asNumber: 64512        # ASN of Cluster 1
```
Apply it:
```bash
calicoctl apply -f bgp-config-c2.yaml
```
*Note: The `IPPool` resources from your original guide are not strictly necessary here, as Calico will automatically advertise the pools created during the installation step. The `BGPConfiguration` and `BGPPeer` are the essential resources for this step.*

---

### **Part 4: Verification and Testing (Your Step 6 & Beyond)**

#### **Section 4.1: Verify BGP Peering Status**

Run this on **either** VM. You should see the peer from the other cluster.

**On `k8s-master-1`:**
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
If `STATE` is `up` and `INFO` is `Established`, it's working! If it says `Active` or `Idle`, check that the IPs and ASNs in your YAML files are correct and that there are no firewalls blocking TCP port 179 between the VMs.

#### **Section 4.2: Test Pod-to-Pod Communication**

This is the ultimate test.

1.  **On `k8s-master-1`,** create a test pod:
    ```bash
    kubectl run nginx-c1 --image=nginx
    ```
    Wait for it to be running and get its IP:
    ```bash
    kubectl get pod nginx-c1 -o wide
    # Note the IP address, it will be something like 10.10.x.x
    ```
    Let's say the IP is `10.10.123.50`.

2.  **On `k8s-master-2`,** create a temporary pod to use for testing:
    ```bash
    kubectl run busybox-c2 --image=busybox -it --rm -- /bin/sh
    ```
    This will drop you into a shell inside the pod on Cluster 2.

3.  **From inside the `busybox-c2` pod,** ping the `nginx-c1` pod on the other cluster:
    ```sh
    # Inside the busybox shell
    ping 10.10.123.50
    ```

If you see ping replies, you have successfully configured cross-cluster networking with Calico and BGP! You have direct, non-encapsulated routing between pods in separate Kubernetes clusters.