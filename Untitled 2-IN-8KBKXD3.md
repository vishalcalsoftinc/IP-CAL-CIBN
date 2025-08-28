
Of course. It is absolutely possible to implement a multi-UPF (User Plane Function) handover on your existing GCP setup. Your current guide masterfully details an **F1 Handover**, which is a RAN-level procedure where the UE is handed off between two DUs under the control of the same CU. This process is largely transparent to the 5G Core.

A **Multi-UPF Handover**, by contrast, is a core network procedure initiated by the SMF. It involves changing the UPF that anchors a UE's PDU session. This is often triggered by UE mobility across different geographical areas (each served by a different UPF), network slice requirements, or specific traffic steering policies.

Here’s a breakdown of how you can adapt your current setup to achieve a multi-UPF handover, leveraging the components you already have.

### **Conceptual Overview: F1 Handover vs. Multi-UPF Handover**

| Feature | Your Current F1 Handover | Proposed Multi-UPF Handover |
| :--- | :--- | :--- |
| **Primary Actors** | UE, OAI-DU0, OAI-DU1, OAI-CU | UE, OAI-RAN, Open5GS-SMF, Open5GS-UPF1, Open5GS-UPF2 |
| **Trigger** | RAN-internal event (e.g., measurement report, or your manual `ci_trigger_f1_ho` command). | Core Network policy (e.g., UE moves to a new Tracking Area) which triggers a PDU Session Modification. |
| **Key Interface** | F1-AP (between CU and DU). | N4 (PFCP between SMF and UPFs) and N2 (between AMF/RAN). |
| **Core Network Role**| Minimal. The AMF is informed of the path switch, but the SMF/UPF are typically unaffected. | Central. The SMF is the decision-maker, orchestrating the change of the user-plane anchor. |
| **Goal** | Maintain connectivity as UE moves between cells controlled by the same CU. | Reroute the UE's data path to a new, potentially more optimal, UPF for reasons like latency or topology. |

---

### **Step-by-Step Guide to Implement Multi-UPF Handover**

The strategy is to deploy a second UPF, make the SMF aware of both, and then create a policy that triggers a UPF change when the UE performs the F1 handover you have already implemented.

#### **Part 1: Deploy a Second Open5GS UPF**

You need a second UPF instance. The simplest way is to create a new, small VM for this role.

1.  **Reserve a New Static IP:**
    *   In the GCP console, navigate to **VPC network > IP addresses**.
    *   Reserve a new internal static IP: `172.17.0.96` (for `oai-upf2-vm`).

2.  **Create the UPF VM (`oai-upf2-vm`):**
    *   Navigate to **Compute Engine > VM instances** and create a new VM.
    *   **Name:** `oai-upf2-vm`
    *   **Machine type:** `e2-small` is sufficient as it only runs the UPF.
    *   **Boot disk:** Ubuntu 24.04 LTS.
    *   **Networking:**
        *   **Network:** `oai-5g-vpc`
        *   **Primary internal IP:** Select the reserved IP `172.17.0.96`.
        *   **Enable IP forwarding:** Check this box.

3.  **Install and Configure Open5GS UPF on the New VM:**
    *   SSH into `oai-upf2-vm`.
    *   Install Open5GS just as you did on the main core VM:
        ```bash
        sudo add-apt-repository ppa:open5gs/latest
        sudo apt update
        sudo apt install open5gs -y
        ```
    *   **Configure `upf.yaml`:** This is the only file you need to edit on this VM.
        ```bash
        sudo nano /etc/open5gs/upf.yaml
        ```
        Modify the `gtpu` and `pfcp` server addresses to use this VM's IP.
        ```yaml
        upf:
          pfcp:
            server:
              - address: 172.17.0.96 # This VM's IP
          gtpu:
            server:
              - address: 172.17.0.96 # This VM's IP
        ```
    *   **Start and Enable only the UPF service:**
        ```bash
        sudo systemctl start open5gs-upfd
        sudo systemctl enable open5gs-upfd
        ```

#### **Part 2: Configure the SMF to Manage Both UPFs**

Now, tell your main Open5GS SMF about both UPFs.

1.  **SSH into your main Open5GS VM (`open5gs-vm`).**
2.  **Modify `smf.yaml`:**
    ```bash
    sudo nano /etc/open5gs/smf.yaml
    ```
3.  **Update the UPF Pool:** Edit the `upf` section to list both your original UPF and the new one. You can associate them with different selection criteria, like a DNN or Tracking Area Identifier (TAI). We will use TAI for the handover trigger.

    ```yaml
    smf:
      upf:
        pool:
          # Original UPF, serving our primary tracking area
          - id: 1
            address: 172.17.0.95
            tai:
              - plmn_id:
                  mcc: 999
                  mnc: 70
                tac: 1 # We'll associate DU0 with TAC 1

          # New UPF, serving our secondary tracking area
          - id: 2
            address: 172.17.0.96
            tai:
              - plmn_id:
                  mcc: 999
                  mnc: 70
                tac: 2 # We'll associate DU1 with TAC 2
    ```
4.  **Restart the SMF:**
    ```bash
    sudo systemctl restart open5gs-smfd
    ```

#### **Part 3: Configure the RAN to Support Different Tracking Areas**

The trigger for our UPF handover will be the UE moving into a new Tracking Area. We will map your existing DU0 and DU1 to different Tracking Area Codes (TACs).

1.  **SSH into your CU VM (`oai-cu-vm`).**
2.  **Modify `oai-cu.conf`:** You need to assign a different `tracking_area_code` to the gNB definition associated with each DU. Since you have a consolidated CU config, you can define different TACs for different cells if your OAI version supports it, or more simply, associate each DU with a different TAC in the gNB list.

    Let's modify your existing configuration to represent two distinct cells/gNBs, one for each DU, each with a different TAC.

    ```bash
    # In /etc/oai/oai-cu.conf
    
    gNBs = (
      {
        gNB_ID = 0xe01;
        gNB_name = "gNB-for-DU0";
        tracking_area_code = 1; # TAC for DU0
        plmn_list = ({ mcc = 999; mnc = 70; ... });
        # ... Other params for DU0
        remote_s_address = "172.17.0.92"; // du0-vm-ip
        ...
      },
      {
        gNB_ID = 0xe02;
        gNB_name = "gNB-for-DU1";
        tracking_area_code = 2; # TAC for DU1
        plmn_list = ({ mcc = 999; mnc = 70; ... });
        # ... Other params for DU1
        remote_s_address = "172.17.0.94"; // du1-vm-ip
        ...
      }
    );
    ```
    *Note: The exact syntax for a multi-gNB/multi-TAC configuration in a single CU may vary by OAI version. The goal is to ensure that when the UE is on DU1, the CU reports its location as TAC 2.*

#### **Part 4: Execute and Verify the Handover**

You will now use your existing F1 handover trigger, but the underlying effect will be different.

1.  **Start All Network Components** exactly as you do in Part 7 of your guide.
    *   Start Open5GS services (including the new `open5gs-upfd` on `oai-upf2-vm`).
    *   Start the CU, UE (as RF Server), and Source DU (DU0).
    *   **Initial State Verification:** The UE will attach to DU0. Check the SMF logs (`/var/log/open5gs/smf.log`) on `open5gs-vm`. You should see it select **UPF1 (172.17.0.95)** for the PDU session because the UE is in TAC 1.
    *   Start a continuous ping from the UE: `ping -I oaitun_ue1 8.8.8.8`

2.  **Start the Target DU (DU1).**

3.  **Trigger the Handover:**
    *   Run your existing F1 handover command:
        ```bash
        echo ci trigger_f1_ho | nc 172.17.0.93 9090
        ```

4.  **Verify the Multi-UPF Handover:**
    *   **RAN Logs:** Your OAI logs will show the F1 handover completing, just like before.
    *   **Core Network Logs (The New Part):**
        *   The CU informs the AMF of the handover to a cell with TAC 2.
        *   The AMF informs the SMF of the UE's new location.
        *   Check `/var/log/open5gs/smf.log` on `open5gs-vm`. You should see log entries indicating a **PDU Session Modification** procedure. The SMF will see the new TAI, apply its policy, and decide to switch the anchor to **UPF2 (172.17.0.96)**.
        *   You will see **PFCP Session Modification Request** messages being sent to the old UPF1 (to release resources) and **PFCP Session Establishment Request** messages to the new UPF2.
    *   **Data Connectivity:** Your continuous `ping` on the UE should survive the handover with minimal interruption. This confirms the data plane was successfully switched from UPF1 to UPF2.


# upf2.yaml
```
logger:
  file:
    path: /var/log/open5gs/upf.log
#  level: info   # fatal|error|warn|info(default)|debug|trace

global:
  max:
    ue: 1024  # The number of UE can be increased depending on memory size.
#    peer: 64

upf:
  pfcp:
    server:
      # MODIFIED: Changed from 127.0.0.7 to the VM's static IP.
      # This is the IP the SMF will use to send control messages to this UPF.
      - address: 172.17.0.96
    client:
#      smf:     #  UPF PFCP Client try to associate SMF PFCP Server
#        - address: 127.0.0.4
  gtpu:
    server:
      # MODIFIED: Changed from 127.0.0.7 to the VM's static IP.
      # This is the IP the RAN (CU-UP) will send user data (GTP-U packets) to.
      - address: 172.17.0.96
  session:
    - subnet: 10.45.0.0/16
      gateway: 10.45.0.1
    - subnet: 2001:db8:cafe::/48
      gateway: 2001:db8:cafe::1
  metrics:
    server:
      - address: 127.0.0.7
        port: 9090

# All sections below this line can remain as default.
################################################################################
# PFCP Server
################################################################################
# o Override PFCP address to be advertised to SMF in PFCP association
#  pfcp:
#    server:
#      - dev: eth0
#        advertise: open5gs-upf.svc.local
#
################################################################################
# GTP-U Server
################################################################################
#  o Override SGW-U GTP-U address to be advertised inside S1AP messages
#  gtpu:
#    server:
#      - dev: ens3
#        advertise: upf1.5gc.mnc001.mcc001.3gppnetwork.org
#
#  o User Plane IP Resource information
#  gtpu:
#    server:
#      - address:
#        - 127.0.0.7
#        - ::1
#        teid_range_indication: 4
#        teid_range: 10
#        network_instance: internet
#        source_interface: 0
#      - address: 127.0.10.4
#        teid_range_indication: 4
#        teid_range: 5
#        network_instance: ims
#        source_interface: 1
#
################################################################################
# 3GPP Specification
################################################################################
#
#  o Specific DNN/APN(e.g 'ims') uses 10.46.0.1/16, 2001:db8:babe::1/48
#  $ sudo ip addr add 10.45.0.1/16 dev ogstun
#  $ sudo ip addr add 2001:db8:cafe::1/48 dev ogstun2
#  $ sudo ip addr add 10.46.0.1/16 dev ogstun3
#  $ sudo ip addr add 2001:db8:babe::1/48 dev ogstun3
#
#  session:
#    - subnet: 10.45.0.0/16
#      gateway: 10.45.0.1
#      dnn: internet
#    - subnet: 2001:db8:cafe::/48
#      dnn: internet
#      dev: ogstun2
#    - subnet: 10.46.0.0/16
#      gateway: 10.46.0.1
#      dnn: ims
#      dev: ogstun3
#    - subnet: 2001:db8:babe::/48
#      dnn: ims
#      dev: ogstun3
```


# smf.yaml

```bash
logger:
  file:
    path: /var/log/open5gs/smf.log
#  level: info   # fatal|error|warn|info(default)|debug|trace

global:
  max:
    ue: 1024
#    peer: 64

smf: 
  sbi:
    server:
      # MODIFIED: Changed to the VM's static IP
      - address: 172.17.0.95
        port: 7777
    client:
#      nrf:
#        - uri: http://127.0.0.10:7777
      scp:
        - uri: http://127.0.0.200:7777
  pfcp:
    server:
      # MODIFIED: Changed to the VM's static IP
      - address: 172.17.0.95
    # The old client section is removed and replaced by the upf.pool below

  # THIS IS THE NEW CRITICAL SECTION FOR MULTI-UPF
  upf:
    pool:
      # Entry for the first UPF (on this same VM)
      - id: 1
        address: 172.17.0.95
        tai:
          - plmn_id:
              mcc: 999
              mnc: 70
            tac: 1 # Assign this UPF to Tracking Area Code 1

      # Entry for the second UPF (on the new upf2-vm)
      - id: 2
        address: 172.17.0.96
        tai:
          - plmn_id:
              mcc: 999
              mnc: 70
            tac: 2 # Assign this UPF to Tracking Area Code 2

  gtpc:
    server:
      # MODIFIED: Changed to the VM's static IP
      - address: 172.17.0.95
  gtpu:
    server:
      # MODIFIED: Changed to the VM's static IP
      - address: 172.17.0.95
  metrics:
    server:
      # MODIFIED: Changed to the VM's static IP
      - address: 172.17.0.95
        port: 9090
  session:
    - subnet: 10.45.0.0/16
      gateway: 10.45.0.1
    - subnet: 2001:db8:cafe::/48
      gateway: 2001:db8:cafe::1
  dns:
    - 8.8.8.8
    - 8.8.4.4
    - 2001:4860:4860::8888
    - 2001:4860:4860::8844
  mtu: 1400
#  p-cscf:
#    - 127.0.0.1
#    - ::1
#  ctf:
#    enabled: auto   # auto(default)|yes|no
  freeDiameter: /etc/freeDiameter/smf.conf

# All sections below this line can remain as default.
# The `info` section is for NRF registration and is not needed for this TAI-based UPF selection to work.
# The logic is now handled by the `upf.pool` above.
################################################################################
# SMF Info
################################################################################
#  <SMF Selection - 5G Core only>
#  1. SMF sends SmfInfo(S-NSSAI, DNN, TAI) to the NRF
#  2. NRF responds to AMF with SmfInfo during NF-Discovery.
#  3. AMF selects SMF based on S-NSSAI, DNN and TAI in SmfInfo.
#
#  Note that if there is no SmfInfo, any AMF can select this SMF.
#
#  o S-NSSAI[SST:1] and DNN[internet] - At least 1 DNN is required in S-NSSAI
#  info:
#    - s_nssai:
#        - sst: 1
#          dnn:
#            - internet
#
```