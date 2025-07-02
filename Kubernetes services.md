
---
### **Kubernetes Networking Series: Part 4 - Services, `kube-proxy`, and Service Discovery**

#### **1. Overview & Agenda**

This video builds upon the previous discussions of container and pod networking (Flannel, Calico) to explore the critical concept of Kubernetes Services.

*   **Topics Covered:**
    1.  **Kubernetes Control Plane:** A quick overview of the components that manage the cluster.
    2.  **Pod Creation Workflow:** How a `kubectl create` command results in a running pod.
    3.  **What is a Kubernetes Service?** The core concept and its purpose.
    4.  **`kube-proxy` and CoreDNS:** Key components that make services work.
    5.  **Service Types:** ClusterIP, NodePort, LoadBalancer, and special types (Headless, ExternalName).
    6.  **Service Discovery:** How pods find and communicate with services.

---

#### **2. The Kubernetes Control Plane: The Brains of the Cluster**

The control plane consists of several components, typically running on the master node(s), that manage the state of the cluster.

*   **`etcd`:** ==A consistent and highly-available key-value store. It is the **single source of truth** for the entire cluster.== All object definitions (pods, services, deployments) and their current state are stored here.
*   **API Server:** ==The front door to the control plane. All communication (from `kubectl`, controllers, etc.) goes through the API server==. It validates requests, processes them, and persists the state to `etcd`.
*   **Controller Manager:** Runs various "controllers," which are non-terminating control loops. Each controller watches for changes to specific objects (like Deployments, ReplicaSets) and works to bring the *current state* of the cluster to the *desired state*. (e.g., "I desire 3 replicas, but I only see 2, so I will create one more.").
*   **Scheduler:** Its only job is to watch for newly created pods that have no node assigned to them and then assign them to a suitable worker node based on resource availability, constraints, etc.

#### **3. Components on a Worker Node**

*   **`kubelet`:** ==The primary Kubernetes agent on each node. It communicates with the API server, watches for pods assigned to its node, and ensures their containers are running and healthy.==
*   **Container Runtime Interface (CRI):** ==A standard interface that allows `kubelet` to talk to different container runtimes== (like `containerd`, CRI-O). **Note:** Direct support for the Docker runtime is deprecated; it now uses a "shim" to be CRI-compliant.
*   **CNI Plugin (e.g., Calico, Flannel):** ==Handles all pod networking setup (creating network namespaces, veth pairs, assigning IPs, setting up routes).==
*   **`kube-proxy`:** ==A network proxy that runs on each node. Its primary job is to implement the Kubernetes Service concept by managing network rules (e.g., `iptables` or IPVS) on the host.==

---

#### **4. The [[Pod Creation Workflow]]: A Step-by-Step Walkthrough**

![[Pasted image 20250701122614.png]]

This sequence demonstrates how the control plane components work together.

1.  **User -> `kubectl`:** A user runs `kubectl create deployment hello-world --image=...`.
2.  **`kubectl` -> API Server:** `kubectl` creates a Deployment manifest and sends it to the API server.
3.  **API Server -> `etcd`:** The API server validates the request and saves the Deployment object in `etcd`. It immediately responds "deployment created."
4.  **Deployment Controller:** The Deployment controller (in the Controller Manager) is watching for Deployment objects. It sees the new one and creates a corresponding **ReplicaSet** object with a desired replica count of 1. It writes this new ReplicaSet to `etcd`.
5.  **ReplicaSet Controller:** The ReplicaSet controller sees the new ReplicaSet object. It sees "desired: 1, current: 0" and creates a **Pod** object with a `pending` status. It writes this Pod to `etcd`.
6.  **Scheduler:** The Scheduler sees the new `pending` pod with no node assigned. It picks a suitable worker node (e.g., `node1`) and updates the Pod object in `etcd` with `spec.nodeName: node1`.
7.  **`kubelet` (on `node1`):** The `kubelet` on `node1` is watching for pods assigned to it. It sees the new pod.
8.  **`kubelet` -> CRI -> Container Runtime:** `kubelet` calls the Container Runtime (via CRI) to pull the specified image (if not already present) and start the container.
9.  **`kubelet` -> CNI Plugin:** `kubelet` then calls the CNI plugin to set up the pod's network (namespace, veth pair, IP address, routes).
10. **`kubelet` -> API Server -> `etcd`:** Once the container is running and networked, `kubelet` updates the Pod's status in `etcd` to `Running`. The desired state is now reached.

---

#### **5. What is a Kubernetes Service?**

==Pods are ephemeral (they come and go), and their IP addresses change. Relying on pod IPs directly is unstable.==

*   **Definition:** ==A Service is an **abstraction** that defines a logical set of pods and a policy by which to access them.==
*   **Core Functions:**
    1.  ==**Stable Endpoint:**== It provides a single, stable virtual IP address (the ClusterIP) that does not change. Clients connect to this virtual IP instead of individual pod IPs.
    2.  **==Service Discovery:**== It allows pods to find each other using a consistent DNS name.
    3.  ==**Load Balancing:**== It automatically distributes incoming traffic across all the healthy pods that match its selector.

*   **How it Works (The Mechanics):**
    *   ==**Labels and Selectors:** A Service finds its backend pods using a **selector**, which matches the **labels** on the pods.==
    *   ==**Endpoints Object:** When a Service is created, Kubernetes automatically creates a corresponding `Endpoints` object. This object holds a list of the actual `IP:Port` pairs of all the healthy pods that match the Service's selector. It is kept up-to-date automatically.==
    *   ==**`kube-proxy`:** `kube-proxy` on each node watches for Service and Endpoints objects. It then programs network rules (typically using **`iptables`** or **IPVS**) on the node to implement the virtual IP and load balancing.==
* [[NodePort vs LoadBalancer]]

---

#### **6. Deep Dive into `iptables` and Service Implementation**

[![](https://mermaid.ink/img/pako:eNqNlG1vmzAUhf_KlaVJmZQwsCEUf6jUptnWtZ2ida9VpMkBl7CCYWC2ZlX--67DyxKVVeXTtXWeY-xz7QcS5pEknFTyZy1VKM8SEZciWyrArxClTsKkEErDLE2k0os8AlG1A8DRkLCutCzPF42wG4wc2wqmVkAtar98TJ0XWqxSWRko6epXcFev5KQo8_vNY-Ti0-n8-_XnmUFMPcF68vXbDYwqWf5KQgnhWiRqYDH8b-cfNV9MTk5nMCpwb86wmh6oz-avGzUdVrMD9Zu3542aDavdA_W7i8tG7Q6oP0iRtgmY0tAwklZsjQFPl7qu5Vgecg3ZJzY5Pu5z4HAtVQSlibvSoHPYy6XjWi1yXSwcFiK8kxrWia5AYc-MIc4xokShRR-YEmhpysapo9GoC4vDldDhGrVlnaLJjzorKvMb-wk2tEg1XOYigpVIhQoTFTfz5uvs0NmEyeFLotdAvRcmc2zjxtj47lJtOuF_OOUwTytkfhsTxiw2bEOftmEHNp49bMKeNnE7k33G3WcwvqYwG0eobQoOZ-9PPnKY4ZqxhMiki9euPwFzBQdapTWizzWiWPcUey7F9in3uZTbU618suvktqs5zlZFrirZdOIKGxRG1-gHmdiAKIp0A8ktKCkjGeFWyZjEZRIRfivMCZNMlpkwY_JgVlkSvZaZXBKOZSRvRZ3qJVmqLXJ4-27yPCNclzWSZV7H696nLiKhu4ezny0xJ1nO8lppwpnt7kwIfyD3OJxaDvW9gLLp1GdHU2dMNoQ7gWf5PnVwzvf8I89n2zH5s1vWsZhtMxr4getS2wkoEjJKdF5eNc_37hXf_gUmecgh?type=png)](https://mermaid.live/edit#pako:eNqNlG1vmzAUhf_KlaVJmZQwsCEUf6jUptnWtZ2ida9VpMkBl7CCYWC2ZlX--67DyxKVVeXTtXWeY-xz7QcS5pEknFTyZy1VKM8SEZciWyrArxClTsKkEErDLE2k0os8AlG1A8DRkLCutCzPF42wG4wc2wqmVkAtar98TJ0XWqxSWRko6epXcFev5KQo8_vNY-Ti0-n8-_XnmUFMPcF68vXbDYwqWf5KQgnhWiRqYDH8b-cfNV9MTk5nMCpwb86wmh6oz-avGzUdVrMD9Zu3542aDavdA_W7i8tG7Q6oP0iRtgmY0tAwklZsjQFPl7qu5Vgecg3ZJzY5Pu5z4HAtVQSlibvSoHPYy6XjWi1yXSwcFiK8kxrWia5AYc-MIc4xokShRR-YEmhpysapo9GoC4vDldDhGrVlnaLJjzorKvMb-wk2tEg1XOYigpVIhQoTFTfz5uvs0NmEyeFLotdAvRcmc2zjxtj47lJtOuF_OOUwTytkfhsTxiw2bEOftmEHNp49bMKeNnE7k33G3WcwvqYwG0eobQoOZ-9PPnKY4ZqxhMiki9euPwFzBQdapTWizzWiWPcUey7F9in3uZTbU618suvktqs5zlZFrirZdOIKGxRG1-gHmdiAKIp0A8ktKCkjGeFWyZjEZRIRfivMCZNMlpkwY_JgVlkSvZaZXBKOZSRvRZ3qJVmqLXJ4-27yPCNclzWSZV7H696nLiKhu4ezny0xJ1nO8lppwpnt7kwIfyD3OJxaDvW9gLLp1GdHU2dMNoQ7gWf5PnVwzvf8I89n2zH5s1vWsZhtMxr4getS2wkoEjJKdF5eNc_37hXf_gUmecgh)
*   **What is `iptables`?** The Linux firewall system that interacts with the kernel's Netfilter hooks to inspect, modify, and route network packets.
*   **Tables & Chains:** Rules are organized into tables (`filter`, `nat`, `mangle`) and processed in chains (`PREROUTING`, `INPUT`, `FORWARD`, `OUTPUT`, `POSTROUTING`).
*   **How `kube-proxy` uses `iptables`:**
    1.  When a Service is created (e.g., with ClusterIP `10.96.92.20`), `kube-proxy` adds rules to the `nat` table.
    2.  It creates a new chain, `KUBE-SERVICES`. All incoming traffic is sent here first.
    3.  Inside `KUBE-SERVICES`, a rule is created: "If the destination IP is `10.96.92.20`, jump to another chain, e.g., `KUBE-SVC-XYZ`."
    4.  Inside `KUBE-SVC-XYZ`, `kube-proxy` creates the load balancing rules. For 4 backend pods, it will look like this:
        *   Rule 1: With a 25% probability, jump to chain `KUBE-SEP-ABC` (for pod 1).
        *   Rule 2: With a 33.3% probability (of the remaining traffic), jump to chain `KUBE-SEP-DEF` (for pod 2).
        *   Rule 3: With a 50% probability, jump to chain `KUBE-SEP-GHI` (for pod 3).
        *   Rule 4: (The remainder) Jump to chain `KUBE-SEP-JKL` (for pod 4).
    5.  Inside a pod-specific chain like `KUBE-SEP-ABC`, the final rule performs **DNAT (Destination Network Address Translation)**. It changes the packet's destination IP from the virtual ClusterIP (`10.96.92.20`) to the actual pod's IP (e.g., `10.244.1.5`).
    6.  The packet is then routed to the real pod.
    7. [[DNAT & SNAT]] 

---

#### **7. Kubernetes Service Types**

1.  **ClusterIP (Default)**
    *   **Purpose:** ==Exposes the Service on an internal IP address *within the cluster*.==
    *   **Accessibility:** ==Only reachable from within the cluster.== This is the standard for internal, microservice-to-microservice communication.
    *   **Important Note:** If a pod selected for load balancing is dead, the connection will fail. You **must** use **Liveness Probes** in your pods to ensure dead pods are removed from the Endpoints list, so `kube-proxy` doesn't route traffic to them.
    * [![](https://mermaid.ink/img/pako:eNqNUrFuwjAQ_RXr5hCBnTgkQxe6VKISoltJBzc-IGpip8auoMC_1zGUSEx4evf87t2TfUeotEQoYGNEtyXzZamIP7OmRmUXWq4uiHhIarWrJZKqcTuL5uNfGaqXxeqGyBuan7pCMhnHOY_38f6q_XKf2Bm9P6x6NArweuX9J6t-yGQgaCDoQLBAME_cpSSj0dNpid8Od5ZYPYQ6DfAubt8yBLrLF_zmWkjyKRqhKjyFgA-p6EMqViqI_KPXEoq1aHYYQYumFX0Nx96iBLvFFksoPJS4Fq6xJZTq7Ps6od61bqGwxvlOo91me_NxnRQWn2vhv7S9sQaVRDPTTlkoGJsGEyiOsIeCUh5nSUITlqcsS6YZj-Dg6SmPE07zlCcpYxlLzhH8hrHjmFPuu8Z5mmeUT3gWAcraavN62aawVOc_Gd3BrA?type=png)](https://mermaid.live/edit#pako:eNqNUrFuwjAQ_RXr5hCBnTgkQxe6VKISoltJBzc-IGpip8auoMC_1zGUSEx4evf87t2TfUeotEQoYGNEtyXzZamIP7OmRmUXWq4uiHhIarWrJZKqcTuL5uNfGaqXxeqGyBuan7pCMhnHOY_38f6q_XKf2Bm9P6x6NArweuX9J6t-yGQgaCDoQLBAME_cpSSj0dNpid8Od5ZYPYQ6DfAubt8yBLrLF_zmWkjyKRqhKjyFgA-p6EMqViqI_KPXEoq1aHYYQYumFX0Nx96iBLvFFksoPJS4Fq6xJZTq7Ps6od61bqGwxvlOo91me_NxnRQWn2vhv7S9sQaVRDPTTlkoGJsGEyiOsIeCUh5nSUITlqcsS6YZj-Dg6SmPE07zlCcpYxlLzhH8hrHjmFPuu8Z5mmeUT3gWAcraavN62aawVOc_Gd3BrA)

2.  **NodePort**
    *   **Purpose:** ==Exposes the Service on a static port on each worker node's IP address.==
    *   **How it Works:** ==It automatically creates a `ClusterIP` service first, then opens a specific port (range 30000-32767 by default) on *every node*. Traffic sent to `[Any-Node-IP]:[NodePort]` is then forwarded to the internal `ClusterIP`.==
    *   **Use Case:** A simple way to expose a service to the outside world for testing or when an external load balancer is not available. The client still needs to know a node's IP, which is not ideal for production.
    * [![](https://mermaid.ink/img/pako:eNqdUz1v2zAQ_SvEzTIhUjT1MWRxOxhwAiPdKnVgrEssVBJdmgqU2P7vpahYcYwMQTgc7t67Lz6QB9joEiGDJ6N2W7K6L1rizs_eomlVvagrbG1-DskY_xmT7lwlywdLGGEpp0wmlFEWXvB85Pklzy74tTY2PzvkF5rnaoMki8IwObdZ1N3ejV-u88mbEllIU0l72r_l_u0ecGd0_5IP3sy7b9Ralyx3hrB3gHuAO-Cza5PZ7OZ4j_863Ftitd93uR53O463_14ZP8_zPYb8SYsL5a7wj5p5ctLjSihPTlJcKePXW2lVkgdVq3aDRy_Nl7Lc4hC4p1KVkD2qeo8BNGgaNcRwGFoUYLfYYAGZc0t8VF1tCyjak6vbqfa31g1k1nSu0ujuaTv16XalsvijUu4hNhNqsC3RLHTXWsgiwX0TyA7QQ8a5pLEQXETpPIpFEssAXhycSCokT-dSzKMojsQpgFc_NqSSS1cVynguopCnSQBYVlab2_EP-K9w-g_tvPWx?type=png)](https://mermaid.live/edit#pako:eNqdUz1v2zAQ_SvEzTIhUjT1MWRxOxhwAiPdKnVgrEssVBJdmgqU2P7vpahYcYwMQTgc7t67Lz6QB9joEiGDJ6N2W7K6L1rizs_eomlVvagrbG1-DskY_xmT7lwlywdLGGEpp0wmlFEWXvB85Pklzy74tTY2PzvkF5rnaoMki8IwObdZ1N3ejV-u88mbEllIU0l72r_l_u0ecGd0_5IP3sy7b9Ralyx3hrB3gHuAO-Cza5PZ7OZ4j_863Ftitd93uR53O463_14ZP8_zPYb8SYsL5a7wj5p5ctLjSihPTlJcKePXW2lVkgdVq3aDRy_Nl7Lc4hC4p1KVkD2qeo8BNGgaNcRwGFoUYLfYYAGZc0t8VF1tCyjak6vbqfa31g1k1nSu0ujuaTv16XalsvijUu4hNhNqsC3RLHTXWsgiwX0TyA7QQ8a5pLEQXETpPIpFEssAXhycSCokT-dSzKMojsQpgFc_NqSSS1cVynguopCnSQBYVlab2_EP-K9w-g_tvPWx)

3.  **LoadBalancer**
    *   **Purpose:** ==Exposes the Service externally using a cloud provider's load balancer.==
    *   **How it Works:** ==This is a superset of `NodePort`==. It creates a `NodePort` and `ClusterIP` service, and then integrates with the underlying cloud platform (AWS, GCP, Azure) to provision an external load balancer. The cloud load balancer then directs traffic to the `NodePort` on all the cluster nodes.
    *   **Use Case:** The standard, production-grade way to expose services to the internet on public clouds. This is not available on-premise without extra tooling (like MetalLB).
    * [![](https://mermaid.ink/img/pako:eNqVU8tu4yAU_RXEOkEGE_xYzKIZjRQpU1nTXe0uSKCNNTZEFDppk_x7MX40jWbReoHvvec-Dgc4wq0WEubwyfD9Dqz_VAr4b6WsNEraZVNLZcvRBb3_0CctG-3E-qYMf7DWXIAb3nC1lQYUbtPUW7AqhtxbPwWX3QowwBlBmKUIIxxd4KTHySWOL_BCG1uOBriT5qXeSpDHUZRGEyX37KmuinKypkQcoYyhAzoMuX_dRu6NPryWnTUP5gAVWuDSLwB_BEgIEB_4n0RgPv8xCvJJnQ44_dLmHzcCWB02sip60qdelm_mk5FBKA6DR1EutLyKf1ZxYDsodCVdACdxrrQK9MJRb_qjPgWxvpTlicOZv2i1gPkjb57lDLbStLzz4bFrUUG7k62sYO5NIR-5a2wFK3X2dXuu7rVuYW6N85VGu6fd1MftBbfyZ839NW6nqJFKSLPUTlmYx5SFJjA_wgPMCWEooZTQOFvECU0Tj776cMoQZSRbMLqI4ySm5xl8C2MjxAjzVRFLFjSOSJbOoBS11eZ3_4LCQzq_A6hCCEs?type=png)](https://mermaid.live/edit#pako:eNqVU8tu4yAU_RXEOkEGE_xYzKIZjRQpU1nTXe0uSKCNNTZEFDppk_x7MX40jWbReoHvvec-Dgc4wq0WEubwyfD9Dqz_VAr4b6WsNEraZVNLZcvRBb3_0CctG-3E-qYMf7DWXIAb3nC1lQYUbtPUW7AqhtxbPwWX3QowwBlBmKUIIxxd4KTHySWOL_BCG1uOBriT5qXeSpDHUZRGEyX37KmuinKypkQcoYyhAzoMuX_dRu6NPryWnTUP5gAVWuDSLwB_BEgIEB_4n0RgPv8xCvJJnQ44_dLmHzcCWB02sip60qdelm_mk5FBKA6DR1EutLyKf1ZxYDsodCVdACdxrrQK9MJRb_qjPgWxvpTlicOZv2i1gPkjb57lDLbStLzz4bFrUUG7k62sYO5NIR-5a2wFK3X2dXuu7rVuYW6N85VGu6fd1MftBbfyZ839NW6nqJFKSLPUTlmYx5SFJjA_wgPMCWEooZTQOFvECU0Tj776cMoQZSRbMLqI4ySm5xl8C2MjxAjzVRFLFjSOSJbOoBS11eZ3_4LCQzq_A6hCCEs)

---

#### **8. Special Service Types**

1.  **Headless Service**
    *   **How to create:** Set `clusterIP: None` in the Service spec.
    *   **What it does:** It does **not** create a virtual ClusterIP and does **not** do any load balancing via `kube-proxy`.
    *   **Its Purpose:** Purely for **service discovery**. When you perform a DNS lookup for a Headless Service, instead of returning a single IP, the DNS server returns a list of **all the individual IP addresses** of the pods backing the service.
    *   **Use Case:** Used for stateful applications (like databases) where you need to connect to a *specific* pod instance (e.g., the primary replica) rather than a random one.

2.  **Services without Selectors / ExternalName**
    *   **Service without Selector:** You can create a Service object without a `selector`. This means Kubernetes won't create an `Endpoints` object automatically. You can then manually create an `Endpoints` object and point it to any IP address you want, even one outside the cluster (e.g., an external database).
    *   **ExternalName Service:** A special type (`type: ExternalName`) that acts as a DNS CNAME record. When a pod looks up this service, the cluster DNS server returns the CNAME record pointing to an external domain (e.g., `mydb.external.com`). All traffic is then routed directly to that external name, bypassing `kube-proxy` entirely.
    *   **Use Case:** Provides a way to refer to external services using an internal Kubernetes service name, making application configuration consistent.

---

#### **9. Service Discovery with CoreDNS**

*   ==**CoreDNS:** The default in-cluster DNS server for Kubernetes. It runs as a `Deployment` and is exposed as a `Service` named `kube-dns` in the `kube-system` namespace.==
*   **Automatic DNS Records:** CoreDNS watches the Kubernetes API. When a new Service is created, it automatically creates DNS records for it.
    *   ==**A Record:** `[service-name].[namespace].svc.cluster.local` resolves to the Service's ClusterIP.==
*   **Pod DNS Configuration:** The `kubelet` configures each pod's `/etc/resolv.conf` file to:
    1.  Use the `kube-dns` service's ClusterIP as its primary `nameserver`.
    2.  Set up search domains (e.g., `<namespace>.svc.cluster.local`, `svc.cluster.local`). This allows pods to use short names. A pod in the `default` namespace can just resolve `hello-world-nodeport` instead of the full name, because the search domains will automatically append the suffix.