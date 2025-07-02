
---
### **Kubernetes Networking Series: Part 2 - Pod Networking, CNI & Flannel**

#### **1. Overview of the Kubernetes Networking Model**

Building on the previous video, this section outlines the fundamental rules Kubernetes imposes on any networking solution.

*   **The Three Core Networking Rules:**
    1.  **All Pods Can Communicate with All Other Pods:** Every pod must be able to reach every other pod directly, without needing Network Address Translation (NAT).
    2.  **All Nodes Can Communicate with All Pods:** Every node in the cluster must be able to reach every pod (and vice-versa), again without NAT. (The video refers to "agents on a node," which primarily means Kubernetes system services running on the node like `kubelet`).
    3.  **A Pod's IP is its "Real" IP:** The IP address that a pod sees for itself is the same IP address that other pods and nodes see it as.

*   **The Rationale Behind These Rules:**
    *   **Simplicity:** Creates a clean, flat network model that is easy for developers to understand. Applications can be moved from VMs to containers with minimal changes to their networking configuration.
    *   **Hides Implementation Details:** Application developers don't need to worry about the complex underlying networking (like overlays or tunnels).
    *   **Service Discovery:** Simplifies how services find and communicate with each other.

*   **Review of the Three Network Types:**

| Network Type      | Purpose                                    | How IPs are Assigned                                                                                             |
| :---------------- | :----------------------------------------- | :--------------------------------------------------------------------------------------------------------------- |
| **Node Network**  | Connects the physical/virtual cluster nodes. | From the underlying network, usually via DHCP or static IP assignment.                                           |
| **Pod Network**   | Connects all the pods across all nodes.    | From a pre-defined IP range (CIDR block), managed and assigned by the **CNI plugin** (e.g., Flannel, Calico).       |
| **Cluster Network** | Used exclusively by Kubernetes **Services**. | From a separate IP range (`service-cluster-ip-range`) configured in the Kubernetes API Server and Controller Manager. |

---

#### **2. How Pods Communicate**

This section breaks down communication at different levels within the cluster.
![[Pasted image 20250627142808.png]]
*   **1. Intra-Pod Communication (Containers in the same Pod)**
    *   ==All containers within a single pod share the **same network namespace** and the **same IP address**.==
    *   They can communicate with each other using the `localhost` interface (e.g., `localhost:8080`).

*   **2. Inter-Pod Communication (On the *Same* Node)**
    *   ==Pods on the same node are connected to a virtual **Linux bridge**== (e.g., `cni0` or `docker0`).
    *   When Pod A wants to talk to Pod B, the traffic flows from Pod A, through its `veth` pair to the bridge, which then forwards it to Pod B's `veth` pair. This is simple Layer 2 (MAC-based) switching.

*   **3. Inter-Pod Communication (On *Different* Nodes)**
    *   This is the most complex scenario. Since the pods are on different network segments, a simple bridge is not enough.
    *   ==The most common solution is an **Overlay Network**. This creates a virtual network on top of the physical node network, making it appear as though all pods are on one large, flat network.==
    *   This is typically achieved by creating a **tunnel** (like a VXLAN tunnel) between the nodes.

---

#### **3. CNI (Container Network Interface)**

*   **What is CNI?**
    *   ==CNI is a standard and a set of specifications/libraries==, managed by the Cloud Native Computing Foundation (CNCF).
    *   It's **not** a single piece of software, but a **plugin-based architecture**.
    *   ==Its purpose is to provide a generic way for a container runtime (like Kubernetes' `kubelet`) to configure network interfaces for containers.==

*   **Why is CNI Important?**
    *   ==**Kubernetes delegates all pod networking responsibilities to the CNI plugin.**== When you install a Kubernetes cluster, you *must* also install a CNI plugin for networking to work.
    *   CNI plugins automate all the low-level tasks we did manually in the previous video:
        *   Creating `veth` pairs.
        *   Connecting pods to a bridge.
        *   Assigning IP addresses to pods from a pool.
        *   Setting up routes and tunnels for inter-node communication.

*   **Popular CNI Plugins:** Flannel, Calico, Weave Net, Cilium, etc.

---

#### **4. Deep Dive: Flannel CNI**

Flannel is one of the oldest and simplest CNI plugins.

*   **How Flannel Works:**
    1.  ==**Subnet Management:** `flanneld` (the Flannel daemon) runs on each node. It communicates with the Kubernetes API server to assign a unique, non-overlapping subnet (a block of IP addresses) to each node in the cluster.==
    2.  ==**Intra-Node Communication:** For pods on the *same node*, Flannel uses a standard Linux bridge named `cni0`.==
    3.  ==**Inter-Node Communication:** Flannel's default backend is **VXLAN (Virtual Extensible LAN)**.==
        *   ==**VXLAN in simple terms:** It creates an overlay network by encapsulating Layer 2 Ethernet frames inside Layer 4 UDP packets.==
        *   ==This creates a **UDP tunnel** between all nodes in the cluster, allowing pods to communicate across nodes as if they were on the same local network.==
        *   ==Flannel creates a special virtual device on each node, `flannel.1`, which represents the gateway to this overlay network.==

*   **Visualizing Flannel Traffic Flow (Pod A on Node 1 -> Pod B on Node 2):**
    1.  Pod A sends a packet destined for Pod B.
    2.  The packet leaves Pod A and hits the `cni0` bridge on Node 1.
    3.  The bridge sees the destination IP is not local and consults the node's routing table.
    4.  The routing table, configured by Flannel, directs the packet to the `flannel.1` interface.
    5.  The `flannel.1` interface encapsulates the entire original packet (with Pod A and Pod B's IPs) inside a new UDP packet.
    6.  This new UDP packet is sent across the physical node network from Node 1 to Node 2.
    7.  Node 2 receives the UDP packet, its `flannel.1` interface de-encapsulates it, retrieving the original packet.
    8.  The original packet is then sent to Node 2's `cni0` bridge, which forwards it to Pod B.
    9. ![[Pasted image 20250627144345.png]]

---

#### **5. Practical Demo: Verifying and Analyzing Flannel**

This section uses command-line tools to prove the theory.

*   **Part 1: Verifying Flannel's Configuration**
    *   **`ip link show type veth`**: Shows the `veth` pairs connecting pods to the `cni0` bridge.
    *   **`ip link show type bridge`**: Confirms the existence and "up" status of the `cni0` bridge.
    *   **`kubectl get pods -o wide`**: Shows the IP address assigned to each pod from the subnet allocated to its node.
    *   **`ip route`**: **This is the most revealing command.** It shows the routing table Flannel has created on the node:
        *   `default via 192.168.0.1...`: The node's default gateway for external traffic.
        *   `10.244.0.0/24 via 10.244.0.0 dev flannel.1`: **Key Route!** Traffic to the pod subnet on the *other* node (`10.244.0.0/24`) is sent to the `flannel.1` interface to be tunneled.
        *   `10.244.1.0/24 dev cni0...`: Traffic to the pod subnet on the *local* node (`10.244.1.0/24`) is sent directly to the `cni0` bridge.

*   **Part 2: Analyzing Live Traffic with `t-shark`**
    *   **Goal:** To capture and inspect a packet traveling through the VXLAN tunnel.
    *   **Setup:**
        1.  SSH into a pod on one node.
        2.  Run `t-shark` on the host to listen for traffic on the VXLAN port (`8472`).
        3.  From the pod, `curl` a service running in a pod on the *other* node.
    *   **Decoding the Packet Capture:** The capture reveals the layers of encapsulation.

        *   **Outer Packet (The Underlay Network - Sent between Nodes)**
            *   **L2 - Ethernet:** Host 1's MAC -> Host 2's MAC.
            *   **L3 - IP:** Host 1's IP -> Host 2's IP.
            *   **L4 - UDP:** Source Port (random) -> Destination Port **8472** (the VXLAN port).

        *   **VXLAN Header:** Contains metadata for the overlay network.

        *   **Inner Packet (The Overlay Network - The Original Pod Traffic)**
            *   **L2 - Ethernet:** Pod A's virtual MAC -> Pod B's virtual MAC.
            *   **L3 - IP:** Pod A's IP -> Pod B's IP.
            *   **L4 - TCP:** Source Port (random) -> Destination Port **8080** (the application's port).

This clearly shows how Flannel "hides" the pod-to-pod communication inside a simple node-to-node UDP packet, allowing it to cross different network segments.