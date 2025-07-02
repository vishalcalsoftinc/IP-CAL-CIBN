
---
### **Topic 2 : The Foundation - Kubernetes Concepts and Commands**

Think of Kubernetes as the specialized operating system for the "data center" that is your `oai-k8s-vm` and `open5gs-vm`. Its job is to manage your applications' lifecycle, networking, and configuration. To be effective in this project, you must understand its core principles and be fluent in its command-line language. The paradigm is **declarative**: you write YAML files describing the *desired state*, and Kubernetes controllers work continuously to make the *actual state* match.

---

#### **A. Core Kubernetes Objects: The "Nouns" of Your 5G System**

These are the fundamental resources you define in YAML files to build your system.

**1. Pod: The Apartment**

*   **Concept:** The smallest deployable unit. A Pod is an abstraction over a container. It provides a shared network namespace (so containers within the same Pod can communicate via `localhost`) and shared storage volumes.
*   **Analogy:** A Pod is like a single apartment in a large building. The containers inside are the rooms. All rooms in the apartment share the same street address (IP address) and utility hookups (storage).
*   **Role in Your 5G Project:** Every single 5G network function will run in a Pod.
    *   One Pod will run the OAI DU container.
    *   Another Pod will run the OAI CU-CP container.
    *   Another Pod will run the Open5GS AMF container.
*   **Critical Nuance:** A Pod's IP address is **ephemeral**. If the Pod crashes and Kubernetes recreates it, it will almost certainly get a *new* IP address. This is why you **never** configure components to talk to a Pod IP directly. This leads us to Services.

**2. Deployment & StatefulSet: The Building Manager**

You need a higher-level controller to manage your Pods. You choose one based on whether your application is "stateless" or "stateful."

*   **Deployment (for Stateless Applications)**
    *   **Concept:** A controller that manages a set of identical, interchangeable Pods (called a ReplicaSet). It ensures a specified number of replicas are always running and handles updates gracefully (rolling updates).
    *   **Analogy:** The manager of a fast-food chain's cashier stations. If a station's screen freezes (Pod crashes), the manager just opens a new, identical station. All cashiers are interchangeable.
    *   **Role in Your 5G Project:** Perfect for most 5G core functions like the **AMF, SMF, NRF, and CU-CP**. These are typically designed to be stateless; their important data is stored in a central database (managed by the UDM). You can scale the AMF by simply increasing the replica count in its Deployment YAML.

*   **StatefulSet (for Stateful Applications)**
    *   **Concept:** A controller for applications that require a stable, unique network identity and/or stable, persistent storage.
    *   **Analogy:** The manager of an office building with named executive suites. `Suite-0` is distinct from `Suite-1`. Each suite has its own dedicated, permanent filing cabinet (persistent storage) that stays with it even if the occupant changes.
    *   **Role in Your 5G Project:**
        *   **Database:** If the Open5GS subscriber database (e.g., MongoDB) is running within Kubernetes, it **must** be a StatefulSet.
        *   **OAI gNB Components:** Depending on the OAI K8s implementation, components like the DU or UE might be in a StatefulSet to give them predictable names like `oai-du-0`, which can simplify initial configuration and scripting.

**3. Service: The Mailbox and Street Address**

This is arguably the most critical networking object to understand for your project.

*   **Concept:** A Service provides a **stable network abstraction** in front of a fluctuating set of Pods. It gets a single, stable IP address (`ClusterIP`) and a stable DNS name that resolves to that IP. It then automatically load-balances traffic sent to it across all the healthy Pods that match its label selector.
*   **Analogy:** Your home's mailbox and street address. The mail carrier (another K8s component) delivers mail to your address. It doesn't need to know who is inside the house (which Pod is active) or if the residents just changed. The address remains the same.
*   **Role in Your 5G Project:** **This is the glue that holds your architecture together.**
    *   The OAI CU-CP's configuration file (`cucp.conf`) will *not* contain the IP `10.42.1.55` (a Pod IP).
    *   Instead, it will contain the DNS name of the AMF's Service, for example: `open5gs-amf.open5gs.svc.cluster.local`.
    *   Kubernetes' internal DNS will resolve this name to the stable `ClusterIP` of the `open5gs-amf` Service.
    *   The Service will then forward the N2 traffic to one of the healthy `amf` Pods.
    *   **Every arrow on your diagram that points to a "service IP: Port" box is pointing to a Kubernetes Service.**

**4. ConfigMap & Secret: The Instruction Manuals and Keys**

*   **Concept:** Objects that decouple configuration data from your container images. This allows you to change configuration without rebuilding your OAI/Open5GS images.
*   **Analogy:** A ConfigMap is the instruction manual for assembling furniture, stored outside the box. A Secret is the key to the apartment, also kept separate. You can change the instructions or the key without having to repackage the furniture.
*   **Role in Your 5G Project:**
    *   **ConfigMap:** You will create a ConfigMap for each 5G component. For example, a `oai-cucp-config` ConfigMap will contain the full text of your `cucp.conf` file. This ConfigMap is then mounted into the `oai-cucp` Pod as a file (e.g., at `/etc/oai/cucp.conf`).
    *   **Secret:** If you have TLS certificates for securing communication between NFs or database passwords, they should be stored in Secrets, which are mounted similarly.

**5. Namespace: The Office Floors**

*   **Concept:** A way to create logical partitions within your Kubernetes cluster. Resources in one namespace are isolated from another.
*   **Analogy:** The different floors in an office building. The Sales department on the 1st floor (`oai` namespace) is separate from the Engineering department on the 2nd floor (`open5gs` namespace). They can't see each other's printers (Services) unless explicitly allowed.
*   **Role in Your 5G Project:** Your setup almost certainly uses namespaces for organization and security.
    *   `kubectl get pods -n oai` will show only the RAN components.
    *   `kubectl get pods -n open5gs` will show only the Core components.
    *   This is critical because it means a Service name like `amf-service` is actually `amf-service.open5gs`. From the `oai` namespace, you must use the full name: `amf-service.open5gs.svc.cluster.local`.

---

#### **B. Essential `kubectl` Commands: The "Verbs" to Manage Your System**

These are your primary tools for interaction. Become fluent with them.

**1. Inspection (Read-Only Commands):**

*   `kubectl get <resource-type> -n <namespace>`: Your go-to command.
    *   `kubectl get pods -n oai -o wide`: See OAI pods, their status, IPs, and which physical node they are on.
    *   `kubectl get svc -n open5gs`: See the stable ClusterIPs for all Open5GS services.
    *   `kubectl get configmap oai-cucp-config -n oai -o yaml`: Dump the YAML definition of the CU-CP's configmap to see the exact configuration being used.

*   `kubectl describe <resource-type> <resource-name> -n <namespace>`: Your main diagnostic tool.
    *   `kubectl describe pod oai-du-xyz-123 -n oai`: Provides a wealth of information. **Always check the `Events` section at the bottom first!** It will tell you why a Pod is stuck in `Pending` (e.g., "FailedScheduling: insufficient CPU") or why it's crashing (e.g., "Failed to pull image").

**2. Debugging (Interactive Commands):**

*   `kubectl logs <pod-name> -n <namespace>`: The voice of your application.
    *   `kubectl logs open5gs-amf-abc-456 -n open5gs`: Shows the stdout/stderr from the AMF process. Look for errors like "N2 connection failed" or "Cannot find SMF".
    *   **Pro-Tips:**
        *   `kubectl logs -f <pod-name> -n <ns>`: Follow the logs in real-time.
        *   `kubectl logs --previous <pod-name> -n <ns>`: If a pod is in `CrashLoopBackOff`, this shows you the logs from its *previous* failed run, which is essential for finding the cause of the crash.

*   `kubectl exec -it <pod-name> -n <namespace> -- <command>`: Your "teleporter" into a running container.
    *   `kubectl exec -it oai-cucp-pod-123 -n oai -- bash`: Gives you a root shell inside the CU-CP container.
    *   **Sample Debugging Workflow from inside the CU-CP pod:**
        1.  Check if the config file was mounted correctly: `ls -l /etc/oai/` and `cat /etc/oai/cucp.conf`.
        2.  Check its own IP: `ip addr`.
        3.  Test DNS resolution for the AMF service: `nslookup open5gs-amf-service.open5gs.svc.cluster.local`.
        4.  Test basic connectivity to the AMF service: `ping <AMF-Service-ClusterIP>`.
        5.  Test SCTP port connectivity: `netstat -an | grep <AMF-Port>` or use a tool like `nmap` if available.

**3. Application Management:**

*   `kubectl apply -f <filename.yaml>`: The command to create or update resources from a YAML file.
*   `kubectl delete -f <filename.yaml>`: The command to delete resources defined in a file.
*   `kubectl rollout restart deployment <deployment-name> -n <namespace>`: A safe way to restart an application. It triggers a rolling update, creating new Pods and terminating old ones. This is the standard way to force an application to re-read its updated ConfigMap.