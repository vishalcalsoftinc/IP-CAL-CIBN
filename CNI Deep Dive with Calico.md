---
aliases:
  - Calico
---

---
### **Kubernetes Networking Series: Part 3 - CNI Deep Dive with Calico**

#### **1. Quick Recap: How CNI Plugins Work**

This section provides a high-level review of the CNI (Container Network Interface) workflow, setting the stage for Calico.

*   **Goal of CNI:** ==To standardize and automate the "network plumbing" for containers, eliminating repetitive manual setup.==
*   **The Workflow:**![[Pasted image 20250627155012.png]]
    1.  ==A user requests a new pod via the Kubernetes API server.==
    2.  ==The `kubelet` on a chosen node receives the request.==
    3.  ==`Kubelet` calls the configured **CNI plugin** (in this case, Calico).==
    4.  ==The CNI plugin executes a series of actions:==
        *   ==Creates a new dedicated **network namespace** for the pod.==
        *   ==Creates a **virtual ethernet (`veth`) pair**.==
        *   ==Moves one end of the `veth` pair into the pod's namespace and leaves the other end in the host's root namespace.==
        *   ==Assigns an **IP address** to the pod's interface from a pre-defined pool.==
        *   ==Sets up a **default route** inside the pod (so it knows how to send traffic out).==
        *   ==Adds a **route on the host** so the host knows how to send traffic *to* the pod.==

---

#### **2. Introduction to Project Calico**

==Calico is a popular, high-performance CNI provider known for its scalability and network policy features.==

*   **Installation:** Calico is typically installed by applying a YAML manifest to a running cluster (`kubectl apply -f <calico.yaml>`).
*   **Subnet Allocation:** ==Calico assigns a block of IP addresses (a `/26` subnet by default) to each node. The node is then responsible for assigning individual IPs from that block to the pods it runs.==
*   **Default Encapsulation: IP-in-IP**![[Pasted image 20250627155413.png]]
    *   This is Calico's default method for inter-node communication.
    *   ==It's a Layer 3 overlay protocol.==
    *   ==**How it works:** It takes the original IP packet from the source pod and wraps it inside a *new, outer* IP header.==
        *   ==**Outer IP Header:** Source IP = Host Node 1, Destination IP = Host Node 2.==
        *   ==**Inner IP Header:** Source IP = Pod A, Destination IP = Pod B.==
    *   This allows pod traffic to be routed across the underlying node network, even if the nodes are on different subnets.

*   **Key Technology: BGP (Border Gateway Protocol)**
    *   ==Calico uses BGP to share and exchange routing information between nodes.==
    *   BGP is the same protocol that powers the global internet, designed for exchanging routing information between different networks (or in this case, between nodes acting as virtual routers).
    *   ==This is how Node 1 learns that to reach Pod B's IP address, it needs to send the packet to Node 2.==

---

#### **3. Calico in Action: A Practical Demo**

This section explores a live Calico installation to see how it sets up the network.

*   **Key Command-Line Tools:** `ip addr`, `ip link`, `ip route`, `kubectl`.
*   **Discoveries from the Demo:**
    *   **Interfaces (`ip addr`):**
        *   You see the standard host interfaces (`lo`, `eth0`).
        *   You see a `veth` interface for each pod running on the node, prefixed with `cali...`. These have **no IP address** on the host side; they are just a "pipe" to the pod.
        *   You see a **`tunnel0`** interface. This is the virtual interface used for the **IP-in-IP tunnel**. It has an IP address from the pod CIDR range.
    *   **Routes (`ip route`):**
        *   **Local Pods:** There is a specific route for each pod on the local node. For example: `172.16.94.5 dev cali...` This means traffic to that pod IP goes directly to its specific `veth` interface. **Notice: Calico does not use a bridge like Flannel.** It creates direct routes, which is more efficient.
        *   **Remote Pods:** There is a route for pods on other nodes. For example: `172.16.200.6 via 192.168.0.40 dev tunnel0 proto bird`.
            *   **`172.16.200.6`**: The IP of the pod on the remote node.
            *   **`via 192.168.0.40`**: The IP of the remote node.
            *   **`dev tunnel0`**: The traffic will be sent through the IP-in-IP tunnel.
            *   ==**`proto bird`**: The route was learned via the BGP protocol (Bird is the BGP agent Calico uses).==

---

#### **4. The Components of Calico**

When you install Calico, it deploys a few key components as pods/daemons on each node.

*   ==**Felix:** This is the primary Calico agent.==
    *   Its main job is to program the **routing tables** and **ACLs (firewall rules)** on the host to provide the desired connectivity.
    *   When a new pod is created, Felix adds the necessary routes to the Linux kernel.
*   ==**Bird:** This is the **BGP client/agent**.==
    *   It's responsible for exchanging routing information with the Bird agents on other nodes.
    *   When Felix adds a new route for a local pod, it informs Bird. Bird then advertises this new route to all other nodes in the cluster via BGP.

*   **The Workflow for Adding a New Pod:**
    1.  ==A new pod is created on Node 1.==
    2.  ==`Kubelet` calls the Calico CNI plugin, which sets up the pod's namespace, veth pair, and IP.==
    3.  ==The CNI plugin notifies **Felix** on Node 1.==
    4.  ==Felix programs the Linux kernel on Node 1 with a route to the new pod's IP via its `veth` interface.==
    5.  ==Felix tells **Bird** on Node 1 about this new route.==
    6.  ==Bird on Node 1 uses BGP to advertise the new route to Bird on Node 2.==
    7.  ==Bird on Node 2 tells Felix on Node 2 about the new remote route.==
    8.  ==Felix on Node 2 programs the Linux kernel with a route to the new pod's IP via the `tunnel0` interface, pointing to Node 1.==

---

#### **5. Calico's BGP and Networking Options (Advanced)**

Calico is highly configurable and offers different networking modes for different environments.

*   **BGP Peering Modes:**
    *   **Full Mesh (Default):** Every node establishes a direct BGP connection with every other node. This is simple and works well for small to medium clusters. It becomes inefficient for very large clusters (e.g., >100 nodes).
    *   **Route Reflectors:** For large clusters, you can designate a few nodes as "route reflectors." Other nodes only peer with the reflectors, not with every single node. The reflectors then share the routes among themselves. This significantly reduces the number of BGP connections.
    *   **Peering with Physical Infrastructure:** In on-premise data centers, you can configure Calico to peer directly with your physical top-of-rack (ToR) switches. This allows the physical network to know about pod IPs, making them directly routable without any overlay.

*   **Networking/Encapsulation Modes:**
    *   You can change the networking mode by editing the `ippools` resource in Calico.
    *   ==**`ipipMode: Always` (Default):**== Full overlay using IP-in-IP. Use this when nodes are on different L2 subnets and you can't peer with the physical network.
    *   ==**`ipipMode: Never` (Direct/No Overlay):**== **Most performant option.** Disables the overlay. This is ideal when all your nodes are on the same L2 subnet (e.g., a flat data center network). Calico uses BGP to exchange routes, but traffic is sent directly without encapsulation.
    *   ==**`ipipMode: CrossSubnet`:**== A hybrid mode. It uses no overlay for traffic between nodes on the *same* subnet but automatically enables IP-in-IP for traffic that needs to cross to a *different* subnet.
    *   ==**`vxlanMode: Always`:**== An alternative to IP-in-IP. VXLAN is a Layer 2 overlay that may be required in some cloud environments (like Azure) that block IP-in-IP traffic.

*   **Changing the Mode (Demo):**
    1.  Install `calicoctl` (the Calico command-line tool).
    2.  Get the default IP pool configuration: `calicoctl get ippool default-ipv4-pool -o yaml > pool.yaml`.
    3.  Edit `pool.yaml` and change `ipipMode` from `Always` to `Never`.
    4.  Apply the change: `calicoctl apply -f pool.yaml`.
    5.  **Result:** Watching the `ip route` table shows that the route to remote pods now goes directly via the `eth0` interface instead of the `tunnel0` interface, confirming the overlay is disabled. A `t-shark` capture confirms there is no longer an outer IP-in-IP header.

[[Calico handson]]
[[Calico handson in WSL]]


