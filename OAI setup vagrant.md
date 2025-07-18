

This guide assumes you have already completed the prerequisite steps outlined in your `Calico BGP setup using Vagrant.md` and `Building OAI docker images.md` documents.

---

### **Guide: Deploying OAI RAN on a Kubernetes Cluster**

This guide details the final phase of your migration: deploying the OAI RAN components (CU-CP, CU-UP, gNB-DU, and nr-UE) as containerized applications on your `oai-k8s-vm` cluster and connecting them to the Open5GS Core running on the `open5gs-k8s-vm` cluster.

### **1. Architecture Overview**

We are deploying the OAI components onto the RAN cluster (`oai-k8s-vm`). The key to making this work is Kubernetes networking, which mirrors the logic from your VM-based setup.

*   **Service Discovery:** Instead of hard-coding IP addresses in configuration files, we will use Kubernetes Service names (e.g., `cucp-svc`, `du-svc`). Kubernetes DNS will resolve these names to the appropriate `ClusterIP`.
*   **Inter-Component Communication:**
    *   **DU <-> CU-CP/CU-UP (F1 Interface):** The DU will connect to the `cucp-svc` for F1-C (control plane) and the `cuup-svc` for F1-U (user plane).
    *   **CU-CP <-> CU-UP (E1 Interface):** These components will discover each other using their respective Kubernetes services.
    *   **UE <-> DU (RF-Sim Interface):** The simulated nr-UE will connect to the `du-svc` on its RF simulator port (4043).
*   **Inter-Cluster Communication (RAN -> Core):**
    *   **N2 (AMF):** The OAI CU-CP pod will connect to the Open5GS AMF service. This is achieved by pointing the configuration to the Open5GS AMF's `NodePort` service IP (`172.17.42.27`) and port.
    *   **N3 (UPF):** The OAI CU-UP pod will connect to the Open5GS UPF service, also via its `NodePort`.

### **2. Prerequisites**

Before you begin, ensure the following are complete:

1.  **Two-Cluster K8s Infrastructure:** Both `oai-k8s-vm` and `open5gs-k8s-vm` are running, with Kubernetes and Calico BGP peering established and verified as per your `Calico BGP setup using Vagrant.md` guide.
2.  **Custom OAI Docker Images:** The `oai-gnb:develop` and `oai-nr-ue:develop` Docker images have been built and pushed to your Docker Hub repository (`vishalcalsoftinc` in your logs).
3.  **Open5GS Deployed on Core Cluster:** The Open5GS core network must be running on the `open5gs-k8s-vm` cluster. If not already deployed, SSH into `open5gs-k8s-vm` and deploy it using a Helm chart.

    ```bash
    # SSH into open5gs-k8s-vm
    vagrant ssh open5gs-k8s-vm

    # Add Open5GS Helm repository
    helm repo add open5gs 'https://open5gs.github.io/open5gs/assets/helm'
    helm repo update

    # Install Open5GS, exposing AMF and UPF via NodePort
    # We use NodePort so the RAN cluster can reach them on the VM's IP
    helm install open5gs open5gs/open5gs \
      --set amf.service.type=NodePort \
      --set amf.service.nodePort=30412 \
      --set upf.service.gtpu.type=NodePort \
      --set upf.service.gtpu.nodePort=30152
    
    # Verify Open5GS pods are running
    kubectl get pods -o wide
    ```
4. **Register Subscriber in Open5GS**: You must register the IMSI that the OAI UE will use. You can do this by port-forwarding the Open5GS WebUI.
    ```bash
    # In a new terminal on your host, port-forward the WebUI
    vagrant ssh open5gs-k8s-vm -- -L 3000:localhost:3000
    
    # Inside the ssh session
    kubectl port-forward svc/open5gs-webui 3000:80
    
    # Now, open http://localhost:3000 on your host machine's browser.
    # Login with admin/1423 and add the subscriber with IMSI `999700000000010` and the keys from your ue-depl-cm.yaml.
    ```
---

### **3. Deploying OAI Components on the RAN Cluster**

All the following commands should be run inside the `oai-k8s-vm`.

```bash
# SSH into the OAI RAN machine
vagrant ssh oai-k8s-vm
```

#### **Step 1: Create Namespace**

It's good practice to isolate our 5G components in their own namespace.

```bash
kubectl create namespace oai-test
```

#### **Step 2: Deploy OAI CU-CP**

The CU-CP handles the RRC and the control plane part of the F1/E1/N2 interfaces.

Create a file named `oai-cucp.yaml` and paste the combined content of your `cucp-depl-cm.yaml` and `cucp-service.yaml` files.

```yaml
# oai-cucp.yaml (Corrected)
apiVersion: v1
kind: Service
metadata:
  name: cucp-svc
  namespace: oai-test
spec:
  selector:
    app: oai-cucp
  ports:
    - name: f1-c
      port: 38472
      targetPort: 38472
      protocol: SCTP
    - name: e1
      port: 38462
      targetPort: 38462
      protocol: SCTP
    - name: n2
      port: 38412
      targetPort: 38412
      protocol: SCTP
  type: ClusterIP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: oai-cucp
  namespace: oai-test
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
        image: vishalcalsoftinc/oai-gnb:develop # FIXED: Using your custom image
        securityContext:
          privileged: true
        command: ["/bin/bash", "-c"]
        # FIXED: The entire command is now a single, clean, multi-line string
        args:
          - >
            /opt/oai/nr-softmodem -O /opt/oai-gnb/etc/gnb.conf --log_config.global_log_options level,nocolor,time --gNBs.[0].local_s_address $(POD_IP) --gNBs.[0].remote_s_address du-svc --gNBs.[0].NETWORK_INTERFACES.GNB_IPV4_ADDRESS_FOR_NG_AMF $(POD_IP) --gNBs.[0].E1_INTERFACE.[0].ipv4_cucp $(POD_IP) --gNBs.[0].E1_INTERFACE.[0].ipv4_cuup cuup-svc;
        env:
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: ASAN_OPTIONS
          value: "detect_leaks=0"
        volumeMounts:
        - mountPath: /opt/oai-gnb/etc/gnb.conf
          subPath: gnb.conf
          name: config-volume
        # FIXED: Correct syntax for probes
        readinessProbe:
          exec:
            command: ["/bin/bash", "-c", "pgrep nr-softmodem"]
          initialDelaySeconds: 15
          periodSeconds: 10
        livenessProbe:
          exec:
            command: ["/bin/bash", "-c", "pgrep nr-softmodem"]
          initialDelaySeconds: 15
          periodSeconds: 30
      volumes:
      - name: config-volume
        configMap:
          name: oai-cucp-config
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: oai-cucp-config
  namespace: oai-test
data:
  gnb.conf: |
    Active_gNBs = ( "gNB-OAI");
    Asn1_verbosity = "info";
    gNBs = (
     {
        gNB_ID = 0xe00;
        gNB_name  =  "gNB-OAI";
        tracking_area_code  =  1;
        plmn_list = ({ mcc = 999; mnc = 70; mnc_length = 2; snssaiList = ({ sst = 1, sd=0x111111}) });
        tr_s_preference = "f1";
        local_s_portc   = 501;
        remote_s_portc  = 38472;
        local_s_portd   = 2154;
        remote_s_portd  = 2153;
        SCTP: { SCTP_INSTREAMS = 2; SCTP_OUTSTREAMS = 2; };
        amf_ip_address = ({ ipv4 = "172.17.42.27"; });
        E1_INTERFACE = ( { type = "cp"; port_cucp = 38462; port_cuup = 38462; } );
        NETWORK_INTERFACES : { };
      }
    );
    security = { ciphering_algorithms = ("nea0"); integrity_algorithms = ("nia2", "nia0"); drb_ciphering = "yes"; drb_integrity = "no"; };
    log_config : { global_log_level = "info"; f1ap_log_level = "info"; ngap_log_level = "info"; };
```

**Deploy it:**

```bash
kubectl apply -f oai-cucp.yaml
```




#### **Step 3: Deploy OAI CU-UP**

The CU-UP handles the user plane GTP-U tunnels for the F1-U and N3 interfaces.

Create a file named `oai-cuup.yaml`.

```yaml
# oai-cuup.yaml (Corrected)
apiVersion: v1
kind: Service
metadata:
  name: cuup-svc
  namespace: oai-test
spec:
  selector:
    app: oai-cuup
  type: ClusterIP
  ports:
    - name: e1-interface
      port: 38462
      targetPort: 38462
      protocol: SCTP
    - name: gtp-u-n3
      port: 2152
      targetPort: 2152
      protocol: UDP
    - name: f1-u
      port: 2153
      targetPort: 2153
      protocol: UDP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: oai-cuup
  namespace: oai-test
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
        image: vishalcalsoftinc/oai-gnb:develop # FIXED: Using your custom image
        securityContext:
          privileged: true
        command: ["/bin/bash", "-c"]
        # FIXED: The entire command is now a single, clean, multi-line string
        args:
          - >
            /opt/oai/nr-cuup -O /opt/oai-gnb/etc/gnb.conf --log_config.global_log_options level,nocolor,time --gNBs.[0].local_s_address $(POD_IP) --gNBs.[0].remote_s_address du-svc --gNBs.[0].NETWORK_INTERFACES.GNB_IPV4_ADDRESS_FOR_NGU $(POD_IP) --gNBs.[0].E1_INTERFACE.[0].ipv4_cucp cucp-svc --gNBs.[0].E1_INTERFACE.[0].ipv4_cuup $(POD_IP);
        env:
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: ASAN_OPTIONS
          value: "detect_leaks=0"
        volumeMounts:
        - mountPath: /opt/oai-gnb/etc/gnb.conf
          subPath: gnb.conf
          name: config-volume
        # FIXED: Correct syntax for probes
        readinessProbe:
          exec:
            command: ["/bin/bash", "-c", "pgrep nr-cuup"]
          initialDelaySeconds: 15
          periodSeconds: 10
        livenessProbe:
          exec:
            command: ["/bin/bash", "-c", "pgrep nr-cuup"]
          initialDelaySeconds: 15
          periodSeconds: 30
      volumes:
      - name: config-volume
        configMap:
          name: oai-cuup-config
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: oai-cuup-config
  namespace: oai-test
data:
  gnb.conf: |
    Active_gNBs = ( "gNB-OAI");
    Asn1_verbosity = "info";
    gNBs = (
     {
        gNB_ID = 0xe00;
        gNB_CU_UP_ID = 0xe01;
        gNB_name  =  "gNB-OAI-CUUP";
        tracking_area_code  =  1;
        plmn_list = ({ mcc = 999; mnc = 70; mnc_length = 2; snssaiList = ({ sst = 1, sd=0x111111}) });
        tr_s_preference = "f1";
        local_s_portd   = 2153;
        remote_s_portd  = 2152;
        SCTP : { SCTP_INSTREAMS  = 2; SCTP_OUTSTREAMS = 2; };
        E1_INTERFACE = ( { type = "up"; ipv4_cucp = "cucp-svc"; port_cucp = 38462; port_cuup = 38462; } );
        NETWORK_INTERFACES : { GNB_IPV4_ADDRESS_FOR_NGU_UPF = "172.17.42.27"; GNB_PORT_FOR_NGU_UPF = 30152; GNB_PORT_FOR_NGU = 2152; };
      }
    );
    log_config : { global_log_level = "info"; pdcp_log_level = "info"; f1ap_log_level = "info"; };
```

**Deploy it:**

```bash
kubectl apply -f oai-cuup.yaml
```

#### **Step 4: Deploy OAI DU (gNB-DU)**

The DU handles the lower layers of the RAN and connects to the CUs via F1. In this setup, it also runs the RF simulator.

Create a file named `oai-du.yaml`.

```yaml
# oai-du.yaml (Corrected)
apiVersion: v1
kind: Service
metadata:
  name: du-svc
  namespace: oai-test
spec:
  selector:
    app: oai-du
  type: ClusterIP
  ports:
    - name: f1-c
      protocol: SCTP
      port: 38472
      targetPort: 38472
    - name: f1-u
      protocol: UDP
      port: 2152
      targetPort: 2152
    - name: rfsim
      protocol: TCP
      port: 4043
      targetPort: 4043
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: oai-du
  namespace: oai-test
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
        image: vishalcalsoftinc/oai-gnb:develop # FIXED: Using your custom image
        securityContext:
          privileged: true
        command: ["/bin/bash", "-c"]
        # FIXED: The entire command is now a single, clean, multi-line string
        args:
          - >
            /opt/oai/nr-softmodem -O /opt/oai-gnb/etc/gnb.conf --rfsim --log_config.global_log_options level,nocolor,time --MACRLCs.[0].local_n_address $(POD_IP) --MACRLCs.[0].remote_n_address cucp-svc --MACRLCs.[0].local_n_address_f1u $(POD_IP) --MACRLCs.[0].remote_n_address_f1u cuup-svc;
        env:
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: ASAN_OPTIONS
          value: "detect_leaks=0"
        volumeMounts:
          - name: gnb-config
            mountPath: /opt/oai-gnb/etc/gnb.conf
            subPath: gnb.conf
        # FIXED: Correct syntax for probes
        readinessProbe:
          exec:
            command: ["/bin/bash", "-c", "pgrep nr-softmodem"]
          initialDelaySeconds: 15
          periodSeconds: 10
        livenessProbe:
          exec:
            command: ["/bin/bash", "-c", "pgrep nr-softmodem"]
          initialDelaySeconds: 15
          periodSeconds: 30
      volumes:
        - name: gnb-config
          configMap:
            name: oai-du-config
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: oai-du-config
  namespace: oai-test
data:
  gnb.conf: |
    Active_gNBs = ( "gNB-OAI");
    Asn1_verbosity = "info";
    gNBs = ( {
        gNB_ID = 0xe00; gNB_DU_ID = 0xe01; gNB_name  = "gNB-OAI-DU";
        tracking_area_code = 1;
        plmn_list = ({ mcc = 999; mnc = 70; mnc_length = 2; snssaiList = ({ sst = 1, sd=0x111111}) });
        nr_cellid = 12345678L;
        servingCellConfigCommon = ({
          physCellId = 0; absoluteFrequencySSB = 641280; dl_frequencyBand = 78; dl_absoluteFrequencyPointA = 640008;
          dl_offstToCarrier = 0; dl_subcarrierSpacing = 1; dl_carrierBandwidth = 106; initialDLBWPlocationAndBandwidth = 28875;
          initialDLBWPsubcarrierSpacing = 1; initialDLBWPcontrolResourceSetZero = 12; initialDLBWPsearchSpaceZero = 0;
          ul_frequencyBand = 78; ul_offstToCarrier = 0; ul_subcarrierSpacing = 1; ul_carrierBandwidth = 106; pMax = 20;
          initialULBWPlocationAndBandwidth = 28875; initialULBWPsubcarrierSpacing = 1; prach_ConfigurationIndex = 98;
          prach_msg1_FDM = 0; prach_msg1_FrequencyStart = 0; zeroCorrelationZoneConfig = 13; preambleReceivedTargetPower = -96;
          preambleTransMax = 6; powerRampingStep = 1; ra_ResponseWindow = 4; ssb_perRACH_OccasionAndCB_PreamblesPerSSB_PR = 4;
          ssb_perRACH_OccasionAndCB_PreamblesPerSSB = 14; ra_ContentionResolutionTimer = 7; rsrp_ThresholdSSB = 19;
          prach_RootSequenceIndex_PR = 2; prach_RootSequenceIndex = 1; msg1_SubcarrierSpacing = 1; restrictedSetConfig = 0;
          msg3_DeltaPreamble = 1; p0_NominalWithGrant =-90; pucchGroupHopping = 0; hoppingId = 40; p0_nominal = -90;
          ssb_PositionsInBurst_Bitmap = 1; ssb_periodicityServingCell = 2; dmrs_TypeA_Position = 0; subcarrierSpacing = 1;
          referenceSubcarrierSpacing = 1; dl_UL_TransmissionPeriodicity = 6; nrofDownlinkSlots = 7; nrofDownlinkSymbols = 6;
          nrofUplinkSlots = 2; nrofUplinkSymbols = 4; ssPBCH_BlockPower = -25;
         });
        SCTP : { SCTP_INSTREAMS = 2; SCTP_OUTSTREAMS = 2; }; }
    );
    MACRLCs = ({ num_cc = 1; tr_s_preference = "local_L1"; tr_n_preference = "f1"; local_n_portc = 38472; remote_n_portc = 38472; local_n_portd = 2152; remote_n_portd = 2153; });
    L1s = ({ num_cc = 1; tr_n_preference = "local_mac"; });
    RUs = ({ local_rf = "yes"; nb_tx = 1; nb_rx = 1; att_tx = 0; att_rx = 0; bands = [78]; max_pdschReferenceSignalPower = -27; max_rxgain = 114; eNB_instances = [0]; clock_src = "internal"; });
    rfsimulator: { serveraddr = "server"; serverport = 4043; };
    log_config: { global_log_level = "info"; };
```

**Deploy it:**

```bash
kubectl apply -f oai-du.yaml
```

#### **Step 5: Deploy OAI NR-UE**

Finally, we deploy the simulated User Equipment.

Create a file named `oai-ue.yaml`.

```yaml
# oai-ue.yaml (Corrected)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: oai-nr-ue
  namespace: oai-test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: oai-nr-ue
  template:
    metadata:
      labels:
        app: oai-nr-ue
    spec:
      containers:
        - name: oai-nr-ue
          image: vishalcalsoftinc/oai-nr-ue:develop # FIXED: Using your custom UE image
          securityContext:
            privileged: true
          command: ["/bin/bash", "-c"]
          # FIXED: The entire command is now a single, clean, multi-line string
          args:
            - >
              /opt/oai/nr-uesoftmodem -O /opt/oai-nr-ue/etc/nr-ue.conf --rfsim -r 106 --numerology 1 -C 3619200000 --rfsimulator.serveraddr du-svc --uicc0.imsi 999700000000010 --log_config.global_log_options level,nocolor,time;
          # FIXED: Correct syntax for probes
          readinessProbe:
            exec:
              command: ["/bin/bash", "-c", "pgrep nr-uesoftmodem"]
            initialDelaySeconds: 20
            periodSeconds: 10
          livenessProbe:
            exec:
              command: ["/bin/bash", "-c", "pgrep nr-uesoftmodem"]
            initialDelaySeconds: 20
            periodSeconds: 30
          volumeMounts:
            - name: nrue-conf
              mountPath: /opt/oai-nr-ue/etc/nr-ue.conf
              subPath: nrue.uicc.conf
      volumes:
        - name: nrue-conf
          configMap:
            name: nrue-config
      restartPolicy: Always
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nrue-config
  namespace: oai-test
data:
  nrue.uicc.conf: |
    uicc0 = {
      key = "465B5CE8B199B49FAA5F0A2EE238A6BC";
      opc= "E8ED289DEBA952E4283B54E88E6183CA";
      dnn= "internet";
      nssai_sst=1;
      nssai_sd=0x111111;
    };
```
**Deploy it:**

```bash
kubectl apply -f oai-ue.yaml
```

---
### **4. Verification and Testing**

#### **Step 1: Check Pod and Service Status**

After a few minutes, all pods should be in the `Running` state.

```bash
kubectl get pods,svc -n oai-test -o wide
```
You should see pods for cucp, cuup, du, and nr-ue, along with their services.

#### **Step 2: Check Component Logs**

Tailing the logs is crucial for debugging. Check them in a logical order.

1.  **CU-CP Logs:** It should establish a connection with the AMF.
    ```bash
    kubectl logs -n oai-test -f deploy/oai-cucp
    ```
    Look for `[NGAP] Sent NGSetupRequest to AMF` and `[NGAP] Received NGSetupResponse from AMF`.

2.  **DU Logs:** It should connect to the CU-CP.
    ```bash
    kubectl logs -n oai-test -f deploy/oai-du
    ```
    Look for `[F1AP] Sent F1 SETUP REQUEST` and `[F1AP] Received F1 SETUP RESPONSE`.

3.  **NR-UE Logs:** The UE should find the DU, start the registration process, and connect.
    ```bash
    kubectl logs -n oai-test -f deploy/oai-nr-ue
    ```
    Look for messages indicating successful RRC connection and registration.

#### **Step 3: End-to-End Connectivity Test**

This is the final test to confirm data plane connectivity from the UE to the internet through the entire 5G chain.

1.  **Exec into the NR-UE pod:**
    ```bash
    kubectl exec -it -n oai-test deploy/oai-nr-ue -- /bin/bash
    ```

2.  **Inside the pod, check for the tunnel interface:**
    ```bash
    # You are now inside the UE pod
    ip a
    ```
    You should see a new interface, `oaitun_ue1`, with an IP address from the Open5GS UPF's pool (e.g., `10.45.0.2`).

3.  **Ping the internet through the tunnel:**
    ```bash
    ping -I oaitun_ue1 8.8.8.8
    ```

If you receive replies, congratulations! You have successfully migrated the OAI RAN to a Kubernetes-native deployment and established a full end-to-end 5G connection.