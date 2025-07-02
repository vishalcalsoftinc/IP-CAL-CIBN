
---
### **Kubernetes Networking Series: Part 1 - Fundamentals & Pod Networking Basics**

#### **1. Introduction to Kubernetes Networking**

*   **Core Idea:** Networking is a fundamental and critical aspect of Kubernetes. A solid understanding is essential for managing Kubernetes clusters and developing applications for them.
* ![[Pasted image 20250627140606.png]]
*   **Three Main Network Types:**
    1.  ==**Node Network:**== This refers to the physical or virtual network that connects the Kubernetes nodes (both master and worker nodes). It's about their subnets, IP addresses, and physical connectivity. While Kubernetes doesn't manage this directly, a correctly configured node network is a prerequisite for the cluster to function at all.
    2.  ==**Cluster Network:**== This network is responsible for communication with **Services**. It allows services to be reachable from both inside and outside the cluster. This is the primary way you expose applications running in Pods to the outside world.
    3.  ==**Pod Network:**== This is the most fundamental and complex network. It handles all communication between Pods. Its main responsibility is to ensure that every Pod can communicate directly with every other Pod, whether they are on the same node or on different nodes.

---

#### **2. The OSI Model: The Foundation of Networking**

The OSI (Open Systems Interconnect) model is a standard framework for understanding how different network protocols interact to enable communication. The video focuses on a simplified 5-layer version.

*   **Why it's important:** Different Kubernetes networking solutions (like Calico or Flannel) operate at different layers (Layer 2 or Layer 3), so understanding these layers is crucial.

**The Five Layers of Networking:**

| Layer # | Layer Name    | Key Protocols | Data Unit | Addressing      | Function                                                                                                 |
| :------ | :------------ | :------------ | :-------- | :-------------- | :------------------------------------------------------------------------------------------------------- |
| **5**   | **Application** | HTTP, SMTP    | Message   | (None)          | Defines the format of the data. It's the language the sender and receiver agree to use.                    |
| **4**   | **Transport**   | TCP, UDP      | Segment   | **Port Number** | Ensures data gets to the correct application/service on a machine. Think of a port as a mailbox on a house. |
| **3**   | **Network**     | IP            | Datagram  | **IP Address**  | Responsible for routing packets from a source machine to a destination machine across different networks.      |
| **2**   | **Data Link**   | Ethernet, WiFi| Frame     | **MAC Address** | Handles point-to-point communication on the *same* local network segment (one "hop").                      |
| **1**   | **Physical**    | 10BASE-T, etc.| Bit       | (None)          | The actual physical medium like fiber optic cables, twisted-pair copper wires, or wireless signals.          |

*   **Analogy: Sending a Physical Letter**
    *   **Application Layer (Language):** You write your letter in a language your friend understands (e.g., English).
    *   **Physical Layer (Roads):** The freeways and roads that connect your city to your friend's city.
    *   **Data Link Layer (Postal Truck):** The postal truck carries the mail (the "frame") on a specific segment of the journey (e.g., from your local post office to the regional sorting facility).
    *   **Network Layer (Full Address & GPS):** Your friend's full street address (the IP address) and the routing directions (IP routing) guide the truck from start to finish across multiple segments.
    *   **Transport Layer (Mailbox):** The specific mailbox (the port) at your friend's house where the letter must be delivered.

---

#### **3. Key Network Devices (Physical & Virtual)**

These physical devices have virtual counterparts that are essential for container and VM networking.
![[Pasted image 20250627135304.png]]

1.  **Network Interface Card (NIC) / Network Adapter (Layer 2)**
    *   ==Connects a device (like a computer) to a network.==
    *   ==Has a unique, hard-coded **MAC address**.==
    *   ==Can be assigned a logical **IP address**.==
    *   **Virtual equivalent:** Virtual NICs are created for VMs and containers.

2.  **Switch (Layer 2)**
    *   Also called a "MAC bridge." It's a multi-port bridge.
    *   ==Forwards data on a *local network* using **MAC addresses**.==
    *   ==It maintains a MAC-to-port mapping table to know which device is connected to which physical port.==
    *   ==Communication between computers on the same subnet happens via the switch, without involving IP addresses for routing.==
    *   **Virtual equivalent:** A virtual switch or bridge connects virtual NICs within a host.

3.  **Router (Layer 3)**
    *   Also called the "Default Gateway."
    *   ==Forwards data packets *between different networks* (e.g., from your home network to the internet).==
    *   ==It uses **IP addresses** and maintains a routing table to decide where to send packets.==
    *   **Virtual equivalent:** Any machine with two network interfaces (real or virtual) on different subnets can act as a router.

---

#### **4. The Building Blocks of Container Networking**

Containers achieve network isolation and connectivity using two key Linux features.

1.  **Network Namespaces**
    *   This is the core concept for container network isolation.
    *   ==A namespace is a segregated "slice" of the host's networking stack.==
    *   Each namespace has its own:
        *   Network devices (interfaces, bridges).
        *   IP addresses and routing tables.
        *   Firewall rules.
        *   Port number space.
    *   **In essence, a container runs inside its own dedicated network namespace.**

2.  **Virtual Ethernet (veth) Pairs**
    *   Think of this as a **virtual patch cable**.
    *   ==It's a pair of connected virtual network interfaces.==
    *   Whatever data goes in one end of the pair comes out the other.
    *   **How it's used:** One end of the veth pair is placed inside the container's network namespace, and the other end is placed on the host (often connected to a virtual bridge). This is how a container connects to the "outside world" (the host).

---

#### **5. Practical Demo 1: Connecting "Containers" on the Same Subnet**

This demo simulates how Docker or Kubernetes connects containers on different nodes when the nodes are on the same network.
![[Pasted image 20250627141222.png]]

*   **Goal:** Establish communication between network namespaces (simulating containers) on two different VMs (simulating nodes).
*   **Setup:**
    1.  Two VMs (`ubuntu1`, `ubuntu2`) are on the same subnet.
    2.  On **each** VM, we create:
        *   Two network namespaces (`ns1`, `ns2`).
        *   A virtual bridge (`br0`).
        *   A `veth` pair for each namespace.
    3.  **Connection Steps:**
        *   One end of a `veth` pair is moved into a namespace and given an IP address (e.g., `172.16.0.2`).
        *   The other end is connected to the virtual bridge on the host.
        *   The bridge itself is given an IP address (e.g., `172.16.0.1`) to act as a gateway for the namespaces.
        *   A routing rule is added inside each namespace to send all outbound traffic to the bridge.
        *   A routing rule is added on the host to direct traffic destined for the other VM's container network to the correct VM's IP address.
*   **Result:** `ns1` on `ubuntu1` can successfully `ping` a namespace on `ubuntu2`. The traffic flows from the namespace, through its veth pair to the bridge, out the host's physical NIC, across the physical network to the other host, and then through its bridge and veth pair to the destination namespace.

---

#### **6. Practical Demo 2: The Challenge of Different Subnets & Overlay Networks**

*   **The Problem:** If the hosts (`ubuntu1`, `ubuntu2`) are on different subnets, they are separated by a router. Simple IP routing from one host to another is no longer sufficient to connect the container networks.
*   **The Solution: Overlay Networks**
    *   ==An overlay network is a **virtual network built on top of an existing physical network**.==
    *   ==It works by **encapsulating** packets.== A packet from a container (the "inner" packet) is wrapped inside another packet (the "outer" packet) that can be routed over the underlying physical network.
    *   **Common Types:**
        *   ==**Layer 2 Overlay (e.g., VXLAN used by Flannel):** Encapsulates Layer 2 Ethernet frames inside Layer 4 UDP packets.==
        *   ==**Layer 3 Overlay (e.g., IP-in-IP used by Calico):** Encapsulates Layer 3 IP packets inside another IP packet.==

*   **Demo: Creating a Simple UDP Tunnel with `socat`**
    *   **Goal:** Connect the container networks on the two VMs using a simple overlay tunnel.
    * ![[Pasted image 20250627141748.png]]
    *   **Tool:** `socat` - a powerful utility for creating bidirectional data connections.
    *   **Method:**
        1.  ==On both hosts, `socat` is used to create a **UDP tunnel**. This command sets up a listener on a specific port (e.g., 9000).==
        2.  ==This command also creates a new virtual interface called a `tun` device (tunnel device).==
        3.  ==Each end of the tunnel is given an IP address from a shared overlay network (e.g., `172.16.0.100` on host 1, `172.16.1.100` on host 2).==
        4.  ==The host's routing table is updated. Any traffic destined for the other host's container network is now sent to the `tun` interface.==
    *   **Result:** When a container on `ubuntu1` pings a container on `ubuntu2`:
        1.  The packet goes from the container to the host's bridge.
        2.  The host's routing table sees the destination is on the remote container network and sends the packet to the `tun` interface.
        3.  `socat` encapsulates this packet inside a UDP datagram and sends it across the physical network to the `socat` process on the other host.
        4.  The receiving `socat` process de-encapsulates the packet and injects it into its `tun` interface, from where it is routed to the final destination container.
        5.  We can use `t-shark` (command-line Wireshark) to monitor the traffic and verify the pings are flowing.

---

#### **7. Video Summary & Key Takeaways**

*   This session provided a foundational understanding of networking concepts crucial for Kubernetes.
*   We reviewed the **OSI model**, key **network devices**, and how traffic flows in Layer 2 (local) and Layer 3 (routed) networks.
*   The core of container networking—**network namespaces** and **veth pairs**—was explained and demonstrated.
*   We built and tested two scenarios for inter-host container communication:
    1.  Hosts on the **same subnet** using standard routing.
    2.  Hosts on **different subnets** using a simple **overlay network** (UDP tunnel).
*   These fundamental concepts are the basis for how real-world Kubernetes CNI plugins like Flannel and Calico work.