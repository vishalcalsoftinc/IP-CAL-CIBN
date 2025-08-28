
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



```
telcomaan@open5gs-vm:~$ sudo systemctl restart open5gs-amfd  
telcomaan@open5gs-vm:~$ sudo journalctl -u open5gs-smfd -n 50 --no-pager
Aug 14 10:05:24 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[580]: 08/14 10:05:24.320: [app] INFO: SMF terminate...done (../src/smf/app.c:39)
Aug 14 10:05:25 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal systemd[1]: open5gs-smfd.service: Deactivated successfully.
Aug 14 10:05:25 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal systemd[1]: Stopped open5gs-smfd.service - Open5GS SMF Daemon.
Aug 14 10:05:25 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal systemd[1]: open5gs-smfd.service: Consumed 1.973s CPU time.
-- Boot 8d99f7d6f9e949c99fd5b4004316029e --
Aug 14 10:27:23 open5gs-vm systemd[1]: Started open5gs-smfd.service - Open5GS SMF Daemon.
Aug 14 10:27:23 open5gs-vm open5gs-smfd[579]: Open5GS daemon v2.7.6
Aug 14 10:27:23 open5gs-vm open5gs-smfd[579]: 08/14 10:27:23.418: [app] INFO: Configuration: '/etc/open5gs/smf.yaml' (../lib/app/ogs-init.c:144)
Aug 14 10:27:23 open5gs-vm open5gs-smfd[579]: 08/14 10:27:23.418: [app] INFO: File Logging: '/var/log/open5gs/smf.log' (../lib/app/ogs-init.c:147)
Aug 14 10:27:24 open5gs-vm open5gs-smfd[579]: 08/14 10:27:24.347: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.200:7777] (../lib/sbi/context.c:507)
Aug 14 10:27:24 open5gs-vm open5gs-smfd[579]: 08/14 10:27:24.348: [metrics] INFO: metrics_server() [http://127.0.0.4]:9090 (../lib/metrics/prometheus/context.c:300)
Aug 14 10:27:24 open5gs-vm open5gs-smfd[579]: 08/14 10:27:24.936: [app] INFO: Polling freeDiameter stats every 60000000 usecs (../lib/diameter/common/stats.c:77)
Aug 14 10:27:24 open5gs-vm open5gs-smfd[579]: 08/14 10:27:24.943: [gtp] INFO: gtp_server() [127.0.0.4]:2123 (../lib/gtp/path.c:30)
Aug 14 10:27:24 open5gs-vm open5gs-smfd[579]: 08/14 10:27:24.943: [gtp] INFO: gtp_server() [127.0.0.4]:2152 (../lib/gtp/path.c:30)
Aug 14 10:27:24 open5gs-vm open5gs-smfd[579]: 08/14 10:27:24.943: [pfcp] INFO: pfcp_server() [127.0.0.4]:8805 (../lib/pfcp/path.c:30)
Aug 14 10:27:24 open5gs-vm open5gs-smfd[579]: 08/14 10:27:24.947: [sbi] INFO: NF Service [nsmf-pdusession] (../lib/sbi/context.c:1994)
Aug 14 10:27:24 open5gs-vm open5gs-smfd[579]: 08/14 10:27:24.947: [sbi] INFO: nghttp2_server() [http://127.0.0.4]:7777 (../lib/sbi/nghttp2-server.c:439)
Aug 14 10:27:24 open5gs-vm open5gs-smfd[579]: 08/14 10:27:24.948: [app] INFO: SMF initialize...done (../src/smf/app.c:31)
Aug 14 10:27:24 open5gs-vm open5gs-smfd[579]: 08/14 10:27:24.949: [smf] INFO: PFCP associated [127.0.0.7]:8805 (../src/smf/pfcp-sm.c:188)
Aug 14 10:27:24 open5gs-vm open5gs-smfd[579]: 08/14 10:27:24.960: [sbi] INFO: [44a7d1b0-78f9-41f0-88c0-5bc67d973425] NF registered [Heartbeat:10s] (../lib/sbi/nf-sm.c:295)
Aug 14 10:27:24 open5gs-vm open5gs-smfd[579]: 08/14 10:27:24.976: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.10:7777] (../lib/sbi/nnrf-handler.c:955)
Aug 14 10:27:24 open5gs-vm open5gs-smfd[579]: 08/14 10:27:24.976: [sbi] INFO: [45141dca-78f9-41f0-a1b3-3fa32fc5945f] Subscription created until 2025-08-15T10:27:24.962197+00:00 [duration:86400000000,validity:86400.000000,patch:43200.000000] (../lib/sbi/nnrf-handler.c:874)
Aug 14 10:27:24 open5gs-vm open5gs-smfd[579]: 08/14 10:27:24.977: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.10:7777] (../lib/sbi/nnrf-handler.c:955)
Aug 14 10:27:24 open5gs-vm open5gs-smfd[579]: 08/14 10:27:24.977: [sbi] INFO: [45142a40-78f9-41f0-a1b3-3fa32fc5945f] Subscription created until 2025-08-15T10:27:24.962482+00:00 [duration:86400000000,validity:86400.000000,patch:43200.000000] (../lib/sbi/nnrf-handler.c:874)
Aug 14 10:27:24 open5gs-vm open5gs-smfd[579]: 08/14 10:27:24.978: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.10:7777] (../lib/sbi/nnrf-handler.c:955)
Aug 14 10:27:24 open5gs-vm open5gs-smfd[579]: 08/14 10:27:24.978: [sbi] INFO: [45142356-78f9-41f0-a1b3-3fa32fc5945f] Subscription created until 2025-08-15T10:27:24.962308+00:00 [duration:86400000000,validity:86400.000000,patch:43200.000000] (../lib/sbi/nnrf-handler.c:874)
Aug 14 10:27:24 open5gs-vm open5gs-smfd[579]: 08/14 10:27:24.978: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.10:7777] (../lib/sbi/nnrf-handler.c:955)
Aug 14 10:27:24 open5gs-vm open5gs-smfd[579]: 08/14 10:27:24.978: [sbi] INFO: [45142658-78f9-41f0-a1b3-3fa32fc5945f] Subscription created until 2025-08-15T10:27:24.962380+00:00 [duration:86400000000,validity:86400.000000,patch:43200.000000] (../lib/sbi/nnrf-handler.c:874)
Aug 14 10:27:24 open5gs-vm open5gs-smfd[579]: 08/14 10:27:24.978: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.10:7777] (../lib/sbi/nnrf-handler.c:955)
Aug 14 10:27:24 open5gs-vm open5gs-smfd[579]: 08/14 10:27:24.978: [sbi] INFO: [45142df6-78f9-41f0-a1b3-3fa32fc5945f] Subscription created until 2025-08-15T10:27:24.962577+00:00 [duration:86400000000,validity:86400.000000,patch:43200.000000] (../lib/sbi/nnrf-handler.c:874)
Aug 14 10:27:24 open5gs-vm open5gs-smfd[579]: 08/14 10:27:24.987: [sbi] INFO: [4413a562-78f9-41f0-bef1-937a95f8eda6] (NRF-profile-get) NF registered (../lib/sbi/nf-sm.c:81)
Aug 14 10:27:24 open5gs-vm open5gs-smfd[579]: 08/14 10:27:24.987: [sbi] INFO: Setup NF EndPoint(addr) [127.0.1.250:7777] (../lib/sbi/context.c:2374)
Aug 14 10:27:24 open5gs-vm open5gs-smfd[579]: 08/14 10:27:24.987: [sbi] INFO: [443b1dfe-78f9-41f0-93a9-4bb588306b33] (NRF-profile-get) NF registered (../lib/sbi/nf-sm.c:81)
Aug 14 10:27:24 open5gs-vm open5gs-smfd[579]: 08/14 10:27:24.987: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.12:80] (../lib/sbi/context.c:2374)
Aug 14 10:27:24 open5gs-vm open5gs-smfd[579]: 08/14 10:27:24.988: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../lib/sbi/context.c:2113)
Aug 14 10:27:24 open5gs-vm open5gs-smfd[579]: 08/14 10:27:24.988: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../lib/sbi/context.c:2113)
Aug 14 10:27:24 open5gs-vm open5gs-smfd[579]: 08/14 10:27:24.988: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../lib/sbi/context.c:2113)
Aug 14 10:27:28 open5gs-vm open5gs-smfd[579]: 08/14 10:27:28.101: [sbi] INFO: [46ee6790-78f9-41f0-8baa-ef18aca07d0c] (NRF-notify) NF registered (../lib/sbi/nnrf-handler.c:1140)
Aug 14 10:27:28 open5gs-vm open5gs-smfd[579]: 08/14 10:27:28.101: [sbi] INFO: [46ee6790-78f9-41f0-8baa-ef18aca07d0c] (NRF-notify) NF Profile updated [type:PCF] (../lib/sbi/nnrf-handler.c:1154)
Aug 14 10:27:28 open5gs-vm open5gs-smfd[579]: 08/14 10:27:28.101: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.13:80] (../lib/sbi/context.c:2374)
Aug 14 10:27:28 open5gs-vm open5gs-smfd[579]: 08/14 10:27:28.101: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../lib/sbi/context.c:2113)
Aug 14 10:27:28 open5gs-vm open5gs-smfd[579]: 08/14 10:27:28.473: [diam] INFO: CONNECTED TO 'pcrf.localdomain' (SCTP,soc#11): (../lib/diameter/common/logger.c:81)
Aug 14 10:27:33 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[579]: 08/14 10:27:33.797: [sbi] INFO: [43c561fe-78f9-41f0-9c42-5d885d521359] (NRF-notify) NF registered (../lib/sbi/nnrf-handler.c:1140)
Aug 14 10:27:33 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[579]: 08/14 10:27:33.797: [sbi] INFO: [43c561fe-78f9-41f0-9c42-5d885d521359] (NRF-notify) NF Profile updated [type:AMF] (../lib/sbi/nnrf-handler.c:1154)
Aug 14 10:27:33 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[579]: 08/14 10:27:33.797: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.5:80] (../lib/sbi/context.c:2374)
Aug 14 10:27:33 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[579]: 08/14 10:27:33.797: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.5:7777] (../lib/sbi/context.c:2113)
Aug 14 11:07:51 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[579]: 08/14 11:07:51.619: [sbi] INFO: [43c561fe-78f9-41f0-9c42-5d885d521359] (NRF-notify) NF_DEREGISTERED event [type:AMF] (../lib/sbi/nnrf-handler.c:1172)
Aug 14 11:07:51 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[579]: 08/14 11:07:51.716: [sbi] INFO: [eb882a3e-78fe-41f0-991c-294701c30d5f] (NRF-notify) NF registered (../lib/sbi/nnrf-handler.c:1140)
Aug 14 11:07:51 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[579]: 08/14 11:07:51.717: [sbi] INFO: [eb882a3e-78fe-41f0-991c-294701c30d5f] (NRF-notify) NF Profile updated [type:AMF] (../lib/sbi/nnrf-handler.c:1154)
Aug 14 11:07:51 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[579]: 08/14 11:07:51.717: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.5:80] (../lib/sbi/context.c:2374)
Aug 14 11:07:51 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[579]: 08/14 11:07:51.717: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.5:7777] (../lib/sbi/context.c:2113)
telcomaan@open5gs-vm:~$ sudo cat /etc/open5gs/smf.yaml
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
telcomaan@open5gs-vm:~$ 
```

# Smf-org.yaml

```bash

logger:
  file:
    path: /var/log/open5gs/smf.log
#  level: info   # fatal|error|warn|info(default)|debug|trace

global:
  max:
    ue: 1024  # The number of UE can be increased depending on memory size.
#    peer: 64

smf: 
  sbi:
    server:
      - address: 127.0.0.4
        port: 7777
    client:
#      nrf:
#        - uri: http://127.0.0.10:7777
      scp:
        - uri: http://127.0.0.200:7777
  pfcp:
    server:
      - address: 127.0.0.4
    client:
      upf:
        - address: 127.0.0.7
  gtpc:
    server:
      - address: 127.0.0.4
  gtpu:
    server:
      - address: 127.0.0.4
  metrics:
    server:
      - address: 127.0.0.4
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
#  o S-NSSAI[SST:1 SD:009000] and DNN[internet or ims]
#  info:
#    - s_nssai:
#        - sst: 1
#          sd: 009000
#          dnn:
#            - internet
#            - ims
#
#  o S-NSSAI[SST:1] and DNN[internet] and TAI[PLMN-ID:99970 TAC:1]
#  info:
#    - s_nssai:
#        - sst: 1
#          dnn:
#            - internet
#      tai:
#        - plmn_id:
#            mcc: 999
#            mnc: 70
#          tac: 1
#
#  o If any of conditions below are met:
#   - S-NSSAI[SST:1] and DNN[internet] and TAI[PLMN-ID:99970 TAC:1-9]
#   - S-NSSAI[SST:2 SD:000080] and DNN[internet or ims]
#   - S-NSSAI[SST:4] and DNN[internet] and TAI[PLMN-ID:99970 TAC:10-20,30-40]
#  info:
#    - s_nssai:
#        - sst: 1
#          dnn:
#            - internet
#      tai:
#        - plmn_id:
#            mcc: 999
#            mnc: 70
#          tac:
#            - 1-9
#    - s_nssai:
#        - sst: 2
#          sd: 000080
#          dnn:
#            - internet
#            - ims
#    - s_nssai:
#        - sst: 4
#          dnn:
#            - internet
#      tai:
#        - plmn_id:
#            mcc: 999
#            mnc: 70
#          tac:
#            - 10-20
#            - 30-40
#
#  o Complex Example
#  info:
#    - s_nssai:
#        - sst: 1
#          dnn:
#            - internet
#        - sst: 1
#          sd: 000080
#          dnn:
#            - internet
#            - ims
#        - sst: 1
#          sd: 009000
#          dnn:
#            [internet, ims]
#        - sst: 2
#          dnn:
#            - internet
#        - sst: 3
#          sd: 123456
#          dnn:
#            - internet
#      tai:
#        - plmn_id:
#            mcc: 999
#            mnc: 70
#          tac: [1, 2, 3]
#        - plmn_id:
#            mcc: 999
#            mnc: 70
#          tac: 4
#        - plmn_id:
#            mcc: 999
#            mnc: 70
#          tac:
#            - 5
#            - 6
#        - plmn_id:
#            mcc: 999
#            mnc: 70
#          tac:
#            - 100-200
#            - 300-400
#        - plmn_id:
#            mcc: 999
#            mnc: 70
#          tac:
#            - 500-600
#            - 700-800
#            - 900-1000
#    - s_nssai:
#        - sst: 4
#          dnn:
#            - internet
#      tai:
#        - plmn_id:
#            mcc: 999
#            mnc: 70
#          tac: 99
#
################################################################################
# SBI Server
################################################################################
#  o Bind to the address on the eth0 and advertise as open5gs-smf.svc.local
#  sbi:
#    server:
#      - dev:eth0
#        advertise: open5gs-smf.svc.local
#
#  o Specify a custom port number 7777 while binding to the given address
#  sbi:
#    server:
#      - address: smf.localdomain
#        port: 7777
#
#  o Bind to 127.0.0.4 and advertise as open5gs-smf.svc.local
#  sbi:
#    server:
#      - address: 127.0.0.4
#        port: 7777
#        advertise: open5gs-smf.svc.local
#
#  o Bind to port 7777 but advertise with a different port number 8888
#  sbi:
#    server:
#      - address: 127.0.0.4
#        port: 7777
#        advertise: open5gs-smf.svc.local:8888
#
################################################################################
# SBI Client
################################################################################
#  o Direct Communication with NRF
#  sbi:
#    client:
#      nrf:
#        - uri: http://127.0.0.10:7777
#
#  o Indirect Communication by Delegating to SCP
#  sbi:
#    client:
#      scp:
#        - uri: http://127.0.0.200:7777
#
#  o Indirect Communication without Delegation
#  sbi:
#    client:
#      nrf:
#        - uri: http://127.0.0.10:7777
#      scp:
#        - uri: http://127.0.0.200:7777
#      delegated:
#        nrf:
#          nfm: no    # Directly communicate NRF management functions
#          disc: no   # Directly communicate NRF discovery
#        scp:
#          next: no   # Do not delegate to SCP for next-hop
#
#  o Indirect Communication with Delegated Discovery
#  sbi:
#    client:
#      nrf:
#        - uri: http://127.0.0.10:7777
#      scp:
#        - uri: http://127.0.0.200:7777
#      delegated:
#        nrf:
#          nfm: no    # Directly communicate NRF management functions
#          disc: yes  # Delegate discovery to SCP
#        scp:
#          next: yes  # Delegate to SCP for next-hop communications
#
#  o Default delegation: all communications are delegated to the SCP
#  sbi:
#    client:
#      nrf:
#        - uri: http://127.0.0.10:7777
#      scp:
#        - uri: http://127.0.0.200:7777
#      # No 'delegated' section; defaults to AUTO delegation
#
################################################################################
# HTTPS scheme with TLS
################################################################################
#  o Set as default if not individually set
#  default:
#    tls:
#      server:
#        scheme: https
#        private_key: /etc/open5gs/tls/smf.key
#        cert: /etc/open5gs/tls/smf.crt
#      client:
#        scheme: https
#        cacert: /etc/open5gs/tls/ca.crt
#  sbi:
#    server:
#      - address: smf.localdomain
#    client:
#      nrf:
#        - uri: https://nrf.localdomain
#
#  o Enable SSL key logging for Wireshark
#    - This configuration allows capturing SSL/TLS session keys
#      for debugging or analysis purposes using Wireshark.
#  default:
#    tls:
#      server:
#        scheme: https
#        private_key: /etc/open5gs/tls/smf.key
#        cert: /etc/open5gs/tls/smf.crt
#        sslkeylogfile: /var/log/open5gs/tls/smf-server-sslkeylog.log
#      client:
#        scheme: https
#        cacert: /etc/open5gs/tls/ca.crt
#        client_sslkeylogfile: /var/log/open5gs/tls/smf-client-sslkeylog.log
#  sbi:
#    server:
#      - address: smf.localdomain
#    client:
#      nrf:
#        - uri: https://nrf.localdomain
#
#  o Add client TLS verification
#  default:
#    tls:
#      server:
#        scheme: https
#        private_key: /etc/open5gs/tls/smf.key
#        cert: /etc/open5gs/tls/smf.crt
#        verify_client: true
#        verify_client_cacert: /etc/open5gs/tls/ca.crt
#      client:
#        scheme: https
#        cacert: /etc/open5gs/tls/ca.crt
#        client_private_key: /etc/open5gs/tls/smf.key
#        client_cert: /etc/open5gs/tls/smf.crt
#  sbi:
#    server:
#      - address: smf.localdomain
#    client:
#      nrf:
#        - uri: https://nrf.localdomain
#
################################################################################
# PFCP Server
################################################################################
#  o Override PFCP address to be advertised to UPF in PFCP association
#  pfcp:
#    server:
#      - dev: eth0
#        advertise: open5gs-smf.svc.local
#
################################################################################
# PFCP Client
################################################################################
#  o UPF selection by eNodeB TAC
#   (either single TAC or multiple TACs, DECIMAL representation)
#  pfcp:
#    client:
#      upf:
#        - address: 127.0.0.7
#          tac: 1
#        - address: 127.0.0.12
#          tac: [3,5,8]
#
#  o UPF selection by UE's DNN/APN (either single DNN/APN or multiple DNNs/APNs)
#  pfcp:
#    client:
#      upf:
#        - address: 127.0.0.7
#          dnn: ims
#        - address: 127.0.0.12
#          dnn: [internet, web]
#
#  o UPF selection by CellID(e_cell_id: 28bit, nr_cell_id: 36bit)
#    (either single enb_id or multiple enb_ids, HEX representation)
#  pfcp:
#    client:
#      upf:
#        - address: 127.0.0.7
#          e_cell_id: 463
#        - address: 127.0.0.12
#          nr_cell_id: [123456789, 9413]
#
################################################################################
# GTP-C Server
################################################################################
#  o Listen on IPv4 and IPv6
#  gtpc:
#    server:
#      - address: 127.0.0.4
#      - address: fd69:f21d:873c:fa::3
#
################################################################################
# GTP-U Server
################################################################################
#  o Listen on IPv4 and IPv6
#  gtpu:
#    server:
#      - address: 127.0.0.4
#      - address: ::1
#
################################################################################
# 3GPP Specification
################################################################################
#  o Specific DNN/APN(e.g 'ims') uses 10.46.0.1/16, 2001:db8:babe::1/48
#   (If the UE has unknown DNN/APN(not internet/ims), SMF/UPF will crash.)
#  session:
#    - subnet: 10.45.0.0/16
#      gateway: 10.45.0.1
#      dnn: internet
#    - subnet: 2001:db8:cafe::/48
#      dnn: internet
#    - subnet: 10.46.0.0/16
#      gateway: 10.46.0.1
#      dnn: ims
#    - subnet: 2001:db8:babe::/48
#      dnn: ims
#
#  o Pool Range
#  session:
#    - subnet: 10.45.0.0/16
#      gateway: 10.45.0.1
#      range:
#        - 10.45.0.100-10.45.0.200
#        - 10.45.1.100-
#        - -10.45.0.200
#    - subnet: 2001:db8:cafe::/48
#      range:
#        - 2001:db8:cafe:a0::0-2001:db8:cafe:b0::0
#        - 2001:db8:cafe:c0::0-2001:db8:cafe:d0::0
#
#  o Security Indication(5G Core only)
#  security_indication:
#    integrity_protection_indication: required|preferred|not-needed
#    confidentiality_protection_indication: required|preferred|not-needed
#    maximum_integrity_protected_data_rate_uplink: bitrate64kbs|maximum-UE-rate
#    maximum_integrity_protected_data_rate_downlink: bitrate64kbs|maximum-UE-rate
```

# smf-multipf.yaml

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
      - address: 127.0.0.4
        port: 7777
    client:
#      nrf:
#        - uri: http://127.0.0.10:7777
      scp:
        - uri: http://127.0.0.200:7777
  pfcp:
    server:
      # MODIFIED: Changed to the VM's static IP
      - address: 127.0.0.4
    # THIS IS THE LEGACY SECTION YOU REQUESTED
    client:
      upf:
        # Entry for the first UPF, selected by TAC 1
        - address: 127.0.0.5
          tac: 1
        
        # Entry for the second UPF, selected by TAC 2
        - address: 127.0.0.12
          tac: 2

  # The modern smf.upf.pool section has been REMOVED

  gtpc:
    server:
      # MODIFIED: Changed to the VM's static IP
      - address: 127.0.0.4
  gtpu:
    server:
      # MODIFIED: Changed to the VM's static IP
      - address: 127.0.0.4
  metrics:
    server:
      # MODIFIED: Changed to the VM's static IP
      - address: 127.0.0.4
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
```