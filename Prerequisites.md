Here is the combined and enhanced study plan, structured to build your knowledge from the foundational concepts up to the specific applications.

---

### Your Consolidated 5G Project Study Plan

This plan is structured in layers, starting with the big picture, moving to the foundational platform, then the networking layer, the applications, and finally an advanced feature.

#### 1. The Big Picture: [[Understanding the End-to-End 5G Architecture]]

First, understand what the system does as a whole. Your goal here is to be able to trace the communication paths for both control signals and user data through your diagram.

*   **5G Standalone (SA) Core Concepts:** Begin with the standard 3GPP 5G architecture. Understand the shift to a Service-Based Architecture (SBA) where Network Functions (NFs) discover and communicate with each other via APIs.
*   **O-RAN CU/DU Split:** Your diagram shows a disaggregated gNB (5G base station). This is a key concept in modern RANs.
    *   **DU (Distributed Unit):** Handles real-time, lower-layer radio functions (Layers 1 & 2). It's "distributed" because it's typically closer to the antenna.
    *   **CU (Centralized Unit):** Handles non-real-time, higher-layer functions. It's "centralized" as it can serve multiple DUs. It's further split into:
        *   **CU-CP (Control Plane):** Manages control signaling (RRC protocol) for setting up and managing radio bearers.
        *   **CU-UP (User Plane):** Manages the user's data packets (SDAP/PDCP protocols).
*   **Key Components and Their Roles:**
    *   **[[RAN]] (`oai-k8s-vm`):**
        *   **OAI UE (+ rf-simulator):** Simulates the phone and radio environment.
        *   **OAI gNB (DU, CU-CP, CU-UP):** The 5G base station that provides radio connectivity.
    *   **[[5G Core]] (`open5gs-vm`):**
        *   **AMF (Access and Mobility Management Function):** The "front door" to the core network. Manages UE registration, authentication, and mobility.
        *   **UPF (User Plane Function):** The "data router." Forwards user traffic between the RAN and the Internet, enforces policies, and reports usage.
*   **Interfaces & Protocols (The Lines on the Diagram):**
    *   **F1:** Connects DU and CU. (F1-C for Control Plane over **SCTP**; F1-U for User Plane over **GTP-U**).
    *   **E1:** Connects CU-CP and CU-UP for coordination.
    *   **N2:** Control Plane link between gNB (CU-CP) and AMF over **SCTP**.
    *   **N3:** User Plane link between gNB (CU-UP) and UPF over **GTP-U**.
    *   **N6:** Connects the UPF to the external Internet.

**Study Goal:** Be able to narrate the two primary flows:
1.  **Registration (Control Plane):** UE → DU → CU-CP → (N2 Interface) → AMF.
2.  **Data Session (User Plane):** UE → DU → CU-UP → (N3 Interface) → UPF → (N6 Interface) → Internet.

---

#### 2. The Foundation: [[Kubernetes Concepts and Commands]]

Your entire 5G system lives inside Kubernetes. Mastering the basics is non-negotiable for deployment, management, and debugging.

*   **Core K8s Objects:**
    *   **Pod:** The smallest unit, running a single container (e.g., the AMF process).
    *   **Service:** A stable network endpoint (IP address and port) that exposes a set of Pods. The "service IP: Port" boxes on your diagram refer to K8s Services.
    *   **Deployment/StatefulSet:** Manages the lifecycle of Pods, ensuring the desired number are running.
    *   **ConfigMap/Secret:** How you inject configuration files (`.conf`, `.yaml`) and sensitive data into your OAI and Open5GS pods.
*   **Essential `kubectl` Commands (Your Daily Tools):**
    *   `kubectl get pods,svc -n <namespace>`: See what's running and their IPs.
    *   `kubectl describe pod <pod-name> -n <namespace>`: Your first step to find out why a pod is crashing or pending. Check the `Events` section.
    *   `kubectl logs <pod-name> -n <namespace>`: **Your most important debugging tool.** This shows the live output from the 5G component.
    *   `kubectl exec -it <pod-name> -n <namespace> -- bash`: Get a shell inside a running container to use network tools like `ping`, `traceroute`, and `tcpdump`.

---

#### 3. [[The Network Plumbing- Calico CNI and BGP Peering]]

This is how your pods communicate, both within and across clusters. The note on your diagram makes this a critical topic.

*   **CNI (Container Network Interface):** Understand that CNI is the Kubernetes standard for network plugins. Your project uses Calico.
*   **Project Calico Fundamentals:** Learn why Calico is used: high performance and powerful network policies. It achieves this by using the underlying network's routing capabilities rather than creating a virtual overlay network.
*   **BGP (Border Gateway Protocol) Basics:** You don't need to be an expert, but you must understand:
    *   **ASNs (Autonomous System Numbers):** A unique number for a network under a single administration. Each of your K8s clusters will likely have its own ASN.
    *   **BGP Peering:** The process where two BGP routers (in this case, your Calico nodes) exchange routing information.
*   **Calico's BGP Mode in Your Setup:**
    *   **Pod-to-Pod (Intra-Cluster):** Within a single cluster, Calico nodes form a "full mesh" of BGP peers. Each node advertises the routes for the pods it is hosting.
    *   **Cluster-to-Cluster (Inter-Cluster):** This is your specific use case. You will configure the Calico in `oai-k8s-vm` to establish a BGP peering session with the Calico in `open5gs-vm`. This allows them to exchange pod routes, making it possible for the CU-CP pod in one cluster to communicate directly with the AMF pod in the other cluster using their real pod IPs.
*   **Configuration and Tools:**
    *   Learn how to use the `calicoctl` command-line tool for diagnostics.
    *   Study Calico's Kubernetes Custom Resources: `BGPConfiguration` (for global settings like ASNs) and `BGPPeer` (to define the external peers to connect to).
* Links
	* [Kubernetes Networking Series - YouTube](https://www.youtube.com/playlist?list=PLSAko72nKb8QWsfPpBlsw-kOdMBD7sra-) (includes Calico and Celium)

---

#### 4. The Applications: Configuring OAI & Open5GS Components

This is where you dive into the 5G software itself. The key is ensuring all components have the correct configuration to find and trust each other.

*   **A) OpenAirInterface (RAN) Configuration:**
    *   **Focus on the `.conf` files** for each component.
    *   **oai-nr-ue:** Configure its credentials (IMSI/SUPI, Key, OPC) and the PLMN ID of the network it should try to join.
    *   **oai-du:** Must know the gNB ID, TAC, PLMN ID, and the **Service IPs** of the CU-CP and CU-UP for the F1 connections.
    *   **oai-cucp:** Must know the PLMN ID, TAC, and the **Service IP** of the AMF for the N2 connection.
    *   **oai-cuup:** Must know the **Service IP** of the UPF for the N3 connection.
    *   **Consistency is Key:** The PLMN ID and TAC must match what the 5G Core expects.
    * [5G RAN – OpenAirInterface](https://openairinterface.org/oai-5g-ran-project/)
    * [Files · develop · oai / openairinterface5G · GitLab](https://gitlab.eurecom.fr/oai/openairinterface5g)

*   **B) Open5GS (Core) and WebUI Configuration:**
    *   **Focus on the `.yaml` files** for each Network Function.
    *   **AMF (`amf.yaml`):** Configure it with the PLMN ID and TAC to serve. Must be configured to listen for N2 connections from the gNB.
    *   **SMF (`smf.yaml`):** Configure the IP address pool to be assigned to UEs and information about the UPF.
    *   **UPF (`upf.yaml`):** Configure its network interfaces for N3 (towards RAN) and N6 (towards Internet).
    *   **NRF, NSSF, etc.:** Understand their roles in service discovery and slice selection.
    *   **Open5GS WebUI (Crucial Tool):** Practice using the UI to:
        *   **Provision Subscribers:** Add new users by entering their IMSI, security keys (K, OPc), and the services/slices they are subscribed to. **These credentials must match what you configured in the `oai-nr-ue`**.
        *   **Monitor Status:** Check which UEs are registered and see the live status of the core network functions.
    * [Quickstart | Open5GS](https://open5gs.org/open5gs/docs/guide/01-quickstart/)

---

#### 5. Advanced Topic: 5G Network Slicing

This is a powerful 5G feature that you are expected to know.

*   **The Concept:** Understand that network slicing is about creating multiple, isolated, virtual end-to-end networks on a single physical infrastructure. Each slice can be tailored for a specific use case (e.g., high bandwidth vs. ultra-low latency).
*   **Key Identifier:** The **S-NSSAI** (Single Network Slice Selection Assistance Information) is the unique ID for a slice.
*   **Implementation Flow:**
    1.  **In Open5GS:** You'll define the available slices (S-NSSAIs) in the configuration files (e.g., `smf.yaml`).
    2.  **In the WebUI:** You will subscribe a user to one or more specific slices.
    3.  **In OAI:** The RAN (gNB) must be configured with the S-NSSAIs it supports.
    4.  When a UE connects, it can request a slice, and the AMF, with help from the NSSF, will select the appropriate slice and SMF to serve the UE's session.