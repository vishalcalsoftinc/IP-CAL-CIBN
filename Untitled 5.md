Following the migration from a Google Cloud Platform (GCP) environment to an ESXi-based virtualization setup, this guide provides a comprehensive walkthrough for deploying a 5G Standalone (SA) network. The architecture utilizes OpenAirInterface (OAI) for the Radio Access Network (RAN) and Open5GS for the 5G Core (5GC).

This setup implements a CU/DU split architecture for the OAI gNB, with a distinct CU-CP (Control Plane), CU-UP (User Plane), and two DUs (Distributed Units). The primary objective is to configure and execute a successful F1 handover between the two DUs, with a simulated OAI UE.

### **Part 1: ESXi Virtual Machine Environment**

The first step is to provision the necessary Virtual Machines on your ESXi host. All VMs must be connected to the same virtual switch to ensure network connectivity. Assign static IP addresses as detailed in the table below.

| Role | VM Name | Static IP Address |
| :--- | :--- | :--- |
| OAI NR-UE (RF Server) | `oai-nr-ue-vm1` | `172.17.42.81` |
| OAI Source DU (DU0) | `oai-du-vm2` | `172.17.42.82` |
| OAI CU-CP | `oai-cucp-vm3` | `172.17.42.83` |
| OAI Target DU (DU1) | `oai-du2-vm4` | `172.17.42.84` |
| OAI CU-UP | `oai-cuup-vm5` | `172.17.42.85` |
| Open5GS Core | `open5gs-vm6` | `172.17.42.86` |

---

### **Part 2: Open5GS Core Network Installation & Configuration**

These steps are to be performed on the `open5gs-vm6` VM.

#### **2.1. Install Prerequisites**
Update your system and install necessary packages.
```bash
sudo apt update
sudo apt install nano iptables git
sudo apt install -y software-properties-common curl gnupg git
```

#### **2.2. Install MongoDB**
Open5GS uses MongoDB as its database.
```bash
curl -fsSL https://pgp.mongodb.com/server-8.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg --dearmor
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg] https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
sudo apt update
sudo apt install -y mongodb-org
sudo systemctl start mongod
sudo systemctl enable mongod
```

#### **2.3. Install Open5GS**
Add the official Open5GS repository and install the packages.
```bash
sudo add-apt-repository ppa:open5gs/latest
sudo apt update
sudo apt install open5gs -y
```

#### **2.4. Configure Open5GS Components**
You must update the configuration files to use the static IP of the `open5gs-vm6`.

*   **In `/etc/open5gs/amf.yaml`**, set the NGAP server address:
    ```yaml
    #...
    amf:
      sbi: #...
      ngap:
        server:
          - address: 172.17.42.86 # open5gs vm ip
    #...
    ```
    Then, restart the AMF service:
    ```bash
    sudo systemctl restart open5gs-amfd
    ```

*   **In `/etc/open5gs/upf.yaml`**, set the GTP-U server address:
    ```yaml
    #...
    upf:
      pfcp: #...
      gtpu:
        server:
          - address: 172.17.42.86 # open5gs vm ip
    #...
    ```
    Then, restart the UPF service:
    ```bash
    sudo systemctl restart open5gs-upfd
    ```

#### **2.5. Enable Network Address Translation (NAT)**
To provide the UE with internet connectivity through the core network, enable IP forwarding and set up a masquerading rule. **Note:** Ensure your network interface name is correct (e.g., `ens34`).
```bash
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -o ens34 -j MASQUERADE
sudo iptables -I FORWARD 1 -j ACCEPT
```

#### **2.6. Setup and Add Subscriber in WebUI**
Install the Open5GS WebUI to manage subscribers.

*   **Install Node.js:**
    ```bash
    sudo apt update
    curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
    sudo apt install nodejs -y
    ```

*   **Clone and prepare the WebUI:**
    ```bash
    cd ~
    git clone https://github.com/open5gs/open5gs
    cd open5gs/webui
    npm install
    ```

*   **Configure the WebUI Host IP:**
    To make the WebUI accessible from your network, edit its server configuration.
    ```bash
    sudo nano server/index.js
    ```
    Find the line:
    `const _hostname = process.env.HOSTNAME || 'localhost';`
    And change it to:
    `const _hostname = process.env.HOSTNAME || '172.17.42.86';`

*   **Start and Access the WebUI:**
    Run the WebUI in the foreground.
    ```bash
    npm run dev
    ```
    You can now access the interface from a browser on your local machine at `http://172.17.42.86:3000`. Log in with the default credentials (`admin` / `1423`) and add a new subscriber with the following details:
    *   **IMSI:** `999700000000001`
    *   **Subscriber Key:** `465B5CE8B199B49FAA5F0A2EE238A6BC`
    *   **USIM Type:** OPc
    *   **Operator Key:** `E8ED289DEBA952E4283B54E88E6183CA`

---

### **Part 3: OAI RAN Installation & Building**

Perform these steps on **all five OAI VMs** (`oai-nr-ue-vm1`, `oai-du-vm2`, `oai-cucp-vm3`, `oai-du2-vm4`, `oai-cuup-vm5`). This process compiles the OAI components with the necessary telnet server for triggering the handover.

```bash
# Run on all five OAI VMs
sudo apt update
sudo apt install git cmake ninja-build build-essential

cd ~
git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git
cd openairinterface5g
source oaienv
cd cmake_targets

# This command builds all necessary components with telnet support
./build_oai --ninja --nrUE --gNB --build-lib telnetsrv
```

---

### **Part 4: OAI RAN Configuration**

Create the following configuration files in the `/etc/oai/` directory on their respective VMs.

#### **4.1. On `oai-cucp-vm3` (CU-CP)**
Create `/etc/oai/oai-cucp.conf`:

```conf
Active_gNBs = ( "oai-cu-cp");
Asn1_verbosity = "info";

gNBs =
(
 {
    gNB_ID = 0xe00;
    gNB_name  =  "oai-cu-cp";
    tracking_area_code  =  1;
    plmn_list = ({ mcc = 999; mnc = 70; mnc_length = 2;
                   snssaiList = ({ sst = 1, sd = 0x000001 },
                                 { sst = 1, sd = 0xFFFFFF },
                                 { sst = 2, sd = 0x111111 }) });
    nr_cellid = 12345678L;
    tr_s_preference = "f1";
    local_s_address = "172.17.42.83"; # CU-CP IP
    remote_s_address = "0.0.0.0";     # Accept DUs from multiple IPs
    local_s_portc   = 501;
    local_s_portd   = 2153;
    remote_s_portc  = 500;
    remote_s_portd  = 2153;

    SCTP : {
        SCTP_INSTREAMS  = 5;
        SCTP_OUTSTREAMS = 5;
    };

    amf_ip_address = ({ ipv4 = "172.17.42.86"; });

    E1_INTERFACE = (
      {
        type = "cp";
        ipv4_cucp = "172.17.42.83"; # CU-CP IP
        port_cucp = 38462;
        ipv4_cuup = "172.17.42.85"; # CU-UP IP
        port_cuup = 38462;
      }
    );

    NETWORK_INTERFACES : {
        GNB_IPV4_ADDRESS_FOR_NG_AMF = "172.17.42.83"; # CU-CP to AMF
    };
  }
);

security = {
  ciphering_algorithms = ( "nea0" );
  integrity_algorithms = ( "nia2", "nia0" );
  drb_ciphering = "yes";
  drb_integrity = "no";
};

log_config : {
   global_log_level = "info";
   rrc_log_level    = "info";
   f1ap_log_level   = "info";
   ngap_log_level   = "debug";
};
```

#### **4.2. On `oai-cuup-vm5` (CU-UP)**
Create `/etc/oai/oai-cuup.conf`:

```conf
Active_gNBs = ( "oai-cu-cp");
Asn1_verbosity = "none";

gNBs =
(
 {
    gNB_ID = 0xe00;
    gNB_CU_UP_ID = 0xe00;
    gNB_name  =  "oai-cuup-sd1";
    tracking_area_code  =  1;
    plmn_list = ({ mcc = 999; mnc = 70; mnc_length = 2; snssaiList = ({ sst = 1, sd = 0x000001 },{ sst = 1, sd = 0xFFFFFF }) });
    tr_s_preference = "f1";
    local_s_address = "172.17.42.85";  #cu-up vm IP
    remote_s_address = "0.0.0.0";
    local_s_portc   = 501;
    local_s_portd   = 2153;
    remote_s_portc  = 500;
    remote_s_portd  = 2153;

    SCTP : {
        SCTP_INSTREAMS  = 5;
        SCTP_OUTSTREAMS = 5;
    };

    E1_INTERFACE = (
      {
        type = "cp";
        ipv4_cucp = "172.17.42.83"; # CU-CP IP
        port_cucp = 38462;
        ipv4_cuup = "172.17.42.85"; # CU-UP IP
        port_cuup = 38462;
      }
    );

    NETWORK_INTERFACES : {
        GNB_IPV4_ADDRESS_FOR_NGU = "172.17.42.85"; # CU-UP IP for N3 interface
        GNB_PORT_FOR_S1U         = 2152;
    };
  }
);

log_config : {
  global_log_level = "info";
  pdcp_log_level   = "info";
  f1ap_log_level   = "info";
  ngap_log_level   = "info";
};
```

#### **4.3. On `oai-du-vm2` (Source DU)**
Create `/etc/oai/oai-du.conf`:

```conf
Active_gNBs = ( "oai-cu-cp");
Asn1_verbosity = "info";

gNBs = ({
    gNB_ID = 0xe00;
    gNB_DU_ID = 0xe01;
    gNB_name  =  "oai-cu-cp";
    tracking_area_code  =  1;
    plmn_list = ({ mcc = 999; mnc = 70; mnc_length = 2; snssaiList =  ({ sst = 1, sd = 0x000001 },{ sst = 1, sd = 0xFFFFFF }) });
    nr_cellid = 12345678L;
    min_rxtxtime = 6;
    servingCellConfigCommon = ({
      physCellId = 1; // PCI for Source DU
      // -- other physical cell parameters --
    });
    SCTP : {
        SCTP_INSTREAMS  = 2;
        SCTP_OUTSTREAMS = 2;
    };
});

MACRLCs = ({
    num_cc           = 1;
    tr_s_preference  = "local_L1";
    tr_n_preference  = "f1";
    local_n_address = "172.17.42.82";  // This DU's IP
    remote_n_address = "172.17.42.83"; // CU-CP's IP
    local_n_portc   = 500;
    local_n_portd   = 2153;
    remote_n_portc  = 501;
    remote_n_portd  = 2153;
});

// -- L1s, RUs, and channelmod sections remain the same --

rfsimulator: {
    serveraddr = "172.17.42.81"; // UE VM IP
    serverport = 4043;
    modelname = "AWGN";
}

log_config: {
  global_log_level = "info";
  f1ap_log_level = "info";
};
```

#### **4.4. On `oai-du2-vm4` (Target DU)**
Create `/etc/oai/oai-du2.conf`:

```conf
Active_gNBs = ( "oai-cu-cp");
Asn1_verbosity = "info";

gNBs = ({
    gNB_ID = 0xe00;
    gNB_DU_ID = 0xe02;
    gNB_name  =  "oai-cu-cp";
    tracking_area_code  =  2;
    plmn_list = ({ mcc = 999; mnc = 70; mnc_length = 2; snssaiList =  ({ sst = 1, sd = 0x000001 },{ sst = 1, sd = 0xFFFFFF }) });
    nr_cellid = 12345679L;
    min_rxtxtime = 6;
    servingCellConfigCommon = ({
      physCellId = 2; // PCI for Target DU
      // -- other physical cell parameters --
    });
    SCTP : {
        SCTP_INSTREAMS  = 2;
        SCTP_OUTSTREAMS = 2;
    };
});

MACRLCs = ({
    num_cc           = 1;
    tr_s_preference  = "local_L1";
    tr_n_preference  = "f1";
    local_n_address = "172.17.42.84";  // This DU's IP
    remote_n_address = "172.17.42.83"; // CU-CP's IP
    local_n_portc   = 500;
    local_n_portd   = 2153;
    remote_n_portc  = 501;
    remote_n_portd  = 2153;
});

// -- L1s, RUs, and channelmod sections remain the same --

rfsimulator: {
    serveraddr = "172.17.42.81"; // UE VM IP
    serverport = 4043;
    modelname = "AWGN";
}

log_config: {
  global_log_level = "info";
  f1ap_log_level = "info";
};
```

#### **4.5. On `oai-nr-ue-vm1` (UE)**
Create `/etc/oai/nr-ue.conf`:

```conf
uicc0 = {
  imsi = "999700000000001";
  key = "465B5CE8B199B49FAA5F0A2EE238A6BC";
  opc = "E8ED289DEBA952E4283B54E88E6183CA";
  dnn = "internet";
  nssai_sst = 1;
  nssai_sd = 0xFFFFFF;
}

rfsimulator: {
    serveraddr = "server"; // Configures the UE to act as the RF server
    serverport = 4043;
    modelname = "AWGN";
}
```

#### **4.6. The RF Simulator: Why the UE is the Server**
The F1 handover scenario requires the UE to listen to the source DU and then switch to the target DU. The OAI RF simulator operates on a one-server-to-many-clients model. If the DUs were servers, the UE client could only connect to one at a time, making it impossible to hear the target DU for handover.

By inverting this model, the UE becomes the central RF simulation server. Both DUs act as clients, feeding their signals into this single, shared simulation environment. This allows the UE to seamlessly switch its attention from the source DU's signal to the target DU's signal within the same environment, which is the key to a successful simulated handover.

---

### **Part 5: Starting the Network & Executing Handover**

Follow this startup sequence precisely, using a separate terminal for each command.

1.  **Start Open5GS Services (on `open5gs-vm6`)**
    ```bash
    sudo systemctl start mongod
    sudo systemctl start open5gs-amfd
    sudo systemctl start open5gs-ausfd
    sudo systemctl start open5gs-bsfd
    sudo systemctl start open5gs-mmed
    sudo systemctl start open5gs-nrf
    sudo systemctl start open5gs-nssfd
    sudo systemctl start open5gs-pcfd
    sudo systemctl start open5gs-sgwcd
    sudo systemctl start open5gs-smfd
    sudo systemctl start open5gs-udmd
    sudo systemctl start open5gs-udrd
    sudo systemctl start open5gs-upfd
    # Verify all services
    sudo systemctl status open5gs-*
    ```

2.  **Enable NAT (on `open5gs-vm6`)**
    ```bash
    sudo sysctl -w net.ipv4.ip_forward=1
    sudo iptables -t nat -A POSTROUTING -o ens34 -j MASQUERADE
    sudo iptables -I FORWARD 1 -j ACCEPT
    ```

3.  **Start the CU-CP (on `oai-cucp-vm3`)**
    ```bash
    cd ~/openairinterface5g/cmake_targets/ran_build/build
    sudo -E ./nr-softmodem -O /etc/oai/oai-cucp.conf --sa --telnetsrv --telnetsrv.shrmod ci
    ```

4.  **Start the CU-UP (on `oai-cuup-vm5`)**
    ```bash
    cd ~/openairinterface5g/cmake_targets/ran_build/build
    sudo -E ./nr-cuup -O /etc/oai/oai-cuup.conf --telnetsrv --telnetsrv.shrmod ci
    ```

5.  **Start the NR UE as RF Server (on `oai-nr-ue-vm1`)**
    ```bash
    cd ~/openairinterface5g/cmake_targets/ran_build/build
    sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --ssb 516 --rfsim -O /etc/oai/nr-ue.conf
    ```

6.  **Start the Source DU (on `oai-du-vm2`)**
    The DU connects to the UE's RF server, and the UE attaches to the network.
    ```bash
    cd ~/openairinterface5g/cmake_targets/ran_build/build
    sudo ./nr-softmodem -O /etc/oai/oai-du.conf --rfsim --sa
    ```

7.  **Start the Target DU (on `oai-du2-vm4`)**
    This DU also connects to the UE. The CU-CP log will show a second F1 setup request.
    ```bash
    cd ~/openairinterface5g/cmake_targets/ran_build/build
    sudo ./nr-softmodem -O /etc/oai/oai-du2.conf --rfsim --sa
    ```

8.  **Trigger the F1 Handover**
    From any machine on the network, run the following command to connect to the CU-CP's telnet server and issue the handover trigger.
    ```bash
    echo ci trigger_f1_ho | nc 172.17.42.83 9090 && echo
    ```

---

### **Part 6: Verifying the Handover**

1.  **CU-CP Logs (`oai-cucp-vm3`):** You will see the "Handover triggered" message, followed by logs indicating the RRC reconfiguration is sent and later that the handover is complete, including a context release for the source DU.

2.  **UE Logs (`oai-nr-ue-vm1`):** The UE log is crucial. It will show the reception of an `RRCReconfiguration` message containing `reconfigurationWithSync`. This is followed by logs showing a successful physical layer synchronization to the new cell (`PCI: 2`) and the transmission of `RRCReconfigurationComplete`.

3.  **Source DU Logs (`oai-du-vm2`):** This log will show a command to release the UE context after the handover is complete.

4.  **Target DU Logs (`oai-du2-vm4`):** This log will show the successful contention-free random access (CFRA) procedure from the UE as it arrives at the new cell.

5.  **Test Data Connectivity:** The ultimate validation is a continuous ping. Before, during, and after triggering the handover, run a ping from the UE VM. The replies should continue without significant interruption, confirming a seamless data plane switch.
    ```bash
    # On oai-nr-ue-vm1
    ping -I oaitun_ue1 8.8.8.8
    ```

---
Of course. Given the migration to a split architecture with a distinct CU-CP (Control Plane) and CU-UP (User Plane), the F1-Handover procedure involves more specific interactions. Here is a rewritten list of the handover steps, adapted from the original diagram to reflect the new CU-split model.

***

### **F1-Handover Procedure with CU-CP/CU-UP Split**

This procedure outlines the sequence of events for a UE handover between two DUs (DU0 and DU1) managed by a disaggregated Central Unit (CU-CP and CU-UP).

#### **Phase 1: Handover Preparation**

1.  **Initial State: UE Connected to DU0**
    *   The UE is initially connected to the network and has an active data session. The user plane path is UE → DU0 → CU-UP → 5GC.

2.  **Measurement Report**
    *   The UE sends a Measurement Report to the source DU (DU0), indicating that it is receiving a stronger signal from a neighboring cell (managed by DU1).

3.  **Measurement Forwarding**
    *   DU0 forwards this measurement information to the **CU-CP** over the F1-C interface. The **CU-CP** analyzes the report and makes the decision to initiate a handover.

4.  **Handover Request (Setup Request)**
    *   The **CU-CP** sends a HANDOVER REQUEST message over the F1-C interface to the target DU (DU1). This message instructs DU1 to prepare the necessary radio resources for the incoming UE.

5.  **Handover Request Acknowledge (Setup Response)**
    *   DU1 allocates the requested resources, prepares for the UE's arrival, and sends a HANDOVER REQUEST ACKNOWLEDGE message back to the **CU-CP**, confirming it's ready.

#### **Phase 2: Handover Execution**

6.  **Switch Command to Source DU**
    *   The **CU-CP** sends a command to the source DU (DU0) that includes the RRC Reconfiguration message to be forwarded to the UE.

7.  **RRC Reconfiguration**
    *   The source DU (DU0) sends the `RRCReconfiguration` message to the UE. This message contains the target cell information and instructs the UE to switch to DU1.

8.  **RACH to Target DU**
    *   The UE detaches from DU0 and initiates the Random Access Channel (RACH) procedure to connect with the target DU (DU1).

9.  **RACH Response**
    *   The target DU (DU1) responds to the UE, confirming the connection and completing the synchronization process.

#### **Phase 3: Handover Completion**

10. **Handover Complete Confirmation**
    *   The UE sends an `RRCReconfigurationComplete` message to the target DU (DU1). DU1 forwards this confirmation to the **CU-CP**, signaling that the UE has successfully switched to the new cell.

11. **CU Path Switch (CU-CP and CU-UP Interaction)**
    *   This is the critical step for switching the data plane:
        *   The **CU-CP**, upon receiving the handover completion, sends an **E1 Bearer Context Modification Request** to the **CU-UP**.
        *   This request instructs the **CU-UP** to change the data path for the UE's session. The **CU-UP** tears down the old GTP-U tunnel to DU0 and establishes a new GTP-U tunnel to the target DU (DU1).
        *   The **CU-CP** may also send a PATH SWITCH REQUEST message to the 5GC (AMF) to update the core network about the new path, although this is transparent to the DUs.

12. **Resource Release**
    *   The **CU-CP** sends a UE CONTEXT RELEASE command to the source DU (DU0), instructing it to release all resources that were previously allocated for the UE.

13. **Release Confirmation (Release OK)**
    *   The source DU (DU0) removes the UE's context and sends a UE CONTEXT RELEASE COMPLETE message back to the **CU-CP**.

14. **Final State: UE Connected to DU1**
    *   The handover is complete. The UE is now fully connected to the target DU (DU1), and the user data flows along the new path: UE → DU1 → **CU-UP** → 5GC.