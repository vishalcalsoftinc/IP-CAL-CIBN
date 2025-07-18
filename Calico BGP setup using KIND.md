Here is a detailed guide to create a similar two-node Kubernetes setup using `kind` within a WSL environment. This guide replicates the architecture and networking principles of the Vagrant-based setup, establishing two distinct Kubernetes clusters on a single machine and enabling direct pod-to-pod communication between them using Calico and BGP.

### **Architecture Overview**

Instead of using two separate VMs, we will use `kind` (Kubernetes in Docker) to create two single-node clusters. Each "node" is a Docker container running on your WSL instance.

1.  **Host Machine:** Your PC running WSL 2 and Docker Desktop.
2.  **Cluster 1 (`oai-ran-cluster`):**
    *   **Role:** Simulates the RAN cluster.
    *   **Pod CIDR:** `10.20.0.0/16`
    *   **Service CIDR:** `10.21.0.0/16`
    *   **BGP ASN:** `64512`
3.  **Cluster 2 (`open5gs-core-cluster`):**
    *   **Role:** Simulates the 5G Core cluster.
    *   **Pod CIDR:** `10.30.0.0/16`
    *   **Service CIDR:** `10.31.0.0/16`
    *   **BGP ASN:** `64513`
4.  **Networking:**
    *   **Inter-Node:** Both `kind` node containers will be on the same Docker bridge network, allowing them to communicate directly via their container IPs.
    *   **Inter-Cluster:** Calico CNI with BGP will be configured on each cluster. They will peer with each other to advertise their pod routes, enabling direct, non-encapsulated routing between pods across the two clusters.

---

### **Part 0: Host Environment Setup (WSL)**

These steps are performed in your WSL 2 terminal.

1.  **Install/Update WSL:** Ensure you have a modern WSL 2 distribution installed (e.g., Ubuntu 20.04 or newer).

2.  **Install Docker Desktop:** Install Docker Desktop for Windows and ensure it's configured to use the WSL 2 backend. Verify it's running.
    ```bash
    # This command should execute without errors in your WSL terminal
    docker ps
    ```

3.  **Install `kubectl`:**
    ```bash
    sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl
    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
    sudo apt-get update
    sudo apt-get install -y kubectl
    ```

4.  **Install `kind`:**
    ```bash
    # For amd64 / x86_64
    [ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-amd64
    chmod +x ./kind
    sudo mv ./kind /usr/local/bin/kind
    ```

---

### **Part 1: Infrastructure Provisioning with KIND**

1.  **Create a Project Directory:**
    ```bash
    mkdir 5g-kind-lab
    cd 5g-kind-lab
    ```

2.  **Create KIND Cluster Configurations:**
    We need two configuration files to define our clusters. The key settings are `disableDefaultCNI` (so we can install Calico) and the distinct networking CIDRs.

    **For `oai-ran-cluster` (`kind-oai-config.yaml`):**
    ```yaml
    # kind-oai-config.yaml
    kind: Cluster
    apiVersion: kind.x-k8s.io/v1alpha4
    name: oai-ran-cluster
    nodes:
    - role: control-plane
    networking:
      disableDefaultCNI: true # IMPORTANT: We will install Calico manually
      podSubnet: "10.20.0.0/16"
      serviceSubnet: "10.21.0.0/16"
    ```

    **For `open5gs-core-cluster` (`kind-core-config.yaml`):**
    ```yaml
    # kind-core-config.yaml
    kind: Cluster
    apiVersion: kind.x-k8s.io/v1alpha4
    name: open5gs-core-cluster
    nodes:
    - role: control-plane
    networking:
      disableDefaultCNI: true # IMPORTANT: We will install Calico manually
      podSubnet: "10.30.0.0/16"
      serviceSubnet: "10.31.0.0/16"
    ```

3.  **Launch the Clusters:**
    Run these commands from your `5g-kind-lab` directory.
    ```bash
    kind create cluster --config kind-oai-config.yaml
    kind create cluster --config kind-core-config.yaml
    ```

4.  **Verify Cluster Creation:**
    You should now have two clusters running and two contexts configured in your `kubeconfig`.
    ```bash
    # List the kind clusters
    kind get clusters

    # List the kubectl contexts
    kubectl config get-contexts
    # You should see:
    # kind-oai-ran-cluster
    # kind-open5gs-core-cluster
    ```

5.  **Get Node Container IPs:**
    This is the most critical step for BGP peering. We need the IP addresses assigned by Docker to our `kind` "node" containers.
    ```bash
    # Get the IP for the OAI RAN cluster node
    OAI_NODE_IP=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' oai-ran-cluster-control-plane)

    # Get the IP for the Open5GS Core cluster node
    CORE_NODE_IP=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' open5gs-core-cluster-control-plane)

    echo "OAI RAN Node IP: $OAI_NODE_IP"
    echo "Open5GS Core Node IP: $CORE_NODE_IP"
    ```
    Keep these IPs handy. The rest of the guide will use these shell variables.

---

### **Part 2: Calico and BGP Setup**

For each of the following sections, you must run the commands for **both clusters** by switching the `kubectl` context.

#### **Section 2.1: Install Calico (Run for BOTH clusters)**

1.  **Install the Tigera (Calico) Operator:**
    ```bash
    # --- For OAI RAN Cluster ---
    kubectl config use-context kind-oai-ran-cluster
    kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml

    # --- For Open5GS Core Cluster ---
    kubectl config use-context kind-open5gs-core-cluster
    kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml
    ```

2.  **Create Calico `Installation` Resources:**
    These files tell the operator how to configure Calico, specifying the pod CIDR we defined in the `kind` configs.

    **For `oai-ran-cluster` (`calico-install-oai.yaml`):**
    ```yaml
    # calico-install-oai.yaml
    apiVersion: operator.tigera.io/v1
    kind: Installation
    metadata:
      name: default
    spec:
      calicoNetwork:
        ipPools:
        - blockSize: 26
          cidr: 10.20.0.0/16  # CIDR for OAI RAN cluster
          encapsulation: None # No encapsulation for direct routing
          natOutgoing: Enabled
          nodeSelector: all()
    ```

    **For `open5gs-core-cluster` (`calico-install-core.yaml`):**
    ```yaml
    # calico-install-core.yaml
    apiVersion: operator.tigera.io/v1
    kind: Installation
    metadata:
      name: default
    spec:
      calicoNetwork:
        ipPools:
        - blockSize: 26
          cidr: 10.30.0.0/16 # CIDR for Open5GS Core cluster
          encapsulation: None # No encapsulation for direct routing
          natOutgoing: Enabled
          nodeSelector: all()
    ```

3.  **Apply the Calico Installations:**
    ```bash
    # --- Apply to OAI RAN Cluster ---
    kubectl config use-context kind-oai-ran-cluster
    kubectl apply -f calico-install-oai.yaml

    # --- Apply to Open5GS Core Cluster ---
    kubectl config use-context kind-open5gs-core-cluster
    kubectl apply -f calico-install-core.yaml
    ```
    Wait a minute or two for Calico pods to start in the `calico-system` namespace on both clusters. You can check with `watch kubectl get pods -n calico-system`.

#### **Section 2.2: Configure BGP Peering**

Now we create the BGP configuration to peer the clusters together.

1.  **Create BGP Configuration YAMLs:**

    **For `oai-ran-cluster` (to peer with Core):**
    Create `bgp-oai.yaml`. This file tells the OAI cluster's Calico instance to peer with the Core cluster's node.
    ```yaml
    # bgp-oai.yaml
    apiVersion: projectcalico.org/v3
    kind: BGPConfiguration
    metadata:
      name: default
    spec:
      asNumber: 64512 # OAI RAN Cluster ASN
      nodeToNodeMeshEnabled: false # We are defining peers explicitly
    ---
    apiVersion: projectcalico.org/v3
    kind: BGPPeer
    metadata:
      name: peer-to-core-cluster
    spec:
      peerIP: __CORE_NODE_IP__   # Placeholder for Core Node IP
      asNumber: 64513          # ASN of Core Cluster
    ```

    **For `open5gs-core-cluster` (to peer with RAN):**
    Create `bgp-core.yaml`. This file tells the Core cluster's Calico instance to peer with the RAN cluster's node.
    ```yaml
    # bgp-core.yaml
    apiVersion: projectcalico.org/v3
    kind: BGPConfiguration
    metadata:
      name: default
    spec:
      asNumber: 64513 # Open5GS Core Cluster ASN
      nodeToNodeMeshEnabled: false # We are defining peers explicitly
    ---
    apiVersion: projectcalico.org/v3
    kind: BGPPeer
    metadata:
      name: peer-to-ran-cluster
    spec:
      peerIP: __OAI_NODE_IP__   # Placeholder for RAN Node IP
      asNumber: 64512          # ASN of RAN Cluster
    ```

2.  **Substitute IPs and Apply BGP Configurations:**
    We use `sed` to replace the placeholders with the actual IPs we captured earlier.
    ```bash
    # Substitute and apply for OAI Cluster
    sed -i "s/__CORE_NODE_IP__/$CORE_NODE_IP/" bgp-oai.yaml
    kubectl config use-context kind-oai-ran-cluster
    kubectl apply -f bgp-oai.yaml

    # Substitute and apply for Core Cluster
    sed -i "s/__OAI_NODE_IP__/$OAI_NODE_IP/" bgp-core.yaml
    kubectl config use-context kind-open5gs-core-cluster
    kubectl apply -f bgp-core.yaml
    ```

#### **Section 2.3: Verify BGP Peering**

1.  **Install `calicoctl`:**
    ```bash
    curl -L https://github.com/projectcalico/calico/releases/download/v3.26.1/calicoctl-linux-amd64 -o calicoctl
    chmod +x ./calicoctl
    sudo mv ./calicoctl /usr/local/bin/
    ```

2.  **Check BGP Status:**
    Exec into the `calico-node` pod on either cluster and check the status.

    ```bash
    # --- Check from the OAI RAN Cluster ---
    kubectl config use-context kind-oai-ran-cluster
    CALICO_NODE_POD=$(kubectl get pods -n calico-system -l k8s-app=calico-node -o jsonpath='{.items[0].metadata.name}')
    kubectl exec -n calico-system $CALICO_NODE_POD -- calico-node status
    ```
    The output should show the peer as **`Established`**.

    **Example Output:**
    ```
    Calico process is running.

    IPv4 BGP status
    +---------------+-----------+-------+----------+-------------+
    |  PEER ADDRESS | PEER TYPE | STATE |  SINCE   |    INFO     |
    +---------------+-----------+-------+----------+-------------+
    | 172.18.0.3    | global    | up    | 15:35:01 | Established |
    +---------------+-----------+-------+----------+-------------+
    ```
    If it shows `Active` or another state, re-check your node IPs, ASNs, and ensure there's no firewall interference.

---

### **Part 3: Test Pod-to-Pod Communication**

This is the final test to confirm everything is working.

1.  **On `oai-ran-cluster`**, create a test NGINX pod:
    ```bash
    kubectl config use-context kind-oai-ran-cluster
    kubectl run nginx-ran --image=nginx
    ```
    Wait for it to be running and get its IP address.
    ```bash
    # Wait until STATUS is 'Running'
    watch kubectl get pod nginx-ran -o wide

    # Get the IP and store it
    POD_IP_RAN=$(kubectl get pod nginx-ran -o jsonpath='{.status.podIP}')
    echo "NGINX Pod IP on RAN cluster: $POD_IP_RAN"
    # Note the IP, it will be in the 10.20.x.x range
    ```

2.  **On `open5gs-core-cluster`**, run a temporary busybox pod for testing:
    ```bash
    kubectl config use-context kind-open5gs-core-cluster
    kubectl run busybox-core --image=busybox -it --rm -- /bin/sh
    ```    This will drop you into a shell inside a pod on the core cluster.

3.  **From inside the `busybox-core` pod,** ping the `nginx-ran` pod on the other cluster using the IP you just noted.
    ```sh
    # This command is run inside the busybox pod's shell
    # Replace the IP with the one you got in step 1
    ping $POD_IP_RAN
    ```
    **Example:**
    ```sh
    / # ping 10.20.183.65
    PING 10.20.183.65 (10.20.183.65): 56 data bytes
    64 bytes from 10.20.183.65: seq=0 ttl=62 time=0.692 ms
    64 bytes from 10.20.183.65: seq=1 ttl=62 time=0.455 ms
    ...
    ```

If you see ping replies, you have successfully configured cross-cluster networking with Calico and BGP on `kind`. You have direct, non-encapsulated routing between pods in separate Kubernetes clusters running on the same host machine.

### **Cleanup**

To tear down the environment, simply delete the `kind` clusters.

```bash
kind delete cluster --name oai-ran-cluster
kind delete cluster --name open5gs-core-cluster
```