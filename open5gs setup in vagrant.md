
---
Of course. You have laid out the perfect foundation. Now, let's deploy the Open5GS 5G Core onto the dedicated Kubernetes cluster.

This guide provides all the necessary Kubernetes manifests and configurations to get Open5GS running on your `open5gs-k8s-vm`. We will configure it to be accessible from your OAI RAN cluster.

### **Deployment Strategy**

1.  **Database:** Open5GS requires a MongoDB database to store subscriber information. We will deploy this as a standard `Deployment` and `Service` within the cluster.
2.  **Configuration:** All Open5GS configuration files (`amf.yaml`, `smf.yaml`, etc.) will be stored in a Kubernetes `ConfigMap`. This decouples the configuration from the container images.
3.  **Core Functions (NFs):** Each core network function (NRF, AMF, SMF, UPF) will be deployed as a separate `Deployment`.
4.  **External Access:** We will use `NodePort` services to expose the AMF's N2 interface (SCTP) and the UPF's N3 interface (GTP-U). This makes them reachable using the node's IP (`172.17.42.27`) from the OAI cluster.
5.  **Web UI:** We will also deploy the Open5GS WebUI using a `NodePort` service, allowing you to easily add and manage subscribers from your host machine's browser.

---

### **Part 3: Deploying Open5GS on the Core Cluster**

The following commands should all be executed on the **`open5gs-k8s-vm`** after you have connected to it with `vagrant ssh open5gs-k8s-vm`.

#### **Step 3.1: Create a Namespace**

It's best practice to isolate our 5G Core components in their own namespace.

```bash
kubectl create namespace open5gs
```

#### **Step 3.2: Deploy MongoDB**

First, we need the database. Create a file named `mongo-deployment.yaml`.

```bash
nano mongo-deployment.yaml
```

Paste the following content:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: open5gs-mongodb-svc
  namespace: open5gs
  labels:
    app: open5gs-mongodb
spec:
  ports:
  - port: 27017
    protocol: TCP
  selector:
    app: open5gs-mongodb
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: open5gs-mongodb
  namespace: open5gs
  labels:
    app: open5gs-mongodb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: open5gs-mongodb
  template:
    metadata:
      labels:
        app: open5gs-mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo:5.0
        ports:
        - containerPort: 27017
```

Apply the manifest to create the MongoDB deployment and service:

```bash
kubectl apply -f mongo-deployment.yaml
```

#### **Step 3.3: Create the Open5GS ConfigMap**

This is the most important configuration step. We will translate the required configurations from your original VM-based setup into a single `ConfigMap`.

Create a file named `open5gs-configmap.yaml`:

```bash
nano open5gs-configmap.yaml
```

Paste the entire configuration below. Note the following key changes from a VM setup:
*   Service discovery (e.g., finding the NRF or UPF) is done using Kubernetes service DNS names (e.g., `open5gs-nrf-svc`).
*   The `dev:` parameter in SBI/interface sections is set to `eth0`, which is the default network interface inside a Kubernetes pod.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: open5gs-config
  namespace: open5gs
data:
  # AMF Configuration
  amf.yaml: |
    logger:
        file: /var/log/open5gs/amf.log
    amf:
        sbi:
          - dev: eth0
            port: 7777
        ngap:
          - dev: eth0 # Listens on the pod's interface for N2
        guami:
          - plmn_id:
              mcc: 999
              mnc: 70
            amf_id:
              region: 2
              set: 1
        tai:
          - plmn_id:
              mcc: 999
              mnc: 70
            tac: 1
        plmn_support:
          - plmn_id:
              mcc: 999
              mnc: 70
            s_nssai:
              - sst: 1
              - sst: 1
                sd: 111111
        security:
            integrity_order: [ NIA2, NIA1, NIA0 ]
            ciphering_order: [ NEA0, NEA1, NEA2 ]
        network_name:
            full: Open5GS
    nrf:
       sbi:
         - hostname: open5gs-nrf-svc # K8s Service Name
           port: 7777
    scp:
       sbi:
         - hostname: open5gs-scp-svc # K8s Service Name
           port: 7777

  # SMF Configuration
  smf.yaml: |
    logger:
        file: /var/log/open5gs/smf.log
    smf:
        sbi:
          - dev: eth0
            port: 7777
        pfcp:
          - dev: eth0
        gtpc:
          - dev: eth0
        gtpu:
          - dev: eth0
        pdn:
          - type: IPv4
            addr: 10.45.0.0/16 # This is the UE IP Pool
        dns:
          - 8.8.8.8
          - 8.8.4.4
    upf:
        pfcp:
          - hostname: open5gs-upf-svc # K8s Service Name
    nrf:
       sbi:
         - hostname: open5gs-nrf-svc # K8s Service Name
           port: 7777
    scp:
       sbi:
         - hostname: open5gs-scp-svc # K8s Service Name
           port: 7777

  # UPF Configuration
  upf.yaml: |
    logger:
        file: /var/log/open5gs/upf.log
    upf:
      pfcp:
        - dev: eth0
      gtpu:
        - dev: eth0
      subnet:
        - dev: ogstun # Interface for N6 to Internet
          addr: 10.45.0.0/16
          dnn: internet

  # NRF Configuration
  nrf.yaml: |
    logger:
        file: /var/log/open5gs/nrf.log
    nrf:
        sbi:
          - dev: eth0
            port: 7777
    scp:
       sbi:
         - hostname: open5gs-scp-svc # K8s Service Name
           port: 7777
           
  # Other NFs (SCP, UDM, AUSF, PCF)
  # These use internal K8s service discovery
  scp.yaml: |
    logger:
      file: /var/log/open5gs/scp.log
    scp:
      sbi:
        - dev: eth0
          port: 7777
    nrf:
      sbi:
        - hostname: open5gs-nrf-svc
          port: 7777
  pcf.yaml: |
    logger:
      file: /var/log/open5gs/pcf.log
    pcf:
      sbi:
        - dev: eth0
          port: 7777
    nrf:
      sbi:
        - hostname: open5gs-nrf-svc
          port: 7777
    scp:
      sbi:
        - hostname: open5gs-scp-svc
          port: 7777
  udm.yaml: |
    logger:
      file: /var/log/open5gs/udm.log
    udm:
      sbi:
        - dev: eth0
          port: 7777
    nrf:
      sbi:
        - hostname: open5gs-nrf-svc
          port: 7777
    scp:
      sbi:
        - hostname: open5gs-scp-svc
          port: 7777
  ausf.yaml: |
    logger:
      file: /var/log/open5gs/ausf.log
    ausf:
      sbi:
        - dev: eth0
          port: 7777
    nrf:
      sbi:
        - hostname: open5gs-nrf-svc
          port: 7777
    scp:
      sbi:
        - hostname: open5gs-scp-svc
          port: 7777
  udr.yaml: |
    logger:
        file: /var/log/open5gs/udr.log
    udr:
        sbi:
          - dev: eth0
            port: 7777
    db_uri: mongodb://open5gs-mongodb-svc/open5gs
    nrf:
       sbi:
         - hostname: open5gs-nrf-svc
           port: 7777
    scp:
       sbi:
         - hostname: open5gs-scp-svc
           port: 7777
```

Apply the ConfigMap:
```bash
kubectl apply -f open5gs-configmap.yaml
```

#### **Step 3.4: Deploy the Open5GS Core Network Functions**

Now we create the deployments for all the core components and the crucial `NodePort` services for AMF and UPF.

Create a file named `open5gs-core.yaml`:

```bash
nano open5gs-core.yaml
```

Paste the following manifest. It contains all NFs.

```yaml
# This file deploys all Open5GS Core Network Functions
# It references the 'open5gs-config' ConfigMap created earlier.
---
# NRF (Network Repository Function)
apiVersion: v1
kind: Service
metadata:
  name: open5gs-nrf-svc
  namespace: open5gs
spec:
  selector:
    app: open5gs-nrf
  ports:
    - protocol: TCP
      port: 7777
      targetPort: 7777
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: open5gs-nrf
  namespace: open5gs
spec:
  replicas: 1
  selector:
    matchLabels:
      app: open5gs-nrf
  template:
    metadata:
      labels:
        app: open5gs-nrf
    spec:
      containers:
      - name: nrf
        image: open5gs/open5gs-nrf:v2.6.6
        volumeMounts:
        - name: open5gs-config-volume
          mountPath: /etc/open5gs
      volumes:
      - name: open5gs-config-volume
        configMap:
          name: open5gs-config
---
# UDM (Unified Data Management)
apiVersion: v1
kind: Service
metadata:
  name: open5gs-udm-svc
  namespace: open5gs
spec:
  selector:
    app: open5gs-udm
  ports:
  - protocol: TCP
    port: 7777
    targetPort: 7777
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: open5gs-udm
  namespace: open5gs
spec:
  replicas: 1
  selector:
    matchLabels:
      app: open5gs-udm
  template:
    metadata:
      labels:
        app: open5gs-udm
    spec:
      containers:
      - name: udm
        image: open5gs/open5gs-udm:v2.6.6
        volumeMounts:
        - name: open5gs-config-volume
          mountPath: /etc/open5gs
      volumes:
      - name: open5gs-config-volume
        configMap:
          name: open5gs-config
---
# AUSF (Authentication Server Function)
apiVersion: v1
kind: Service
metadata:
  name: open5gs-ausf-svc
  namespace: open5gs
spec:
  selector:
    app: open5gs-ausf
  ports:
  - protocol: TCP
    port: 7777
    targetPort: 7777
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: open5gs-ausf
  namespace: open5gs
spec:
  replicas: 1
  selector:
    matchLabels:
      app: open5gs-ausf
  template:
    metadata:
      labels:
        app: open5gs-ausf
    spec:
      containers:
      - name: ausf
        image: open5gs/open5gs-ausf:v2.6.6
        volumeMounts:
        - name: open5gs-config-volume
          mountPath: /etc/open5gs
      volumes:
      - name: open5gs-config-volume
        configMap:
          name: open5gs-config
---
# UDR (Unified Data Repository)
apiVersion: v1
kind: Service
metadata:
  name: open5gs-udr-svc
  namespace: open5gs
spec:
  selector:
    app: open5gs-udr
  ports:
  - protocol: TCP
    port: 7777
    targetPort: 7777
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: open5gs-udr
  namespace: open5gs
spec:
  replicas: 1
  selector:
    matchLabels:
      app: open5gs-udr
  template:
    metadata:
      labels:
        app: open5gs-udr
    spec:
      containers:
      - name: udr
        image: open5gs/open5gs-udr:v2.6.6
        volumeMounts:
        - name: open5gs-config-volume
          mountPath: /etc/open5gs
      volumes:
      - name: open5gs-config-volume
        configMap:
          name: open5gs-config
---
# PCF (Policy Control Function)
apiVersion: v1
kind: Service
metadata:
  name: open5gs-pcf-svc
  namespace: open5gs
spec:
  selector:
    app: open5gs-pcf
  ports:
  - protocol: TCP
    port: 7777
    targetPort: 7777
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: open5gs-pcf
  namespace: open5gs
spec:
  replicas: 1
  selector:
    matchLabels:
      app: open5gs-pcf
  template:
    metadata:
      labels:
        app: open5gs-pcf
    spec:
      containers:
      - name: pcf
        image: open5gs/open5gs-pcf:v2.6.6
        volumeMounts:
        - name: open5gs-config-volume
          mountPath: /etc/open5gs
      volumes:
      - name: open5gs-config-volume
        configMap:
          name: open5gs-config
---
# SCP (Service Communication Proxy)
apiVersion: v1
kind: Service
metadata:
  name: open5gs-scp-svc
  namespace: open5gs
spec:
  selector:
    app: open5gs-scp
  ports:
  - protocol: TCP
    port: 7777
    targetPort: 7777
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: open5gs-scp
  namespace: open5gs
spec:
  replicas: 1
  selector:
    matchLabels:
      app: open5gs-scp
  template:
    metadata:
      labels:
        app: open5gs-scp
    spec:
      containers:
      - name: scp
        image: open5gs/open5gs-scp:v2.6.6
        volumeMounts:
        - name: open5gs-config-volume
          mountPath: /etc/open5gs
      volumes:
      - name: open5gs-config-volume
        configMap:
          name: open5gs-config
---
# SMF (Session Management Function)
apiVersion: v1
kind: Service
metadata:
  name: open5gs-smf-svc
  namespace: open5gs
spec:
  selector:
    app: open5gs-smf
  ports:
  - protocol: TCP
    port: 7777
    targetPort: 7777
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: open5gs-smf
  namespace: open5gs
spec:
  replicas: 1
  selector:
    matchLabels:
      app: open5gs-smf
  template:
    metadata:
      labels:
        app: open5gs-smf
    spec:
      containers:
      - name: smf
        image: open5gs/open5gs-smf:v2.6.6
        volumeMounts:
        - name: open5gs-config-volume
          mountPath: /etc/open5gs
      volumes:
      - name: open5gs-config-volume
        configMap:
          name: open5gs-config
---
# AMF (Access and Mobility Management Function) - EXPOSED VIA NODEPORT
apiVersion: v1
kind: Service
metadata:
  name: open5gs-amf-svc
  namespace: open5gs
spec:
  type: NodePort # This exposes the service on the node's IP
  selector:
    app: open5gs-amf
  ports:
    - name: sctp-n2
      protocol: SCTP
      port: 38412
      targetPort: 38412
      nodePort: 30412 # Static port for N2, accessible from outside
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: open5gs-amf
  namespace: open5gs
spec:
  replicas: 1
  selector:
    matchLabels:
      app: open5gs-amf
  template:
    metadata:
      labels:
        app: open5gs-amf
    spec:
      containers:
      - name: amf
        image: open5gs/open5gs-amf:v2.6.6
        volumeMounts:
        - name: open5gs-config-volume
          mountPath: /etc/open5gs
      volumes:
      - name: open5gs-config-volume
        configMap:
          name: open5gs-config
---
# UPF (User Plane Function) - EXPOSED VIA NODEPORT
apiVersion: v1
kind: Service
metadata:
  name: open5gs-upf-svc
  namespace: open5gs
spec:
  type: NodePort # This exposes the service on the node's IP
  selector:
    app: open5gs-upf
  ports:
    - name: udp-n3
      protocol: UDP
      port: 2152
      targetPort: 2152
      nodePort: 30152 # Static port for N3 (GTP-U)
    - name: udp-n4
      protocol: UDP
      port: 8805
      targetPort: 8805
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: open5gs-upf
  namespace: open5gs
spec:
  replicas: 1
  selector:
    matchLabels:
      app: open5gs-upf
  template:
    metadata:
      labels:
        app: open5gs-upf
    spec:
      containers:
      - name: upf
        image: open5gs/open5gs-upf:v2.6.6
        securityContext:
          privileged: false
          capabilities:
            add:
              - NET_ADMIN # Required to create the 'ogstun' interface
        volumeMounts:
        - name: open5gs-config-volume
          mountPath: /etc/open5gs
      volumes:
      - name: open5gs-config-volume
        configMap:
          name: open5gs-config
```

Apply the manifest to deploy all the core functions:
```bash
kubectl apply -f open5gs-core.yaml
```

#### **Step 3.5: Enable Internet Access for UEs (N6 Interface)**

The UPF pod will create a tunnel `ogstun` to route UE traffic. For this traffic to reach the internet, we must add a NAT rule on the `open5gs-k8s-vm` node itself.

Find your node's internet-facing interface (it's `enp0s3` for the default Vagrant NAT adapter).
```bash
ip a
```
Then, add the `iptables` rule to masquerade traffic coming from the UE IP pool (`10.45.0.0/16`).
```bash
sudo iptables -t nat -A POSTROUTING -s 10.45.0.0/16 -o enp0s3 -j MASQUERADE
```

#### **Step 3.6: Deploy and Access the Web UI**

This will allow you to add subscribers easily. Create a file named `open5gs-webui.yaml`.

```bash
nano open5gs-webui.yaml
```

Paste the following:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: open5gs-webui-svc
  namespace: open5gs
spec:
  type: NodePort
  selector:
    app: open5gs-webui
  ports:
  - port: 80
    targetPort: 3000
    nodePort: 30080 # We will access the UI on this port
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: open5gs-webui
  namespace: open5gs
spec:
  replicas: 1
  selector:
    matchLabels:
      app: open5gs-webui
  template:
    metadata:
      labels:
        app: open5gs-webui
    spec:
      containers:
      - name: webui
        image: open5gs/open5gs-webui:v2.6.6
        env:
        - name: DB_URI
          value: "mongodb://open5gs-mongodb-svc/open5gs"
        ports:
        - containerPort: 3000
```
Apply it:
```bash
kubectl apply -f open5gs-webui.yaml
```

#### **Step 3.7: Verification**

1.  **Check Pods:** Ensure all pods are in the `Running` state.
    ```bash
    kubectl get pods -n open5gs -o wide
    ```
2.  **Check Services:** Verify the `NodePort` services are up and note the assigned ports.
    ```bash
    kubectl get svc -n open5gs
    ```
    You should see `30412` for the AMF and `30152` for the UPF.
3.  **Access WebUI and Add Subscriber:**
    *   Open a web browser on your **host machine** (not the VM).
    *   Navigate to `http://172.17.42.27:30080`.
    *   Log in with the default credentials (`admin`/`admin`).
    *   Click "Add Subscriber".
    *   Enter the IMSI `999700000000001` and the K/OPC keys from your OAI configuration.

Your Open5GS core is now fully deployed on Kubernetes and ready to accept connections from the OAI RAN cluster.