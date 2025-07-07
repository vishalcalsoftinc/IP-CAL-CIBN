
---
Of course. This is an excellent evolution of your lab setup. Migrating from a VM-based to a Kubernetes-based architecture is a standard industry practice, and your BGP-peered foundation is the perfect starting point.

Here is a similarly detailed, comprehensive guide to deploy the OAI RAN and Open5GS Core onto your two-cluster Kubernetes environment. This guide assumes you have successfully completed the BGP setup and can ping between pods in the two clusters.

---

## **Comprehensive Guide: Deploying OAI RAN and Open5GS on a Multi-Cluster Kubernetes Fabric**

### **1. High-Level Goal & Architecture Translation**

The objective is to take the functional, VM-based 5G lab (OAI + Open5GS) and deploy it as containerized microservices on your two BGP-peered Kubernetes clusters. This modernizes the deployment, making it more scalable, resilient, and manageable.

**Architecture Mapping:**

*   **From VMs to Pods:** Each component (AMF, SMF, UPF, CU-CP, DU, UE) that was previously a dedicated process on a VM will now be a set of one or more Pods within a Kubernetes `Deployment`.
*   **From Static IPs to Services:** In the VM world, components found each other using hardcoded static IPs. In Kubernetes, this is an anti-pattern. We will use Kubernetes **Services** for discovery.
*   **From Manual Process to Declarative State:** Instead of running binaries manually (`sudo ./nr-softmodem ...`), we will define the desired state in YAML manifests (`Deployments`, `ConfigMaps`, `Services`) and let Kubernetes handle the lifecycle.

**Our Kubernetes Lab Environment:**

*   **Cluster 1 (`k8s-master-1`):**
    *   **Namespace:** `oai`
    *   **Workloads:** OAI gNB (CU-CP, CU-UP, DU) and OAI NR-UE
    *   **Pod CIDR:** `10.10.0.0/16`
    *   **DNS Domain:** `open5gs.local`
*   **Cluster 2 (`k8s-master-2`):**
    *   **Namespace:** `open5gs`
    *   **Workloads:** Open5GS Core (AMF, SMF, UPF, NRF, etc.)
    *   **Pod CIDR:** `10.0.0.0/16`
    *   **DNS Domain:** `oran.local`

### **Key Architectural Concept: Cross-Cluster Service Discovery**

This is the most critical concept. Your Calico BGP peering makes every **Pod IP** routable across both clusters. However, it does **not** make `ClusterIP` Service IPs routable.

**How do we solve this?**

We will use **Headless Services** for the core Open5GS components (AMF, UPF) that need to be reached from Cluster 1.

*   A normal `ClusterIP` Service gets a single virtual IP. A request to `my-service` goes to this virtual IP, which then gets load-balanced to a backing Pod.
*   A **Headless Service** does not get a virtual IP. Instead, a DNS lookup for `my-headless-service.namespace.svc.cluster.local` resolves **directly to the list of routable Pod IPs** of the pods backing that service.

This is the magic link:
1.  The OAI gNB in Cluster 1 will be configured to connect to the AMF at its FQDN: `open5gs-amf.open5gs.svc.oran.local`.
2.  The CoreDNS in Cluster 1 will resolve this.
3.  The result of the DNS query will be the **actual Pod IP** of the AMF pod in Cluster 2 (e.g., `10.0.x.y`).
4.  Since that Pod IP is routable via BGP, the gNB can connect directly.

---

### **Part 1: Deploying Open5GS Core on Cluster 2 (`k8s-master-2`)**

We will start by deploying the "server" side of our 5G network. The easiest way to deploy a complex application like Open5GS is with a Helm chart.

#### **Section 1.1: Install Helm and Add the Open5GS Repo**

Helm is the package manager for Kubernetes.

```bash
# Execute these commands on k8s-master-2

# Install Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Add the community Open5GS chart repository
helm repo add open5gs https://gradiant.github.io/open5gs-helm-charts/
helm repo update
```

#### **Section 1.2: Create a Namespace**

It's best practice to isolate applications in their own namespaces.

```bash
# Execute on k8s-master-2
kubectl create namespace open5gs
```

#### **Section 1.3: Configure Open5GS via `values.yaml`**

This is where we translate the configuration from your original `/etc/open5gs/*.yaml` files into the Helm chart's format. Create a file named `open5gs-values.yaml`.

```yaml
# open5gs-values.yaml - Configuration for our Open5GS deployment on Cluster 2

# Global setting for the WebUI.
webui:
  enabled: true

# Configuration for the AMF (Access and Mobility Management Function)
amf:
  # CRITICAL: We create a headless service for the AMF's NGAP interface.
  # This allows the gNB in the other cluster to discover its Pod IP directly.
  service:
    # This is the standard ClusterIP service for intra-cluster communication.
    clusterIP:
      enabled: true
    # This is the new headless service for cross-cluster communication.
    headless:
      enabled: true
      # The name MUST match what the gNB will look for.
      name: open5gs-amf-ngap
  # Configuration settings mirroring your amf.yaml
  config:
    amf:
      plmn_support:
        - plmn_id:
            mcc: 999
            mnc: 70
          s_nssai:
            - sst: 1
            - sst: 2
              sd: 111111
            - sst: 3
              sd: 111111
            - sst: 4
              sd: 111111
    metrics:
      enabled: false # Disabling for simplicity

# Configuration for the SMF (Session Management Function)
smf:
  config:
    smf:
      plmn:
        - mcc: 999
          mnc: 70
      info:
        - s_nssai:
            sst: 1
          dnn:
            - internet
        # Add other slices as needed
    pfcp:
      # The SMF needs to find the UPF. It will use the UPF's service name.
      client:
        upf:
          - name: open5gs-upf-pfcp # This resolves to the UPF's service IP

# Configuration for the UPF (User Plane Function)
upf:
  # CRITICAL: We also make the UPF's GTP-U interface a headless service.
  service:
    gtpu:
      # We don't need a NodePort. The gNB will connect to the Pod IP.
      type: ClusterIP
      # We add an annotation to make it headless.
      annotations:
        "projectcalico.org/skip-advertise-service-ips": "true" # Alternative method
  config:
    upf:
      # The IP range for UEs, same as your VM setup
      subnet:
        - addr: 10.45.0.0/16
          dnn: internet
      # The name of the network interface for outbound internet traffic on the k8s node
      nat:
        ifname: enp0s3 # MATCH YOUR NODE's INTERNET ADAPTER

# Disable components we don't need for this basic setup to save resources
nrf:
  enabled: true
scp:
  enabled: false
nssf:
  enabled: true
bsf:
  enabled: false
pcf:
  enabled: true
udr:
  enabled: true
udm:
  enabled: true
ausf:
  enabled: true
```
*Note on UPF NAT*: The `ifname` must match the interface on your `k8s-master-2` node that has internet access (the NAT adapter). You can find it with `ip a`.

#### **Section 1.4: Deploy Open5GS with Helm**

Now, apply the configuration.

```bash
# Execute on k8s-master-2
helm install open5gs open5gs/open5gs --namespace open5gs -f open5gs-values.yaml
```

#### **Section 1.5: Verify the Deployment**

Check that all pods are running and the services are created.

```bash
# Execute on k8s-master-2
kubectl get pods -n open5gs -o wide
# You should see pods for amf, smf, upf, nrf, etc. in a Running state.
# Note the IP of the open5gs-amf pod. It will be in the 10.0.x.x range.

kubectl get svc -n open5gs
# You should see ClusterIP services for all components.
# Crucially, check for 'open5gs-amf-ngap' and verify it has a CLUSTER-IP of 'None'. This confirms it's headless.
```

---

### **Part 2: Deploying OAI RAN on Cluster 1 (`k8s-master-1`)**

Now we deploy the RAN components that will connect to the core we just launched.

#### **Section 2.1: Containerizing OAI Components (Conceptual)**

Before deploying to Kubernetes, each OAI binary (`nr-softmodem`, `nr-uesoftmodem`) must be in a container image. You would need a `Dockerfile` for each.

**Example `Dockerfile` for OAI gNB (`nr-softmodem`):**
```Dockerfile
# Use a base image with OAI's dependencies installed
FROM ubuntu:20.04

# Install dependencies (apt-get update, install libsctp-dev, etc.)
# ... (omitted for brevity, but this is a required step)

# Copy the compiled OAI binary into the image
COPY ./build/nr-softmodem /usr/local/bin/

# Copy the configuration file entrypoint script
COPY ./entrypoint.sh /
RUN chmod +x /entrypoint.sh

# The entrypoint script will launch the binary with the correct config
ENTRYPOINT ["/entrypoint.sh"]
```
You would build this (`docker build -t your-repo/oai-gnb:v1 .`) and push it to a registry (like Docker Hub) that your Kubernetes cluster can access. For this guide, we will assume you have these images ready: `oai/oai-gnb` and `oai/oai-nr-ue`.

#### **Section 2.2: Create Namespace and ConfigMaps**

First, create a namespace and then store the OAI configuration files in `ConfigMaps`.

```bash
# Execute on k8s-master-1
kubectl create namespace oai
```

Now, create the YAML files for your configurations.

**`oai-gnb-configmap.yaml`:**
This single `ConfigMap` will hold the configurations for CU-CP, CU-UP, and DU. We will use one `nr-softmodem` pod for all three (monolithic gNB), simplifying the lab setup.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: oai-gnb-config
  namespace: oai
data:
  gnb.conf: |
    # This is a merged configuration for a monolithic gNB (CU-CP, CU-UP, DU)
    Active_gNBs = ("oai-gnb-1");
    gNBs = ({
        gNB_ID = 0xe00;
        gNB_name = "oai-gnb-1";
        plmn_list = ({ mcc = 999; mnc = 70; mnc_length = 2; snssaiList = ({sst=1, sd=0xFFFFFF}) });
        # ... other gNB parameters ...

        # --- AMF Configuration ---
        # CRITICAL: Point to the FQDN of the AMF's headless service in Cluster 2
        amf_ip_address = ( {
            ipv4 = "open5gs-amf.open5gs.svc.oran.local";
        });

        # --- N3/UPF Interface Configuration ---
        # The local IP of the gNB for the N3 interface
        GNB_IPV4_ADDRESS_FOR_NG_AMF = "0.0.0.0"; # Let k8s assign
        GNB_IPV4_ADDRESS_FOR_NGU = "0.0.0.0"; # Let k8s assign
        # CRITICAL: The IP/port of the UPF's GTP-U interface in Cluster 2
        # We will use a NodePort service on the UPF for simplicity here
        GNB_PORT_FOR_S1U = 2152; # Standard GTP-U port

        # --- RF Simulator Configuration ---
        tr_s_preference = "f1";
        local_s_address = "127.0.0.1";
        remote_s_address = "127.0.0.1";

        # RF Simulation specific for connection to the UE
        rfsimulator = {
            serveraddr = "oai-gnb-svc.oai.svc.open5gs.local"; # gNB service for UE discovery
            serverport = 4043;
        };
    });
    # Add security, log, and other sections as per your original file
    security = {
        # ...
    };
    log_config = {
      global_log_level = "info";
    };
```
*Correction Note:* While a headless service is ideal for NGAP, the GTPU interface on the UPF is often easier to expose via `NodePort` for the gNB data plane. We will need to update the Open5GS helm values to reflect this.

**Update to `open5gs-values.yaml` and re-deploy:**
```yaml
# Add this to the upf section in open5gs-values.yaml on k8s-master-2
upf:
  service:
    gtpu:
      type: NodePort # Change from ClusterIP to NodePort
      nodePort: 32152 # Explicitly set the port
```
Apply the change: `helm upgrade open5gs open5gs/open5gs --namespace open5gs -f open5gs-values.yaml`

Now, let's create the UE config.

**`oai-ue-configmap.yaml`:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: oai-ue-config
  namespace: oai
data:
  ue.conf: |
    # Configuration for the OAI NR-UE
    # IMSI, Keys, and DNN must match a subscriber in the Open5GS DB
    imsi = "999700000000001";
    key = "465B5CE8B199B49FAA5F0A2EE238A6BC";
    opc = "E8ED289DEBA952E4283B54E88E6183CA";
    dnn = "internet";
    nssai_sst = 1;
    nssai_sd = 0xFFFFFF;

    # --- RF Simulator connection to gNB ---
    # CRITICAL: Point to the service of the gNB pod in this cluster
    rfsimulator = {
        serveraddr = "oai-gnb-svc.oai.svc.open5gs.local";
        serverport = 4043;
    };
```
Apply the ConfigMaps:
```bash
# Execute on k8s-master-1
kubectl apply -f oai-gnb-configmap.yaml
kubectl apply -f oai-ue-configmap.yaml
```

#### **Section 2.3: Deploy OAI RAN Components**

Now, create the `Deployment` manifests.

**`oai-gnb-deployment.yaml`:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: oai-gnb
  namespace: oai
spec:
  replicas: 1
  selector:
    matchLabels:
      app: oai-gnb
  template:
    metadata:
      labels:
        app: oai-gnb
    spec:
      containers:
      - name: oai-gnb
        image: oai/oai-gnb:latest # Your gNB container image
        imagePullPolicy: IfNotPresent
        securityContext:
          privileged: true # Required for network interface manipulation
        command: ["/usr/local/bin/nr-softmodem", "-O", "/etc/oai/gnb.conf", "--gNBs.[0].GNB_IPV4_ADDRESS_FOR_NGU", "k8s-master-2-node-ip", "--rfsim"] # We need to pass the node IP of k8s-master-2
        volumeMounts:
        - name: gnb-config-volume
          mountPath: /etc/oai
      volumes:
      - name: gnb-config-volume
        configMap:
          name: oai-gnb-config
---
apiVersion: v1
kind: Service
metadata:
  name: oai-gnb-svc
  namespace: oai
spec:
  selector:
    app: oai-gnb
  ports:
  - name: rfsim
    protocol: TCP
    port: 4043
    targetPort: 4043
```

**`oai-ue-deployment.yaml`:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: oai-ue
  namespace: oai
spec:
  replicas: 1
  selector:
    matchLabels:
      app: oai-ue
  template:
    metadata:
      labels:
        app: oai-ue
    spec:
      # The UE needs to start after the gNB is ready
      initContainers:
      - name: wait-for-gnb
        image: busybox:1.32
        command: ['sh', '-c', 'until nc -z -v -w30 oai-gnb-svc 4043; do echo "Waiting for gNB..."; sleep 2; done;']
      containers:
      - name: oai-ue
        image: oai/oai-nr-ue:latest # Your UE container image
        imagePullPolicy: IfNotPresent
        securityContext:
          privileged: true # Required to create the tun interface
        command: ["/usr/local/bin/nr-uesoftmodem", "-O", "/etc/oai/ue.conf", "--rfsim"]
        volumeMounts:
        - name: ue-config-volume
          mountPath: /etc/oai
      volumes:
      - name: ue-config-volume
        configMap:
          name: oai-ue-config
```

Apply the deployments:
```bash
# Execute on k8s-master-1
# IMPORTANT: Manually replace 'k8s-master-2-node-ip' in the gNB deployment with 172.17.42.15
sed -i 's/k8s-master-2-node-ip/172.17.42.15/g' oai-gnb-deployment.yaml
kubectl apply -f oai-gnb-deployment.yaml
kubectl apply -f oai-ue-deployment.yaml
```

---

### **Part 3: Verification and End-to-End Testing**

#### **Step 1: Check Pod Status**
```bash
# On k8s-master-1
kubectl get pods -n oai

# On k8s-master-2
kubectl get pods -n open5gs
```
Ensure all pods are in the `Running` state.

#### **Step 2: Check Logs for Connection**

This is the most important debugging step.

*   **On `k8s-master-2`**, check the AMF logs to see if the NGAP setup from the gNB was successful.
    ```bash
    # Get the AMF pod name
    AMF_POD=$(kubectl get pods -n open5gs -l "app.kubernetes.io/name=open5gs-amf" -o jsonpath='{.items[0].metadata.name}')
    # View the logs
    kubectl logs -f $AMF_POD -n open5gs
    ```
    You should see log lines indicating an **NG Setup Request** from the gNB's Pod IP (e.g., `10.10.x.y`) and a successful **NG Setup Response**.

*   **On `k8s-master-1`**, check the gNB logs.
    ```bash
    GNB_POD=$(kubectl get pods -n oai -l "app=oai-gnb" -o jsonpath='{.items[0].metadata.name}')
    kubectl logs -f $GNB_POD -n oai
    ```
    You should see it successfully resolving `open5gs-amf.open5gs.svc.oran.local` and establishing the SCTP connection for NGAP.

*   **On `k8s-master-1`**, check the UE logs.
    ```bash
    UE_POD=$(kubectl get pods -n oai -l "app=oai-ue" -o jsonpath='{.items[0].metadata.name}')
    kubectl logs -f $UE_POD -n oai
    ```
    You should see it go through RRC setup, registration, and finally establishing the PDU session.

#### **Step 3: The Final Test - Ping the Internet**

If all logs look good, a `tunn` interface should have been created inside the UE pod.

1.  **Get a shell inside the UE pod:**
    ```bash
    # On k8s-master-1
    kubectl exec -it $UE_POD -n oai -- /bin/bash
    ```

2.  **Inside the UE pod's shell, check for the tunnel interface:**
    ```bash
    ip a
    ```
    Look for an interface named `oaitun_ue1` with an IP address in the `10.45.0.0/16` range.

3.  **Ping Google through the tunnel:**
    ```bash
    ping -I oaitun_ue1 8.8.8.8
    ```

**If you see `PING` replies, congratulations!** You have successfully migrated the entire 5G lab to a modern, scalable, multi-cluster Kubernetes environment, leveraging BGP for the underlying fabric and Kubernetes-native service discovery to connect the components across clusters.