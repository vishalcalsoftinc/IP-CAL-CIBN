
---
### **Kubernetes Networking Series: Part 5 - Network Policies (Network Security)**

#### **1. Introduction: From Traditional Security to Kubernetes Security**

This section contrasts the old way of securing applications with the modern Kubernetes approach.

*   **Traditional "Farm" Model (Pre-Kubernetes):**
    *   **Architecture:** Applications were deployed in rigid, physically or logically separated "farms" (e.g., a web farm, an application farm, a database farm).
    *   **Security Model:** Security was based on **physical network topology**. Firewalls were placed between these farms, and rules were based on static IP addresses and port numbers.
    *   **Process:** Managed by separate teams (Network Team, Infosec Team). Making a change was slow, required tickets, and was not agile.
    *   **Limitations:** This model is too static and slow for the dynamic, frequent deployments required by modern microservices.

*   **The Kubernetes Model:**
    *   **Architecture:** A flat network where, by default, any pod can communicate with any other pod, regardless of which node it's on.
    *   **Security Model:** Security is **declarative** and **decoupled from the network topology**. It is defined as code (YAML) and applied to pods based on metadata (labels).
    *   **Process:** Empowered developers and DevOps engineers can define security policies as part of their application's deployment lifecycle (GitOps, CI/CD).
    *   **Key Idea:** Instead of physical firewalls, Kubernetes uses **`iptables`** rules on each node to intercept and filter traffic, enforcing the declared policies.

---

#### **2. Why Network Policies are Important**

*   ==**Sophisticated Attacks:**== As attacks become more frequent and sophisticated, developers must take responsibility for securing their applications.
*   ==**Perimeter Security is Not Enough:**== Traditional firewalls protect the perimeter of the cluster (**North-South traffic** - traffic entering/leaving the cluster). However, they are ineffective at protecting against attacks that have already breached the perimeter and are moving laterally within the cluster (**East-West traffic** - pod-to-pod communication).
*   ==**Designed for Dynamic Environments:**== Pods are ephemeral, and their IPs change constantly. Network Policies use **label selectors** to group pods, which is a stable and reliable method, unlike relying on volatile IP addresses.
*   **Empowers Developers (DevSecOps):** Developers can define security intent (e.g., "only the `business` tier can talk to the `database` tier") directly in their application's YAML, integrating security into the development workflow. This is much faster and more agile than raising tickets with a separate security team.

---

#### **3. Core Features of Kubernetes Network Policies**

*   ==**Namespace-Scoped:**== A NetworkPolicy you create only applies to pods within the **same namespace**.
*   ==**Applied via Label Selectors:**== The `podSelector` field in a policy determines which pods the policy applies to.
*   **Rule Types:**
    *   ==**Ingress:** Rules that control *incoming* traffic to the selected pods.==
    *   ==**Egress:** Rules that control *outgoing* traffic from the selected pods.==
*   **Traffic Sources/Destinations:** Rules can specify allowed traffic based on:
    *   **`podSelector`:** From/to other pods that have specific labels.
    *   **`namespaceSelector`:** From/to any pod in a namespace that has a specific label.
    *   **`ipBlock`:** From/to a specific CIDR range (useful for allowing traffic from outside the cluster or restricting access to the internet).
*   **Protocols and Ports:** Rules can be further refined by protocol (TCP, UDP) and port number or named port.
*   **Enforcement by CNI Plugin:** ==**Kubernetes itself does not enforce Network Policies.** It's the responsibility of the installed CNI plugin (like Calico or Cilium) to read the policy objects from the API server and translate them into actual filtering rules (`iptables`, eBPF) on the nodes. Flannel, for instance, does not support Network Policies.==

---

#### **4. Demo 1: Default-Deny Ingress and Whitelisting**

This demo shows how to lock down a namespace and then selectively allow traffic.

*   ==**The "Zero Trust" Starting Point:** By default, Kubernetes allows all traffic. The first step in securing a namespace is to apply a **default-deny** policy.==
    *   **How to create a Default-Deny Policy:**
        ```yaml
        apiVersion: networking.k8s.io/v1
        kind: NetworkPolicy
        metadata:
          name: default-deny-all-ingress
          namespace: prod
        spec:
          podSelector: {} # An empty selector matches ALL pods in the namespace
          ingress: []      # An empty ingress rule list means NO ingress traffic is allowed
        ```
    *   **Effect:** After applying this, **no pods** from anywhere can send traffic to any pod in the `prod` namespace.

*   **Step-by-Step Whitelisting:** After locking everything down, you add policies to allow specific traffic flows.
    1.  **Allow UI to receive traffic from Business:**
        *   **`podSelector`:** `app: products-business` (Apply this policy to the business pods).
        *   **`ingress.from.podSelector`:** `app: products-ui` (Allow traffic *from* pods with the UI label).
    2.  **Allow Business to receive traffic from Database:**
        *   **`podSelector`:** `app: products-db` (Apply this policy to the database pods).
        *   **`ingress.from.podSelector`:** `app: products-business` (Allow traffic *from* pods with the business label).

*   **Testing:** After applying these policies, tests show that `UI -> Business` and `Business -> DB` calls succeed, but `UI -> DB` calls fail (timeout), proving the policy is working.

---

#### **5. Demo 2: Cross-Namespace Policies**

This demo shows how to allow a pod in one namespace (`stage`) to talk to a pod in another (`prod`).

*   **The Challenge:** A standard `podSelector` only works for pods within the *same* namespace as the policy.
*   **The Solution: `namespaceSelector`**
    1.  **Label the Source Namespace:** First, you must apply a label to the source namespace (`stage`).
        ```bash
        kubectl label namespace stage "products-db-access=true"
        ```
    2.  **Modify the Policy:** In the policy applied to the `prod-db` pod, add a `namespaceSelector` to the `ingress.from` rule.
        ```yaml
        # Policy applied to the DB pod in the 'prod' namespace
        spec:
          podSelector:
            matchLabels:
              app: products-db
          ingress:
          - from:
            # Allow traffic from pods with this label...
            - podSelector:
                matchLabels:
                  app: products-business
              # ...IF they are in a namespace with this label.
              namespaceSelector:
                matchLabels:
                  products-db-access: "true"
        ```
*   **Result:** Now, the `business` pod in the `stage` namespace can successfully connect to the `db` pod in the `prod` namespace.

---

#### **6. Demo 3: Restricting Egress (Outgoing) Traffic**

This demo shows how to prevent a sensitive pod (like a database) from making unauthorized outbound connections.

1.  **Apply a Default-Deny Egress Policy:** Similar to ingress, an empty `egress` block denies all outgoing traffic from the selected pods.
    ```yaml
    spec:
      podSelector:
        matchLabels:
          app: products-db
      egress: [] # Deny all outgoing traffic from the DB pod
    ```
    *   **Effect:** After this is applied, the DB pod cannot connect to the internet, other pods, or even resolve DNS names.

2.  **Allow Specific Egress Traffic:**
    *   **Allow DNS:** To restore DNS, you must explicitly allow egress traffic to the `kube-dns` service. This is done by selecting the `kube-system` namespace and targeting UDP port 53.
        ```yaml
        egress:
        - to:
          - namespaceSelector: # Find the kube-system namespace...
              matchLabels:
                name: kube-system # (Assuming you've labeled the namespace)
          ports:
          - protocol: UDP
            port: 53
        ```
    *   **Allow Traffic to Other Pods:** To allow the DB pod to talk to other pods within the cluster, add a rule that allows traffic to the cluster's pod CIDR range.
        ```yaml
        egress:
        - to:
          - ipBlock:
              cidr: 10.244.0.0/16 # The pod network CIDR
        ```

*   **Result:** The database pod can now resolve DNS and talk to other pods in the cluster, but it is still blocked from making connections to the public internet, successfully securing it.