
---
Of course. This is an excellent migration path. Moving from a manual VM setup to a declarative Kubernetes environment is a significant upgrade in terms of reproducibility, scalability, and management.

Here is the detailed, step-by-step guide to deploy the OAI RAN components (CU-CP, CU-UP, DU, NR-UE) onto your `oai-k8s-vm` cluster. We will translate the configurations from your VM-based guide into Kubernetes `ConfigMap` and `Deployment` manifests.

---

### **Prerequisites**

1.  You have successfully completed the "Calico BGP setup using Vagrant" guide.
2.  You are logged into the OAI RAN machine: `vagrant ssh oai-k8s-vm`.
3.  Your `kubectl` context is correctly pointing to the local cluster. Verify with `kubectl get nodes`.

---

### **Part 1: Deploying OAI RAN on Kubernetes**

We will perform all the following steps inside the `oai-k8s-vm`.

#### **Step 1: Create a Namespace**

It's a best practice to isolate our applications. Let's create a namespace for all OAI components.

```bash
kubectl create namespace oai
```

#### **Step 2: Create OAI Configuration Files (`ConfigMaps`)**

Instead of having `.conf` files on the VM's filesystem, we will store them as `ConfigMap` objects in Kubernetes. This allows us to mount them into our pods as files.

Create a file named `oai-ran-configs.yaml` and paste the entire content below. We will create all `ConfigMap`s and `Service`s in one go.

**A Note on Networking in Kubernetes:**
*   **Pod-to-Pod:** Instead of using dynamic Pod IPs, we create Kubernetes `Services`. This gives us stable DNS names (e.g., `oai-du.oai.svc.cluster.local`) that other pods can use to communicate.
*   **Pod to External:** To reach the Open5GS cluster, we will use the host IP of the core VM (`172.17.42.27`) and the `NodePort`s we will define later.
*   **Pod's Own IP:** We will use the Kubernetes Downward API to inject a pod's own IP address as an environment variable, which we can then use in the container's startup command.

```yaml
# oai-ran-configs.yaml

# --- Service Definitions for Intra-Cluster Communication ---

apiVersion: v1
kind: Service
metadata:
  name: oai-cucp
  namespace: oai
spec:
  selector:
    app: oai-cucp
  ports:
    - name: f1c
      protocol: SCTP
      port: 38472
      targetPort: 38472
    - name: e1
      protocol: SCTP
      port: 38462
      targetPort: 38462
---
apiVersion: v1
kind: Service
metadata:
  name: oai-cuup
  namespace: oai
spec:
  selector:
    app: oai-cuup
  ports:
    - name: f1u
      protocol: UDP
      port: 2152
      targetPort: 2152
    - name: e1
      protocol: SCTP
      port: 38462
      targetPort: 38462
---
apiVersion: v1
kind: Service
metadata:
  name: oai-du
  namespace: oai
spec:
  selector:
    app: oai-du
  ports:
    - name: f1c
      protocol: SCTP
      port: 38472
      targetPort: 38472
    - name: f1u
      protocol: UDP
      port: 2152
      targetPort: 2152
    - name: rfsim
      protocol: TCP
      port: 4043
      targetPort: 4043
---

# --- ConfigMap for OAI CU-CP ---

apiVersion: v1
kind: ConfigMap
metadata:
  name: oai-cucp-config
  namespace: oai
data:
  oai-cucp.conf: |
    Active_gNBs = ("oai-cu-cp");
    Asn1_verbosity = "info";
    gNBs = (
      {
        gNB_ID = 0xe01;
        gNB_name = "oai-cu-cp";
        cell_type = "CELL_MACRO_GNB";
        tracking_area_code = 1;
        plmn_list = ( { mcc = 999; mnc = 70; mnc_length = 2; snssaiList = ({sst=1, sd=0xFFFFFF},{sst=2, sd=0x111111})} );
        tr_s_preference = "f1";
        local_s_address = "127.0.0.1"; # Overridden by command line
        remote_s_address = "oai-du.oai"; # K8s Service Name
        local_s_portc = 501;
        remote_s_portc = 500;
        local_s_portd = 2154;
        remote_s_portd = 2153;

        amf_ip_address = ( { ipv4 = "172.17.42.27"; } ); # Open5GS VM IP (for NodePort)

        E1_INTERFACE = (
          {
            type = "cp";
            ipv4_cucp = "127.0.0.1"; # Overridden by command line
            port_cucp = 38462;
            ipv4_cuup = "oai-cuup.oai"; # K8s Service Name
            port_cuup = 38462;
          }
        );
        NETWORK_INTERFACES = {
          GNB_IPV4_ADDRESS_FOR_NG_AMF = "127.0.0.1"; # Overridden by command line
        };
      }
    );
    SCTP: { SCTP_INSTREAMS = 5; SCTP_OUTSTREAMS = 5; };
    security = { ciphering_algorithms = ("nea0"); integrity_algorithms = ("nia2", "nia0"); };
    log_config: { global_log_level = "info"; };
---

# --- ConfigMap for OAI CU-UP ---

apiVersion: v1
kind: ConfigMap
metadata:
  name: oai-cuup-config
  namespace: oai
data:
  oai-cuup.conf: |
    Active_gNBs = ("oai-cu-up");
    Asn1_verbosity = "info";
    gNBs = (
      {
        gNB_ID = 0xe01;
        gNB_CU_UP_ID = 0xe00;
        gNB_name = "oai-cuup-sst1";
        cell_type = "CELL_MACRO_GNB";
        tracking_area_code = 1;
        plmn_list = ( { mcc = 999; mnc = 70; mnc_length = 2; snssaiList = ({sst=1, sd=0xFFFFFF},{sst=2, sd=0x111111})} );
        tr_s_preference = "f1";
        local_s_address = "127.0.0.1"; # Overridden
        remote_s_address = "oai-du.oai";
        local_s_portc = 501;
        remote_s_portc = 500;
        local_s_portd = 2154;
        remote_s_portd = 2153;

        E1_INTERFACE = (
          {
            type = "up";
            ipv4_cucp = "oai-cucp.oai";
            ipv4_cuup = "127.0.0.1"; # Overridden
          }
        );

        NETWORK_INTERFACES: {
          GNB_IPV4_ADDRESS_FOR_NGU = "127.0.0.1"; # Overridden
          GNB_PORT_FOR_S1U = 2152;
        };
      }
    );
    log_config: { global_log_level = "info"; };
---

# --- ConfigMap for OAI DU ---

apiVersion: v1
kind: ConfigMap
metadata:
  name: oai-du-config
  namespace: oai
data:
  oai-du.conf: |
    Active_gNBs = ("oai-du");
    gNBs = ({
        gNB_ID = 0xe01;
        gNB_DU_ID = 0xe01;
        gNB_name = "oai-du";
        tracking_area_code = 1;
        plmn_list = ({ mcc = 999; mnc = 70; mnc_length = 2; snssaiList = ({sst=1,sd=0xFFFFFF},{sst=2,sd=0x111111})} );
        nr_cellid = 12345688L;
        tr_s_preference = "f1";

        # This section will be overridden by command-line parameters
        MACRLCs = ({
          num_cc = 1;
          tr_s_preference = "local_L1";
          tr_n_preference = "f1";
          local_n_address = "127.0.0.1";
          remote_n_address = "127.0.0.1";
          local_n_portc = 500;
          remote_n_portc = 501;
          local_n_portd = 2153;
          remote_n_portd = 2154;
        });

        # RF simulator config
        rfsimulator = {
          serveraddr = "oai-du.oai";
          serverport = 4043;
        };
        # Other parameters copied from the original guide
        servingCellConfigCommon = {
            physCellId = 1;
            dl_frequencyBand = 78;
            dl_absoluteFrequencySSB = 641280;
            dl_absoluteFrequencyPointA = 640008;
            dl_subcarrierSpacing = 1;
            dl_carrierBandwidth = 106;
            ul_frequencyBand = 78;
            ul_subcarrierSpacing = 1;
            ul_carrierBandwidth = 106;
        };
        log_config: { global_log_level = "info"; };
    });
---

# --- ConfigMap for OAI NR-UE ---

apiVersion: v1
kind: ConfigMap
metadata:
  name: oai-nrue-config
  namespace: oai
data:
  nr-ue.conf: |
    uicc0 = {
        imsi = "999700000000001";
        key = "465B5CE8B199B49FAA5F0A2EE238A6BC";
        opc = "E8ED289DEBA952E4283B54E88E6183CA";
        dnn = "internet";
        nssai_sst = 1;
        nssai_sd = 0xFFFFFF;
    };
    # RF simulator config
    rfsimulator = {
        serveraddr = "oai-du.oai"; # Connect to DU service
        serverport = "4043";
    };
    log_config: { global_log_level = "info"; };
```

Now, apply this file to your cluster:

```bash
kubectl apply -f oai-ran-configs.yaml
```

Verify that the `ConfigMap`s and `Service`s were created:

```bash
kubectl get configmaps -n oai
kubectl get services -n oai
```

#### **Step 3: Create OAI Deployments**

Now we define the `Deployment` for each OAI component. This tells Kubernetes how to run the containers, what image to use, what commands to execute, and how to mount the configuration files.

Create a file named `oai-ran-deployments.yaml` and paste the following content.

**A Note on the Container Image:** We are using `rdefosse/oai-ran:2023.w40` which is a community-built image. You may need to change this to a different tag or your own custom-built image if required.

```yaml
# oai-ran-deployments.yaml

# --- OAI CU-CP Deployment ---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: oai-cucp-deployment
  namespace: oai
  labels:
    app: oai-cucp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: oai-cucp
  template:
    metadata:
      labels:
        app: oai-cucp
    spec:
      containers:
        - name: oai-cucp
          image: rdefosse/oai-ran:2023.w40
          imagePullPolicy: IfNotPresent
          securityContext:
            privileged: true
          command: ["/bin/bash", "-c"]
          args:
            - >
              /opt/oai/bin/nr-softmodem -O /etc/oai/oai-cucp.conf
              --gNBs.[0].local_s_address $POD_IP
              --gNBs.[0].E1_INTERFACE.[0].ipv4_cucp $POD_IP
              --gNBs.[0].NETWORK_INTERFACES.GNB_IPV4_ADDRESS_FOR_NG_AMF $POD_IP
          env:
            - name: POD_IP
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
          volumeMounts:
            - name: config-volume
              mountPath: /etc/oai
      volumes:
        - name: config-volume
          configMap:
            name: oai-cucp-config
---

# --- OAI CU-UP Deployment ---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: oai-cuup-deployment
  namespace: oai
  labels:
    app: oai-cuup
spec:
  replicas: 1
  selector:
    matchLabels:
      app: oai-cuup
  template:
    metadata:
      labels:
        app: oai-cuup
    spec:
      containers:
        - name: oai-cuup
          image: rdefosse/oai-ran:2023.w40
          imagePullPolicy: IfNotPresent
          securityContext:
            privileged: true
          command: ["/bin/bash", "-c"]
          args:
            - >
              /opt/oai/bin/nr-cuup -O /etc/oai/oai-cuup.conf
              --gNBs.[0].local_s_address $POD_IP
              --gNBs.[0].E1_INTERFACE.[0].ipv4_cuup $POD_IP
              --gNBs.[0].NETWORK_INTERFACES.GNB_IPV4_ADDRESS_FOR_NGU $POD_IP
          env:
            - name: POD_IP
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
          volumeMounts:
            - name: config-volume
              mountPath: /etc/oai
      volumes:
        - name: config-volume
          configMap:
            name: oai-cuup-config
---

# --- OAI DU Deployment ---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: oai-du-deployment
  namespace: oai
  labels:
    app: oai-du
spec:
  replicas: 1
  selector:
    matchLabels:
      app: oai-du
  template:
    metadata:
      labels:
        app: oai-du
    spec:
      containers:
        - name: oai-du
          image: rdefosse/oai-ran:2023.w40
          imagePullPolicy: IfNotPresent
          securityContext:
            privileged: true
          command: ["/bin/bash", "-c"]
          args:
            - >
              /opt/oai/bin/nr-softmodem -O /etc/oai/oai-du.conf
              --MACRLCs.[0].local_n_address $POD_IP
              --MACRLCs.[0].remote_n_address oai-cucp.oai
              --gNBs.[0].f1u_remote_address oai-cuup.oai # OAI uses this param for CU-UP
              --gNBs.[0].f1u_local_address $POD_IP
              --rfsim
          env:
            - name: POD_IP
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
          volumeMounts:
            - name: config-volume
              mountPath: /etc/oai
      volumes:
        - name: config-volume
          configMap:
            name: oai-du-config
---

# --- OAI NR-UE Deployment ---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: oai-nrue-deployment
  namespace: oai
  labels:
    app: oai-nrue
spec:
  replicas: 1
  selector:
    matchLabels:
      app: oai-nrue
  template:
    metadata:
      labels:
        app: oai-nrue
    spec:
      containers:
        - name: oai-nrue
          image: rdefosse/oai-ran:2023.w40
          imagePullPolicy: IfNotPresent
          securityContext:
            privileged: true
          command: ["/bin/bash", "-c"]
          args:
            - >
              /opt/oai/bin/nr-uesoftmodem -O /etc/oai/nr-ue.conf
              -r 106 --numerology 1 --band 78 -C 3619200000
              --rfsim --rfsimulator.serveraddr oai-du.oai
              --uicc0.imsi 999700000000001
          volumeMounts:
            - name: config-volume
              mountPath: /etc/oai
            - name: tun-dev
              mountPath: /dev/net/tun
      volumes:
        - name: config-volume
          configMap:
            name: oai-nrue-config
        - name: tun-dev
          hostPath:
            path: /dev/net/tun
```

Now, deploy all the OAI components:

```bash
kubectl apply -f oai-ran-deployments.yaml
```

#### **Step 4: Verify the OAI Deployment**

The pods will take a minute or two to start. You can monitor their status.

```bash
# Watch the pods until they are all in the 'Running' state
kubectl get pods -n oai -w
```

You should see output similar to this:

```
NAME                                   READY   STATUS    RESTARTS   AGE
oai-cucp-deployment-5f7b8c4b6c-xxxxx   1/1     Running   0          2m
oai-cuup-deployment-6b8d9c5b7d-xxxxx   1/1     Running   0          2m
oai-du-deployment-7d4f5g6h8j-xxxxx     1/1     Running   0          2m
oai-nrue-deployment-8k9l1m2n3p-xxxxx    1/1     Running   0          2m
```

Once the pods are running, you can inspect their logs to see if they are trying to connect to each other. They won't fully succeed yet because the Open5GS core is not running, but you should see initialization messages and connection attempts.

```bash
# Check the CU-CP logs - look for attempts to connect to AMF and DU
kubectl logs -n oai -l app=oai-cucp -f

# Check the DU logs - look for F1 setup with CU-CP/CU-UP
kubectl logs -n oai -l app=oai-du -f

# Check the UE logs - look for attempts to connect to the DU via rfsim
kubectl logs -n oai -l app=oai-nrue -f
```

For example, in the `oai-cucp` logs, you should see it trying to send an `NGSetupRequest` to the AMF IP (`172.17.42.27`). This is expected and correct.

---

### **Conclusion of Part 1**

You have now successfully:

1.  Configured all OAI RAN components using Kubernetes `ConfigMap`s and `Service`s.
2.  Deployed the OAI CU-CP, CU-UP, DU, and NR-UE as Kubernetes `Deployment`s.
3.  Verified that the pods are running and attempting to establish their respective connections.

The OAI RAN cluster is now fully deployed and waiting for the 5G Core to come online. In the next guide, we will deploy the Open5GS components on the `open5gs-k8s-vm` and establish the final end-to-end connection.