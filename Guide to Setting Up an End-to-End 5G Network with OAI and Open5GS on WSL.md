
--

---
This document provides a comprehensive guide to deploying a complete 5G Standalone (SA) network, including OpenAirInterface (OAI) for the Radio Access Network (RAN) and Open5GS for the 5G Core, on a single WSL Ubuntu 20.04 instance.

This guide adapts the original multi-VM documentation for a unified setup on a single machine with the IP address `172.28.251.212`. All components will be configured to communicate with each other on this single IP.

### **1. Overview of the Architecture**

In this single-machine setup, all 5G components—OAI NR-UE (User Equipment), DU (Distributed Unit), CU-CP (Central Unit - Control Plane), CU-UP (Central Unit - User Plane), and the Open5GS Core Network—will run on your WSL instance.

*   **Open5GS (5GC):** Provides the core network functions, including AMF, SMF, and UPF. It will handle UE authentication, session management, and routing user data to the internet.
*   **OAI RAN (gNB):** Acts as the 5G base station, disaggregated into:
    *   **CU-CP:** Manages the control signaling (RRC) between the UE and the network.
    *   **CU-UP:** Handles the user data traffic, forwarding it between the DU and the 5G Core's UPF.
    *   **DU:** Manages the lower layers of the radio stack (PHY, MAC).
*   **OAI NR-UE:** A simulated UE that connects to the network, authenticates, and exchanges data.
*   **RF Simulator:** Since no physical radio hardware is used, OAI's RF simulator will create a virtual link between the OAI DU and the OAI NR-UE.

All inter-component communication (F1, E1, N2, N3 interfaces) will occur on your WSL IP `172.28.251.212`.

### **2. Installation of Components**

#### **2.1. Install Open5GS Core Network**

First, install the Open5GS 5G Core components and the WebUI to manage subscribers.

```bash
# Add Open5GS PPA repository
sudo add-apt-repository ppa:open5gs/latest
sudo apt update

# Install Open5GS components
sudo apt install open5gs

# Install Node.js for the WebUI
sudo apt install -y curl
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# Clone the Open5GS WebUI
git clone https://github.com/open5gs/open5gs.git
```


#### **2.2. Install OAI (gNB and nrUE)**

The OAI installation involves cloning the source code, patching it to fix known build issues, and then compiling the executables.

**Step 1: Install Dependencies and Clone OAI**

```bash
# Install build dependencies
sudo apt install -y git cmake build-essential

# Clone the OAI repository
git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git oai
```

**Step 2: Patch OAI Source Code for AVX Compatibility**

The OAI source code contains invalid AVX-512/AVX2 intrinsics that will cause the build to fail. You must patch the two affected files before compiling.

```bash
# Navigate to the OAI directory
cd oai

# Patch common/utils/nr/nr_common.c
sed -i 's/_mm512_loadu_epi8(&in\[i\])/_mm512_loadu_si512((__m512i const \*)in+i)/g' common/utils/nr/nr_common.c
sed -i 's/_mm512_storeu_epi8(&out\[i\], reversed)/_mm512_storeu_si512((__m512i \*)out+i, reversed)/g' common/utils/nr/nr_common.c

# Patch openair1/PHY/NR_TRANSPORT/nr_scrambling.c
sed -i 's/_mm256_load_epi32(&((\(uint32_t \*\)in)\[i_32\]))/_mm256_loadu_si256((__m256i const *)\&((\(uint32_t *\)in)\[i_32\]))/g' openair1/PHY/NR_TRANSPORT/nr_scrambling.c
sed -i 's/_mm256_load_epi32(&seq\[i_32\])/_mm256_loadu_si256((__m256i const *)\&seq\[i_32\])/g' openair1/PHY/NR_TRANSPORT/nr_scrambling.c
sed -i 's/_mm256_storeu_epi32(&out\[i_32\]/_mm256_storeu_si256((__m256i *)\&out\[i_32\]/g' openair1/PHY/NR_TRANSPORT/nr_scrambling.c

echo "OAI source code patched successfully."
```

**Step 3: Build OAI Components**

With the patches applied, build the OAI components in stages.

```bash
# Navigate to the build script directory
source oaienv
cd cmake_targets

# Install dependencies for OAI build
./build_oai -I

# Remove the conflicting package
sudo apt remove -y libyaml-cpp-dev && sudo apt autoremove -y

# Build the gNB (CU/DU) components
./build_oai -w SIMU --gNB --ninja

# Build the nrUE component
./build_oai -w SIMU --nrUE --ninja

# Build the E2 agent (optional but recommended)
./build_oai --build-e2 --ninja
```

### **3. Network and Core Configuration**

#### **3.1. Configure Host Networking and Firewall**

To allow the 5G core to route traffic from the UE to the internet, you need to enable IP forwarding and set up a NAT rule.

**Note:** The primary network interface in WSL is typically `eth0`. Verify this with the `ip a` command and replace `eth0` if necessary.

```bash
# 1. Enable IP Forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# 2. Add a NAT Masquerading Rule
# This allows the UE subnet (10.45.0.0/16) to access the internet via your WSL's eth0 interface.
sudo iptables -t nat -A POSTROUTING -s 10.45.0.0/16 -o eth0 -j MASQUERADE

# 3. Stop Uncomplicated Firewall (UFW)
# UFW can interfere with the NAT and forwarding rules.
sudo systemctl stop ufw
sudo systemctl disable ufw

# 4. Allow All Forwarded Packets
sudo iptables -I FORWARD 1 -j ACCEPT
```


#### **3.2. Configure Open5GS Components**

You must update the Open5GS configuration files to use your WSL IP address `172.28.251.212` for interfaces connecting to the RAN.
##### **File: `/etc/open5gs/amf.yaml` (Full File)**

This configuration file for the Access and Mobility Management Function (AMF) has been updated to set the NGAP interface to the specified WSL IP address and to define the supported network slices.

```yaml
logger:
  file:
    path: /var/log/open5gs/amf.log
#  level: info  # Options: fatal|error|warn|info(default)|debug|trace

global:
  max:
    ue: 1024  # Maximum number of User Equipments (UEs)
#   peer: 64

amf:
  sbi:
    server:
      - address: 127.0.0.5
        port: 7777
    client:
#     nrf:
#       - uri: http://127.0.0.10:7777
      scp:
        - uri: http://127.0.0.200:7777
  ngap:
    server:
      - address: 172.28.251.212  # Your WSL IP for the NGAP interface (gNB connection)
  metrics:
    server:
      - address: 127.0.0.5
        port: 9090
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
      s_nssai: # Supported Network Slice Selection Assistance Information
        - sst: 1
        - sst: 1
          sd: 111111
        - sst: 1
          sd: FFFFFF
        - sst: 2
          sd: 111111
        - sst: 3
          sd: 111111
        - sst: 4
          sd: 111111
  security:
    integrity_order : [ NIA2, NIA1, NIA0 ]
    ciphering_order : [ NEA0, NEA1, NEA2 ]
  network_name:
    full: Open5GS
    short: Next
  amf_name: open5gs-amf0
  time:
#   t3502:
#     value: 720  # 12 minutes * 60 = 720 seconds
    t3512:
      value: 540    # 9 minutes * 60 = 540 seconds
```

##### **File: `/etc/open5gs/upf.yaml` (Full File)**

The User Plane Function (UPF) configuration has been updated to use the WSL IP address for the GTP-U interface. Additionally, a DNN named `internet` is now associated with the `ogstun` device.

```yaml
logger:
  file:
    path: /var/log/open5gs/upf.log
#  level: info  # Options: fatal|error|warn|info(default)|debug|trace

global:
  max:
    ue: 1024  # Maximum number of User Equipments (UEs)
#   peer: 64

upf:
  pfcp:
    server:
      - address: 127.0.0.7
    client:
#     smf:      # UPF PFCP Client attempts to associate with SMF PFCP Server
#       - address: 127.0.0.4
  gtpu:
    server:
      - address: 172.28.251.212 # Your WSL IP for the GTP-U interface
  session:
    - subnet: 10.45.0.0/16
      gateway: 10.45.0.1
      dnn: internet # Data Network Name for this subnet
      dev: ogstun   # Interface for this user plane traffic
    - subnet: 2001:db8:cafe::/48
      gateway: 2001:db8:cafe::1
  metrics:
    server:
      - address: 127.0.0.7
        port: 9090
```

##### **File: `/etc/open5gs/smf.yaml` (Full File)**

The Session Management Function (SMF) configuration has been updated to define the network slices it serves, ensuring alignment with the AMF's supported slices.

```yaml
logger:
  file:
    path: /var/log/open5gs/smf.log
#  level: info  # Options: fatal|error|warn|info(default)|debug|trace

global:
  max:
    ue: 1024  # Maximum number of User Equipments (UEs)
#   peer: 64

smf:
  sbi:
    server:
      - address: 127.0.0.4
        port: 7777
    client:
#     nrf:
#       - uri: http://127.0.0.10:7777
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
# p-cscf:
#   - 127.0.0.1
#   - ::1
# ctf:
#   enabled: auto   # auto(default)|yes|no
  freeDiameter: /etc/freeDiameter/smf.conf

  info: # SMF Information for Network Slice Selection
    - s_nssai:
        - sst: 1
          dnn:
            - internet
        - sst: 2
          sd: 111111
          dnn:
            - internet
        - sst: 3
          sd: 111111
          dnn:
            - internet
        - sst: 4
          sd: 111111
          dnn:
            - internet
```

##### **File: `/etc/open5gs/nssf.yaml` (Full File)**

The Network Slice Selection Function (NSSF) configuration has been modified to recognize the defined network slices, enabling it to map the slices to the appropriate Network Slice Instances (NSIs).

```yaml
logger:
  file:
    path: /var/log/open5gs/nssf.log
#  level: info  # Options: fatal|error|warn|info(default)|debug|trace

global:
  max:
    ue: 1024  # Maximum number of User Equipments (UEs)
#   peer: 64

nssf:
  sbi:
    server:
      - address: 127.0.0.14
        port: 7777
    client:
#     nrf:
#       - uri: http://127.0.0.10:7777
      scp:
        - uri: http://127.0.0.200:7777
      nsi: # Network Slice Instance configuration
        - uri: http://127.0.0.10:7777 # NRF URI for this slice
          s_nssai:
            sst: 1
        - uri: http://127.0.0.10:7777
          s_nssai:
            sst: 2
            sd: 111111
        - uri: http://127.0.0.10:7777
          s_nssai:
            sst: 3
            sd: 111111
        - uri: http://127.0.0.10:7777
          s_nssai:
            sst: 4
            sd: 111111
```



#### **3.3. Add a Subscriber in Open5GS WebUI**

Before the UE can connect, you must register it as a subscriber in the core network.

Configure the WebUI : Modify `open5gs/webui/server/index.js` to listen on all interfaces.
```js
const hostname = process.env.HOSTNAME || '0.0.0.0';
```


1.  **Start the WebUI Server:**
    ```bash
    cd open5gs/webui
    npm run dev -- --host 0.0.0.0
    ```
2.  **Access the WebUI:** Open a browser on your Windows host and navigate to `http://<your_wsl_ip>:3000` (e.g., `http://172.28.251.212:3000`).
3.  **Login and Add Subscriber:**
    *   The default credentials are `admin` / `admin`.
    *   Click "Add Subscriber".
    *   Enter the following details, which must match the OAI UE configuration:
        *   **IMSI:** `999700000000001`
        *   **Key (K):** `465B5CE8B199B49FAA5F0A2EE238A6BC`
        *   **Operator Code (OPc):** `E8ED289DEBA952E4283B54E88E6183CA`
        *   **Slice/S-NSSAI -> SST:** `1`
        *   **Slice/S-NSSAI -> SD:** `ffffff`
        *   **Default APN/DNN:** `internet`

### **4. OAI RAN Configuration**

These configurations set up the OAI gNodeB components (CU-CP, CU-UP, DU) and the UE. All components are configured to use the WSL IP address (`172.28.251.212`) for communication, as they are all running on the same host.

##### **File: `/etc/oai/oai-cucp.conf` (OAI CU-CP Configuration)**

This file configures the Centralized Unit - Control Plane (CU-CP), which manages the control signaling for the gNodeB. It handles the NGAP interface to the AMF and the F1-C/E1 interfaces to the DU/CU-UP.

```conf
# Active gNodeB CU-CP instance name
Active_gNBs = ("oai-cu-cp");
Asn1_verbosity = "info";

gNBs =
(
  {
    # gNodeB Identifier
    gNB_ID = 0xe01;
    gNB_name = "oai-cu-cp";
    cell_type = "CELL_MACRO_GNB";

    # F1 Interface Configuration (to DU)
    tr_s_preference = "f1";
    local_s_address = "172.28.251.212";   # CU-CP F1-C IP address
    local_s_portc = 501;
    remote_s_address = "172.28.251.212";  # DU F1-C IP address
    remote_s_portc = 500;
    local_s_portd = 2154;                  # CU-CP F1-U port (forwards to CU-UP)
    remote_s_portd = 2153;                 # DU F1-U port

    # PLMN and Slice Information (must match Open5GS config)
    plmn_list = ({
      mcc = 999;
      mnc = 70;
      mnc_length = 2;
      snssaiList = ({ sst = 1; sd = 0xffffff; },{ sst = 2; sd = 0x111111; });
    });

    # Tracking Area and Cell Identity
    tracking_area_code = 1;
    nr_cellid = 12345678L;

    # AMF (Core Network) IP address
    amf_ip_address = ({ ipv4 = "172.28.251.212"; });

    # E1 Interface Configuration (to CU-UP)
    E1_INTERFACE = ({
        type = "cp";
        ipv4_cucp = "172.28.251.212";   # Local CU-CP E1 IP
        port_cucp = 38462;
        ipv4_cuup = "172.28.251.212";   # Remote CU-UP E1 IP
        port_cuup = 38462;
    });

    # Network Interfaces for Core Network Communication
    NETWORK_INTERFACES = {
      GNB_IPV4_ADDRESS_FOR_NG_AMF = "172.28.251.212"; # NGAP interface to AMF
      GNB_IPV4_ADDRESS_FOR_NGU = "172.28.251.212";    # NGU interface (forwarded to CU-UP)
      GNB_PORT_FOR_NGU = 2152;
    };
  }
);

# SCTP Protocol Parameters
SCTP: {
  SCTP_INSTREAMS = 5;
  SCTP_OUTSTREAMS = 5;
};

# Security Algorithm Configuration
security = {
  ciphering_algorithms = ("nea0");
  integrity_algorithms = ("nia2", "nia0");
  drb_ciphering = "yes";
  drb_integrity = "no";
};

# Logging Configuration
log_config: {
  global_log_level = "info";
  hw_log_level = "info"; 
  phy_log_level = "info"; 
  mac_log_level = "info"; 
  rlc_log_level = "info"; 
  pdcp_log_level = "info"; 
  rrc_log_level = "info"; 
  f1ap_log_level = "info"; 
  ngap_log_level = "info"; 
  sctp_log_level = "info";
};
```

##### **File: `/etc/oai/oai-cuup.conf` (OAI CU-UP Configuration)**

This file configures the Centralized Unit - User Plane (CU-UP). Its primary role is to handle the user data traffic, managing the NGU interface to the core's UPF and the F1-U/E1 interfaces to the DU/CU-CP.

```conf
# Active gNodeB CU-UP instance name
Active_gNBs = ("oai-cu-up");
Asn1_verbosity = "info";

gNBs =
(
 {
    # Identifiers for the gNodeB and this specific CU-UP
    gNB_ID = 0xe01;
    gNB_CU_UP_ID = 0xe00;
    gNB_name = "oai-cuup-sst1";
    cell_type = "CELL_MACRO_GNB";
    
    # Tracking Area 
    tracking_area_code = 1;
    
    # F1-U Interface Configuration (to DU)
    tr_s_preference = "f1";
    local_s_address = "172.28.251.212";   # CU-UP F1-U IP address
    remote_s_address = "172.28.251.212";  # DU F1-U IP address
    local_s_portd = 2154;
    remote_s_portd = 2153;
    local_s_portc = 501;
    remote_s_portc = 500;

    # PLMN and Slice Information
    plmn_list = ({
      mcc = 999;
      mnc = 70;
      mnc_length = 2;
      snssaiList = ({ sst = 1; sd = 0xffffff; },{ sst = 2; sd = 0x111111; });
    });

    # E1 Interface Configuration (to CU-CP)
    E1_INTERFACE = ({
      type = "up";
      ipv4_cucp = "172.28.251.212";   # Remote CU-CP E1 IP
      ipv4_cuup = "172.28.251.212";   # Local CU-UP E1 IP
    });

    # Network Interfaces for User Plane traffic
    NETWORK_INTERFACES = {
      GNB_IPV4_ADDRESS_FOR_NG_AMF = "0.0.0.0";        # CU-UP does not connect to AMF
      GNB_IPV4_ADDRESS_FOR_NGU    = "172.28.251.212"; # User plane interface to UPF
      GNB_PORT_FOR_S1U            = 2152;             # NGU port (3GPP Spec 2152)
    };
 }
);

# SCTP Protocol Parameters
SCTP: {
  SCTP_INSTREAMS  = 5;
  SCTP_OUTSTREAMS = 5;
};

# Logging Configuration
log_config: {
  global_log_level = "info";
  pdcp_log_level = "info"; 
  f1ap_log_level = "info"; 
  ngap_log_level = "info";
};
```

##### **File: `/etc/oai/oai-du.conf` (OAI DU Configuration)**

This file configures the Distributed Unit (DU), which handles the lower layers of the radio protocol stack and communicates with the CU via the F1 interface.

```conf
# Active gNodeB DU instance name
Active_gNBs = ( "oai-du" );
Asn1_verbosity = "info";

gNBs =
(
 {
    # Identifiers for the gNodeB and this specific DU
    gNB_ID = 0xe01;
    gNB_DU_ID = 0xe01;
    gNB_name = "oai-du";

    # PLMN and Slice Information
    plmn_list = ({
      mcc = 999;
      mnc = 70;
      mnc_length = 2;
      snssaiList = ({ sst = 1; sd = 0xffffff; },{ sst = 2; sd = 0x111111; });
    });

    # Tracking Area and Cell Identity
    tracking_area_code = 1;
    nr_cellid = 12345688L;
    min_rxtxtime = 6;

    # Physical Cell Configuration (Example for band n78)
    servingCellConfigCommon = {
      # spCellConfigCommon 
      physCellId = 1; 
      
      # downlinkConfigCommon 
      absoluteFrequencySSB = 641280; 
      dl_frequencyBand = 78; 
      dl_absoluteFrequencyPointA = 640008; 
      dl_offstToCarrier = 0; 
      dl_subcarrierSpacing = 1; 
      dl_carrierBandwidth = 106; 
      initialDLBWPlocationAndBandwidth = 28875; 
      initialDLBWPsubcarrierSpacing = 1; 
      initialDLBWPcontrolResourceSetZero = 12; 
      initialDLBWPsearchSpaceZero = 0;
      
      # uplinkConfigCommon 
      ul_frequencyBand = 78; 
      ul_offstToCarrier = 0; 
      ul_subcarrierSpacing = 1; 
      ul_carrierBandwidth = 106; 
      pMax = 20; 
      initialULBWPlocationAndBandwidth = 28875; 
      initialULBWPsubcarrierSpacing = 1;
      
      prach_ConfigurationIndex = 98; 
      prach_msg1_FDM = 0; 
      prach_msg1_FrequencyStart = 0; 
      zeroCorrelationZoneConfig = 13; 
      preambleReceivedTargetPower = -96; 
      preambleTransMax = 6; 
      powerRampingStep = 1; 
      ra_ResponseWindow = 4;      
      ssb_perRACH_OccasionAndCB_PreamblesPerSSB_PR = 4; 
      ssb_perRACH_OccasionAndCB_PreamblesPerSSB = 14; 
      ra_ContentionResolutionTimer = 7; 
      rsrp_ThresholdSSB = 19; 
      prach_RootSequenceIndex_PR = 2; 
      prach_RootSequenceIndex = 1; 
      msg1_SubcarrierSpacing = 1; 
      restrictedSetConfig = 0; 
      msg3_DeltaPreamble = 1; 
      p0_NominalWithGrant = -90;
      
      pucchGroupHopping = 0; 
      hoppingId = 40; 
      p0_nominal = -90; 
      ssb_PositionsInBurst_Bitmap = 1; 
      ssb_periodicityServingCell = 2; 
      dmrs_TypeA_Position = 0; 
      subcarrierSpacing = 1; 
      referenceSubcarrierSpacing = 1; 
      dl_UL_TransmissionPeriodicity = 6; 
      nrofDownlinkSlots = 7; 
      nrofDownlinkSymbols = 6; 
      nrofUplinkSlots = 2; 
      nrofUplinkSymbols = 4; 
      ssPBCH_BlockPower = -25;
    };
 }
);

# SCTP Protocol Parameters
SCTP: {
  SCTP_INSTREAMS  = 5;
  SCTP_OUTSTREAMS = 5;
};

# MAC/RLC Layer Configuration
MACRLCs = (
  {
    num_cc = 1; # Number of component carriers
    tr_s_preference = "local_L1";
    tr_n_preference = "f1"; # Use F1 interface for transport to CU
    
    # F1 Interface network details
    local_n_address = "172.28.251.212";   # DU F1 IP address
    remote_n_address = "172.28.251.212";  # CU F1 IP address
    local_n_portc = 500;                  # F1-C (Control) port
    remote_n_portc = 501;
    local_n_portd = 2153;                  # F1-U (User) port
    remote_n_portd = 2154;
    
    pusch_TargetSNRx10 = 200; 
    pucch_TargetSNRx10 = 200;
  }
);

L1s = ( 
  { 
    num_cc = 1; 
    tr_n_preference = "local_mac"; 
    prach_dtx_threshold = 200; 
    pucch0_dtx_threshold = 150; 
    ofdm_offset_divisor = 8; # set this to UINT_MAX for offset 0 
  } 
);

# RF Simulator Configuration (used instead of a physical radio)
rfsimulator: {
  serveraddr = "server"; # Run in server mode to accept UE connections
  serverport = 4043;
  options = (); # ("saviq"); or/and "chanmod"
  modelname = "AWGN";
  IQfile = "/tmp/rfsimulator.iqs";
};

# Logging Configuration
log_config: {
  global_log_level = "info";
  hw_log_level = "info"; 
  phy_log_level = "info"; 
  mac_log_level = "info"; 
  rlc_log_level = "info"; 
  f1ap_log_level = "info";
};

channelmod = {     
    max_chan = 10; 
    modellist = "modellist_rfsimu_1"; 
    modellist_rfsimu_1 = ( { 
      model_name = "rfsimu_channel_enB0"; 
      type = "AWGN"; 
      ploss_dB = 20; 
      noise_power_dB = -4; 
      forgetfact = 0; 
      offset = 0; 
      ds_tdl = 0; 
    }, { 
      model_name = "rfsimu_channel_ue0"; 
      type = "AWGN"; 
      ploss_dB = 20; 
      noise_power_dB = -2; 
      forgetfact = 0; 
      offset = 0; 
      ds_tdl = 0;
    } ); 
};


```

##### **File: `/etc/oai/nr-ue.conf` (OAI NR UE Configuration)**

This file configures the OAI NR User Equipment (UE). It contains the simulated SIM card (UICC) details, which must match a subscriber provisioned in the Open5GS core network.

```conf
# Simulated UICC (SIM Card) details
uicc0 = {
  # These values must match a subscriber in the Open5GS WebUI
  imsi = "999700000000001";
  key = "465B5CE8B199B49FAA5F0A2EE238A6BC";
  opc = "E8ED289DEBA952E4283B54E88E6183CA";

  # Requested Data Network and Slice
  dnn = "internet";
  nssai_sst = 1;
  nssai_sd = 0xFFFFFF; # Use 0x for hex values
};

position0 = { x = 0.0; y = 0.0; z = 6377900.0; }

thread-pool = "-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1";

channelmod = { 
    max_chan = 10; 
    modellist = "modellist_rfsimu_1"; 
    modellist_rfsimu_1 = ( { 
      # DL, modify on UE side 
      model_name = "rfsimu_channel_enB0"; 
      type = "AWGN"; 
      ploss_dB = 20; 
      noise_power_dB = -4; 
      forgetfact = 0; 
      offset = 0; 
      ds_tdl = 0; 
    }, { 
      # UL, modify on gNB side 
      model_name = "rfsimu_channel_ue0"; 
      type = "AWGN"; 
      ploss_dB = 20; 
      noise_power_dB = -2; 
      forgetfact = 0; 
      offset = 0; 
      ds_tdl = 0; 
    } ); 
};

# RF simulator configuration to connect to the DU's RF server
rfsimulator: {
  serveraddr = "172.28.251.212"; # The IP address of the DU
  serverport = 4043;
};

# Logging Configuration
log_config: {
  global_log_level = "info";
};
```

### **5. Starting the Network Components**

Open **five separate WSL terminals**. Each component must be run in its own terminal session.

**Terminal 1: Start Open5GS Core**
```bash
# Start all required Open5GS services individually 
sudo systemctl start open5gs-nrfd 
sudo systemctl start open5gs-udmd 
sudo systemctl start open5gs-ausfd 
sudo systemctl start open5gs-udrd 
sudo systemctl start open5gs-pcfd 
sudo systemctl start open5gs-smfd 
sudo systemctl start open5gs-amfd 
sudo systemctl start open5gs-upfd 

# Verify that all services are running 
sudo systemctl status open5gs-*
```
Wait for the core network to initialize completely. You can check the status with `sudo systemctl status open5gs-amfd`.
![[Pasted image 20250717163554.png]]

**Terminal 2: Start OAI CU-CP**
Navigate to the OAI build directory first.
```bash
cd oai/cmake_targets/ran_build/build
sudo ./nr-softmodem -O /etc/oai/oai-cucp.conf --sa
```

**Terminal 3: Start OAI CU-UP**
Navigate to the OAI build directory.
```bash
cd oai/cmake_targets/ran_build/build
sudo ./nr-cuup -O /etc/oai/oai-cuup.conf --sa
```

**Terminal 4: Start OAI DU**
Navigate to the OAI build directory.
```bash
cd oai/cmake_targets/ran_build/build
sudo ./nr-softmodem -O /etc/oai/oai-du.conf --rfsim --sa
```
The DU will start and wait for a connection from the CU and the RF simulator.

**Terminal 5: Start OAI NR-UE**
Navigate to the OAI build directory.
```bash
cd oai/cmake_targets/ran_build/build
sudo ./nr-uesoftmodem -O /etc/oai/nr-ue.conf --rfsim --sa --nokrnmod 

sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 48 -C 3619200000 --rfsim --rfsimulator.serveraddr 172.28.251.212 -O /etc/oai/nr-ue.conf
```
The UE will now attempt to connect to the DU via the RF simulator, and the registration process with the 5G core will begin.

#### output :

##### open5gs
```bash
user@IN-8KBKXD3:~$ sudo systemctl status open5gs-*
● open5gs-amfd.service - Open5GS AMF Daemon
     Loaded: loaded (/lib/systemd/system/open5gs-amfd.service; >     Active: active (running) since Thu 2025-07-17 11:13:55 IST>   Main PID: 349 (open5gs-amfd)
      Tasks: 2 (limit: 9264)
     Memory: 11.4M
        CPU: 1.277s
     CGroup: /system.slice/open5gs-amfd.service
             └─349 /usr/bin/open5gs-amfd -c /etc/open5gs/amf.ya>
Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.37>Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.37>Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.42>Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.42>Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.42>Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.42>Jul 17 11:13:58 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:58.17>Jul 17 11:13:58 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:58.17>Jul 17 11:13:58 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:58.17>Jul 17 11:13:58 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:58.17>
● open5gs-pcfd.service - Open5GS PCF Daemon
     Loaded: loaded (/lib/systemd/system/open5gs-pcfd.service; >     Active: active (running) since Thu 2025-07-17 11:13:58 IST>   Main PID: 556 (open5gs-pcfd)
      Tasks: 2 (limit: 9264)
     Memory: 8.3M
        CPU: 1.220s
     CGroup: /system.slice/open5gs-pcfd.service
             └─556 /usr/bin/open5gs-pcfd -c /etc/open5gs/pcf.ya>lines 1-30...skipping...
● open5gs-amfd.service - Open5GS AMF Daemon
     Loaded: loaded (/lib/systemd/system/open5gs-amfd.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2025-07-17 11:13:55 IST; 5h 17min ago
   Main PID: 349 (open5gs-amfd)
      Tasks: 2 (limit: 9264)
     Memory: 11.4M
        CPU: 1.277s
     CGroup: /system.slice/open5gs-amfd.service
             └─349 /usr/bin/open5gs-amfd -c /etc/open5gs/amf.yaml

Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.375: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../lib/sbi/>Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.375: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../lib/sbi/>Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.424: [sbi] INFO: [078e227c-62d1-41f0-ad35-b9f57b8e9d49] (NRF-notify) NF>Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.424: [sbi] INFO: [078e227c-62d1-41f0-ad35-b9f57b8e9d49] (NRF-notify) NF>Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.424: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.4:80] (../lib/sbi/con>Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.424: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../lib/sbi/c>Jul 17 11:13:58 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:58.176: [sbi] INFO: [08acd55e-62d1-41f0-a6ec-7f31951e4806] (NRF-notify) NF>Jul 17 11:13:58 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:58.177: [sbi] INFO: [08acd55e-62d1-41f0-a6ec-7f31951e4806] (NRF-notify) NF>Jul 17 11:13:58 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:58.177: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.13:80] (../lib/sbi/co>Jul 17 11:13:58 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:58.177: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../lib/sbi/>
● open5gs-pcfd.service - Open5GS PCF Daemon
     Loaded: loaded (/lib/systemd/system/open5gs-pcfd.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2025-07-17 11:13:58 IST; 5h 17min ago
   Main PID: 556 (open5gs-pcfd)
      Tasks: 2 (limit: 9264)
     Memory: 8.3M
        CPU: 1.220s
     CGroup: /system.slice/open5gs-pcfd.service
             └─556 /usr/bin/open5gs-pcfd -c /etc/open5gs/pcf.yaml

lines 1-31...skipping...
● open5gs-amfd.service - Open5GS AMF Daemon
     Loaded: loaded (/lib/systemd/system/open5gs-amfd.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2025-07-17 11:13:55 IST; 5h 17min ago
   Main PID: 349 (open5gs-amfd)
      Tasks: 2 (limit: 9264)
     Memory: 11.4M
        CPU: 1.277s
     CGroup: /system.slice/open5gs-amfd.service
             └─349 /usr/bin/open5gs-amfd -c /etc/open5gs/amf.yaml

Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.375: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../lib/sbi/context.c:2>Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.375: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../lib/sbi/context.c:2>Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.424: [sbi] INFO: [078e227c-62d1-41f0-ad35-b9f57b8e9d49] (NRF-notify) NF registered>Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.424: [sbi] INFO: [078e227c-62d1-41f0-ad35-b9f57b8e9d49] (NRF-notify) NF Profile up>Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.424: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.4:80] (../lib/sbi/context.c:2374)Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.424: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../lib/sbi/context.c:21>Jul 17 11:13:58 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:58.176: [sbi] INFO: [08acd55e-62d1-41f0-a6ec-7f31951e4806] (NRF-notify) NF registered>Jul 17 11:13:58 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:58.177: [sbi] INFO: [08acd55e-62d1-41f0-a6ec-7f31951e4806] (NRF-notify) NF Profile up>Jul 17 11:13:58 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:58.177: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.13:80] (../lib/sbi/context.c:237>Jul 17 11:13:58 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:58.177: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../lib/sbi/context.c:2>
● open5gs-pcfd.service - Open5GS PCF Daemon
     Loaded: loaded (/lib/systemd/system/open5gs-pcfd.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2025-07-17 11:13:58 IST; 5h 17min ago
   Main PID: 556 (open5gs-pcfd)
      Tasks: 2 (limit: 9264)
     Memory: 8.3M
        CPU: 1.220s
     CGroup: /system.slice/open5gs-pcfd.service
             └─556 /usr/bin/open5gs-pcfd -c /etc/open5gs/pcf.yaml

Jul 17 11:13:58 IN-8KBKXD3 open5gs-pcfd[556]: 07/17 11:13:58.179: [sbi] INFO: [08adf402-62d1-41f0-93f4-27646812f29b] Subscription created until>Jul 17 11:13:58 IN-8KBKXD3 open5gs-pcfd[556]: 07/17 11:13:58.182: [sbi] INFO: [0761b570-62d1-41f0-b1d7-f917a6192bc5] (NRF-profile-get) NF regis>lines 1-33...skipping...● open5gs-amfd.service - Open5GS AMF Daemon
     Loaded: loaded (/lib/systemd/system/open5gs-amfd.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2025-07-17 11:13:55 IST; 5h 17min ago
   Main PID: 349 (open5gs-amfd)
      Tasks: 2 (limit: 9264)
     Memory: 11.4M
        CPU: 1.277s
     CGroup: /system.slice/open5gs-amfd.service
             └─349 /usr/bin/open5gs-amfd -c /etc/open5gs/amf.yaml

Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.375: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../lib/sbi/context.c:2113)
Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.375: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../lib/sbi/context.c:2113)
Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.424: [sbi] INFO: [078e227c-62d1-41f0-ad35-b9f57b8e9d49] (NRF-notify) NF registered (../lib/sbi>Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.424: [sbi] INFO: [078e227c-62d1-41f0-ad35-b9f57b8e9d49] (NRF-notify) NF Profile updated [type:>Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.424: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.4:80] (../lib/sbi/context.c:2374)
Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.424: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../lib/sbi/context.c:2113)
Jul 17 11:13:58 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:58.176: [sbi] INFO: [08acd55e-62d1-41f0-a6ec-7f31951e4806] (NRF-notify) NF registered (../lib/sbi>Jul 17 11:13:58 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:58.177: [sbi] INFO: [08acd55e-62d1-41f0-a6ec-7f31951e4806] (NRF-notify) NF Profile updated [type:>Jul 17 11:13:58 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:58.177: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.13:80] (../lib/sbi/context.c:2374)
Jul 17 11:13:58 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:58.177: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../lib/sbi/context.c:2113)

● open5gs-pcfd.service - Open5GS PCF Daemon
     Loaded: loaded (/lib/systemd/system/open5gs-pcfd.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2025-07-17 11:13:58 IST; 5h 17min ago
   Main PID: 556 (open5gs-pcfd)
      Tasks: 2 (limit: 9264)
     Memory: 8.3M
        CPU: 1.220s
     CGroup: /system.slice/open5gs-pcfd.service
             └─556 /usr/bin/open5gs-pcfd -c /etc/open5gs/pcf.yaml

Jul 17 11:13:58 IN-8KBKXD3 open5gs-pcfd[556]: 07/17 11:13:58.179: [sbi] INFO: [08adf402-62d1-41f0-93f4-27646812f29b] Subscription created until 2025-07-18T>Jul 17 11:13:58 IN-8KBKXD3 open5gs-pcfd[556]: 07/17 11:13:58.182: [sbi] INFO: [0761b570-62d1-41f0-b1d7-f917a6192bc5] (NRF-profile-get) NF registered (../li>Jul 17 11:13:58 IN-8KBKXD3 open5gs-pcfd[556]: 07/17 11:13:58.182: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.15:80] (../lib/sbi/context.c:2374)
Jul 17 11:13:58 IN-8KBKXD3 open5gs-pcfd[556]: 07/17 11:13:58.182: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.15:7777] (../lib/sbi/context.c:2113)
Jul 17 11:13:58 IN-8KBKXD3 open5gs-pcfd[556]: 07/17 11:13:58.183: [sbi] INFO: [07601210-62d1-41f0-86ec-8f353462c376] (NRF-profile-get) NF registered (../li>Jul 17 11:13:58 IN-8KBKXD3 open5gs-pcfd[556]: 07/17 11:13:58.183: [sbi] INFO: Setup NF EndPoint(addr) [127.0.1.250:7777] (../lib/sbi/context.c:2374)
Jul 17 11:13:58 IN-8KBKXD3 open5gs-pcfd[556]: 07/17 11:13:58.184: [sbi] INFO: [08ad93ea-62d1-41f0-825a-d3e0d59523f1] (NRF-notify) NF registered (../lib/sbi>lines 1-38...skipping...
● open5gs-amfd.service - Open5GS AMF Daemon
     Loaded: loaded (/lib/systemd/system/open5gs-amfd.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2025-07-17 11:13:55 IST; 5h 17min ago
   Main PID: 349 (open5gs-amfd)
      Tasks: 2 (limit: 9264)
     Memory: 11.4M
        CPU: 1.277s
     CGroup: /system.slice/open5gs-amfd.service
             └─349 /usr/bin/open5gs-amfd -c /etc/open5gs/amf.yaml

Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.375: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../lib/sbi/context.c:2113)
Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.375: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../lib/sbi/context.c:2113)
Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.424: [sbi] INFO: [078e227c-62d1-41f0-ad35-b9f57b8e9d49] (NRF-notify) NF registered (../lib/sbi/nnrf-handler.>Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.424: [sbi] INFO: [078e227c-62d1-41f0-ad35-b9f57b8e9d49] (NRF-notify) NF Profile updated [type:SMF] (../lib/s>Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.424: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.4:80] (../lib/sbi/context.c:2374)
Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.424: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../lib/sbi/context.c:2113)
Jul 17 11:13:58 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:58.176: [sbi] INFO: [08acd55e-62d1-41f0-a6ec-7f31951e4806] (NRF-notify) NF registered (../lib/sbi/nnrf-handler.>Jul 17 11:13:58 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:58.177: [sbi] INFO: [08acd55e-62d1-41f0-a6ec-7f31951e4806] (NRF-notify) NF Profile updated [type:PCF] (../lib/s>Jul 17 11:13:58 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:58.177: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.13:80] (../lib/sbi/context.c:2374)
Jul 17 11:13:58 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:58.177: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../lib/sbi/context.c:2113)

● open5gs-pcfd.service - Open5GS PCF Daemon
     Loaded: loaded (/lib/systemd/system/open5gs-pcfd.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2025-07-17 11:13:58 IST; 5h 17min ago
   Main PID: 556 (open5gs-pcfd)
      Tasks: 2 (limit: 9264)
     Memory: 8.3M
        CPU: 1.220s
     CGroup: /system.slice/open5gs-pcfd.service
             └─556 /usr/bin/open5gs-pcfd -c /etc/open5gs/pcf.yaml

Jul 17 11:13:58 IN-8KBKXD3 open5gs-pcfd[556]: 07/17 11:13:58.179: [sbi] INFO: [08adf402-62d1-41f0-93f4-27646812f29b] Subscription created until 2025-07-18T11:13:58.17906>Jul 17 11:13:58 IN-8KBKXD3 open5gs-pcfd[556]: 07/17 11:13:58.182: [sbi] INFO: [0761b570-62d1-41f0-b1d7-f917a6192bc5] (NRF-profile-get) NF registered (../lib/sbi/nf-sm.c:>Jul 17 11:13:58 IN-8KBKXD3 open5gs-pcfd[556]: 07/17 11:13:58.182: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.15:80] (../lib/sbi/context.c:2374)
Jul 17 11:13:58 IN-8KBKXD3 open5gs-pcfd[556]: 07/17 11:13:58.182: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.15:7777] (../lib/sbi/context.c:2113)
Jul 17 11:13:58 IN-8KBKXD3 open5gs-pcfd[556]: 07/17 11:13:58.183: [sbi] INFO: [07601210-62d1-41f0-86ec-8f353462c376] (NRF-profile-get) NF registered (../lib/sbi/nf-sm.c:>Jul 17 11:13:58 IN-8KBKXD3 open5gs-pcfd[556]: 07/17 11:13:58.183: [sbi] INFO: Setup NF EndPoint(addr) [127.0.1.250:7777] (../lib/sbi/context.c:2374)
Jul 17 11:13:58 IN-8KBKXD3 open5gs-pcfd[556]: 07/17 11:13:58.184: [sbi] INFO: [08ad93ea-62d1-41f0-825a-d3e0d59523f1] (NRF-notify) NF registered (../lib/sbi/nnrf-handler.>Jul 17 11:13:58 IN-8KBKXD3 open5gs-pcfd[556]: 07/17 11:13:58.184: [sbi] INFO: [08ad93ea-62d1-41f0-825a-d3e0d59523f1] (NRF-notify) NF Profile updated [type:UDR] (../lib/s>lines 1-39
● open5gs-amfd.service - Open5GS AMF Daemon
     Loaded: loaded (/lib/systemd/system/open5gs-amfd.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2025-07-17 11:13:55 IST; 5h 17min ago
   Main PID: 349 (open5gs-amfd)
      Tasks: 2 (limit: 9264)
     Memory: 11.4M
        CPU: 1.277s
     CGroup: /system.slice/open5gs-amfd.service
             └─349 /usr/bin/open5gs-amfd -c /etc/open5gs/amf.yaml

Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.375: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../lib/sbi/context.c:2113)
Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.375: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../lib/sbi/context.c:2113)
Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.424: [sbi] INFO: [078e227c-62d1-41f0-ad35-b9f57b8e9d49] (NRF-notify) NF registered (../lib>Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.424: [sbi] INFO: [078e227c-62d1-41f0-ad35-b9f57b8e9d49] (NRF-notify) NF Profile updated [t>Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.424: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.4:80] (../lib/sbi/context.c:2374)
Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.424: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../lib/sbi/context.c:2113)
Jul 17 11:13:58 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:58.176: [sbi] INFO: [08acd55e-62d1-41f0-a6ec-7f31951e4806] (NRF-notify) NF registered (../lib>Jul 17 11:13:58 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:58.177: [sbi] INFO: [08acd55e-62d1-41f0-a6ec-7f31951e4806] (NRF-notify) NF Profile updated [t>Jul 17 11:13:58 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:58.177: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.13:80] (../lib/sbi/context.c:2374)
Jul 17 11:13:58 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:58.177: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../lib/sbi/context.c:2113)

● open5gs-pcfd.service - Open5GS PCF Daemon
     Loaded: loaded (/lib/systemd/system/open5gs-pcfd.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2025-07-17 11:13:58 IST; 5h 17min ago
   Main PID: 556 (open5gs-pcfd)
      Tasks: 2 (limit: 9264)
     Memory: 8.3M
        CPU: 1.220s
     CGroup: /system.slice/open5gs-pcfd.service
             └─556 /usr/bin/open5gs-pcfd -c /etc/open5gs/pcf.yaml

Jul 17 11:13:58 IN-8KBKXD3 open5gs-pcfd[556]: 07/17 11:13:58.179: [sbi] INFO: [08adf402-62d1-41f0-93f4-27646812f29b] Subscription created until 2025-07>Jul 17 11:13:58 IN-8KBKXD3 open5gs-pcfd[556]: 07/17 11:13:58.182: [sbi] INFO: [0761b570-62d1-41f0-b1d7-f917a6192bc5] (NRF-profile-get) NF registered (.>Jul 17 11:13:58 IN-8KBKXD3 open5gs-pcfd[556]: 07/17 11:13:58.182: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.15:80] (../lib/sbi/context.c:2374)
Jul 17 11:13:58 IN-8KBKXD3 open5gs-pcfd[556]: 07/17 11:13:58.182: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.15:7777] (../lib/sbi/context.c:2113)
Jul 17 11:13:58 IN-8KBKXD3 open5gs-pcfd[556]: 07/17 11:13:58.183: [sbi] INFO: [07601210-62d1-41f0-86ec-8f353462c376] (NRF-profile-get) NF registered (.>lines 1-36
● open5gs-amfd.service - Open5GS AMF Daemon
     Loaded: loaded (/lib/systemd/system/open5gs-amfd.service; enabled; vendor pr>
     Active: active (running) since Thu 2025-07-17 11:13:55 IST; 5h 17min ago
   Main PID: 349 (open5gs-amfd)
      Tasks: 2 (limit: 9264)
     Memory: 11.4M
        CPU: 1.277s
     CGroup: /system.slice/open5gs-amfd.service
             └─349 /usr/bin/open5gs-amfd -c /etc/open5gs/amf.yaml

Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.375: [sbi] INFO: Set>
Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.375: [sbi] INFO: Set>
Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.424: [sbi] INFO: [07>
Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.424: [sbi] INFO: [07>
Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.424: [sbi] INFO: Set>
Jul 17 11:13:56 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:56.424: [sbi] INFO: Set>
Jul 17 11:13:58 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:58.176: [sbi] INFO: [08>
Jul 17 11:13:58 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:58.177: [sbi] INFO: [08>
Jul 17 11:13:58 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:58.177: [sbi] INFO: Set>
Jul 17 11:13:58 IN-8KBKXD3 open5gs-amfd[349]: 07/17 11:13:58.177: [sbi] INFO: Set>

● open5gs-pcfd.service - Open5GS PCF Daemon
     Loaded: loaded (/lib/systemd/system/open5gs-pcfd.service; enabled; vendor pr>
     Active: active (running) since Thu 2025-07-17 11:13:58 IST; 5h 17min ago
   Main PID: 556 (open5gs-pcfd)
      Tasks: 2 (limit: 9264)
     Memory: 8.3M
        CPU: 1.220s
     CGroup: /system.slice/open5gs-pcfd.service
             └─556 /usr/bin/open5gs-pcfd -c /etc/open5gs/pcf.yaml

Jul 17 11:13:58 IN-8KBKXD3 open5gs-pcfd[556]: 07/17 11:13:58.179: [sbi] INFO: [08>
Jul 17 11:13:58 IN-8KBKXD3 open5gs-pcfd[556]: 07/17 11:13:58.182: [sbi] INFO: [07>
Jul 17 11:13:58 IN-8KBKXD3 open5gs-pcfd[556]: 07/17 11:13:58.182: [sbi] INFO: Set>
Jul 17 11:13:58 IN-8KBKXD3 open5gs-pcfd[556]: 07/17 11:13:58.182: [sbi] INFO: Set>
Jul 17 11:13:58 IN-8KBKXD3 open5gs-pcfd[556]: 07/17 11:13:58.183: [sbi] INFO: [07>
Jul 17 11:13:58 IN-8KBKXD3 open5gs-pcfd[556]: 07/17 11:13:58.183: [sbi] INFO: Set>
Jul 17 11:13:58 IN-8KBKXD3 open5gs-pcfd[556]: 07/17 11:13:58.184: [sbi] INFO: [08>
Jul 17 11:13:58 IN-8KBKXD3 open5gs-pcfd[556]: 07/17 11:13:58.184: [sbi] INFO: [08>
lines 1-39
```

##### cucp
```bash
user@IN-8KBKXD3:~/oai/cmake_targets/ran_build/build$ sudo ./nr-softmodem -O /etc/oai/oai-cucp.conf --sa
CMDLINE: "./nr-softmodem" "-O" "/etc/oai/oai-cucp.conf" "--sa"
[CONFIG] function config_libconfig_init returned 0
[UTIL]   running in SA mode (no --phy-test, --do-ra, --nsa option present)
[OPT]   OPT disabled
[HW]   Version: Branch: develop Abrev. Hash: e82fde247a Date: Fri Jul 11 08:03:04 2025 +0000
[GNB_APP]   Initialized RAN Context: RC.nb_nr_inst = 1, RC.nb_nr_macrlc_inst = 0, RC.nb_nr_L1_inst = 0, RC.nb_RU = 0, RC.nb_nr_CC[0] = 0
[GNB_APP]   F1AP: gNB_CU_id[0] 3585
[GNB_APP]   F1AP: gNB_CU_name[0] oai-cu-cp
[GNB_APP]   SDAP layer is disabled
[GNB_APP]   Data Radio Bearer count 1
[GNB_APP]   Parsed IPv4 address for NG AMF: 172.28.251.212
[UTIL]   threadCreate() for TASK_SCTP: creating thread with affinity ffffffff, priority 50
[X2AP]   X2AP is disabled.
[UTIL]   threadCreate() for TASK_NGAP: creating thread with affinity ffffffff, priority 50
[NGAP]   Registered new gNB[0] and macro gNB id 3585
[UTIL]   threadCreate() for TASK_RRC_GNB: creating thread with affinity ffffffff, priority 50
[NGAP]   [gNB 0] check the amf registration state
[UTIL]   threadCreate() for TASK_GNB_APP: creating thread with affinity ffffffff, priority 50
[NR_RRC]   Entering main loop of NR_RRC message task
[NGAP]   Send NGSetupRequest to AMF
[NGAP]   3585 -> 0000e010
[UTIL]   threadCreate() for TASK_CU_F1: creating thread with affinity ffffffff, priority 50
[UTIL]   threadCreate() for time source realtime: creating thread with affinity ffffffff, priority 2
[F1AP]   Starting F1AP at CU
[UTIL]   threadCreate() for TASK_CUCP_E1: creating thread with affinity ffffffff, priority 50
[UTIL]   time manager configuration: [time source: reatime] [mode: standalone] [server IP: 127.0.0.1} [server port: 7374] (server IP/port not used)
[F1AP]   F1AP_CU_SCTP_REQ(create socket) for 172.28.251.212 len 15
[F1AP]   In F1AP connection, don't start GTP-U, as we have also E1AP
[NGAP]   Supported PLMN 0: MCC=999 MNC=70
[NGAP]   Supported slice (PLMN 0): SST=0x01 SD=000
[NGAP]   Supported slice (PLMN 0): SST=0x01 SD=171717
[NGAP]   Supported slice (PLMN 0): SST=0x01 SD=000
[NGAP]   Supported slice (PLMN 0): SST=0x02 SD=171717
[NGAP]   Supported slice (PLMN 0): SST=0x03 SD=171717
[NGAP]   Supported slice (PLMN 0): SST=0x04 SD=171717
[NGAP]   Received NGSetupResponse from AMF
[E1AP]   Starting E1AP at CU CP
[GTPU]   Configuring GTPu
[GTPU]   SA mode
[GNB_APP]   [gNB 0] Received NGAP_REGISTER_GNB_CNF: associated AMF 1
[E1AP]   E1AP_CUCP_SCTP_REQ(create socket) for 172.28.251.212 len 15
E2 agent is DISABLED (for activation, define .e2_agent.{near_ric_ip_addr,sm_dir} parameters)
TYPE <CTRL-C> TO TERMINATE
[NR_RRC]   Accepting new CU-UP ID 3584 name oai-cuup-sst1 (assoc_id 12)

```

##### cuup
```bash
user@IN-8KBKXD3:~/oai/cmake_targets/ran_build/build$ sudo ./nr-cuup -O /etc/oai/oai-cuup.conf --sa
CMDLINE: "./nr-cuup" "-O" "/etc/oai/oai-cuup.conf" "--sa"
[CONFIG] function config_libconfig_init returned 0
[UTIL]   threadCreate() for time source realtime: creating thread with affinity ffffffff, priority 2
[UTIL]   time manager configuration: [time source: reatime] [mode: standalone] [server IP: 127.0.0.1} [server port: 7374] (server IP/port not used)
[HW]   Version: Branch: develop Abrev. Hash: e82fde247a Date: Fri Jul 11 08:03:04 2025 +0000
[UTIL]   threadCreate() for TASK_SCTP: creating thread with affinity ffffffff, priority 50
[UTIL]   threadCreate() for TASK_GTPV1_U: creating thread with affinity ffffffff, priority 50
[UTIL]   threadCreate() for TASK_CUUP_E1: creating thread with affinity ffffffff, priority 50
[GTPU]   Configuring GTPu
[E1AP]   Starting E1AP at CU UP
[GTPU]   SA mode
[GTPU]   Initializing UDP for local address 172.28.251.212 with port 2154
E2 agent is DISABLED (for activation, define .e2_agent.{near_ric_ip_addr,sm_dir} parameters)
[GTPU]   Created gtpu instance id: 92
TYPE <CTRL-C> TO TERMINATE
[GTPU]   Configuring GTPu address : 172.28.251.212, port : 2152
[GTPU]   Initializing UDP for local address 172.28.251.212 with port 2152
[GTPU]   bind: Address already in use
[GTPU]   failed to bind socket: 172.28.251.212 2152
[GTPU]   can't create GTP-U instance
[GTPU]   Created gtpu instance id: -1
[E1AP]   Failed to create CUUP N3 UDP listener
[E1AP]   E1 connection established (SCTP_STATE_ESTABLISHED)
```

##### du
```bash
user@IN-8KBKXD3:~/oai/cmake_targets/ran_build/build$ sudo ./nr-softmodem -O /etc/oai/oai-du.conf --rfsim --sa
CMDLINE: "./nr-softmodem" "-O" "/etc/oai/oai-du.conf" "--rfsim" "--sa"
[CONFIG] function config_libconfig_init returned 0
[UTIL]   running in SA mode (no --phy-test, --do-ra, --nsa option present)
[OPT]   OPT disabled
[HW]   Version: Branch: develop Abrev. Hash: e82fde247a Date: Fri Jul 11 08:03:04 2025 +0000
[GNB_APP]   Initialized RAN Context: RC.nb_nr_inst = 1, RC.nb_nr_macrlc_inst = 1, RC.nb_nr_L1_inst = 1, RC.nb_RU = 0, RC.nb_nr_CC[0] = 1
[NR_PHY]   Initializing gNB RAN context: RC.nb_nr_L1_inst = 1
[NR_PHY]   Registered with MAC interface module (0x5f3bfbb210d0)
[NR_PHY]   Initializing NR L1: RC.nb_nr_L1_inst = 1
[NR_PHY]   L1_RX_THREAD_CORE -1 (15)
[NR_PHY]   TX_AMP = 519 (-36 dBFS)
[PHY]   No prs_config configuration found..!!
[GNB_APP]   pdsch_AntennaPorts N1 1 N2 1 XP 1 pusch_AntennaPorts 1
[GNB_APP]   RU information not present in config file. Assuming physical antenna ports equal to logical antenna ports 1
[GNB_APP]   minTXRXTIME 6
[GNB_APP]   SIB1 TDA 1
[GNB_APP]   CSI-RS 0, SRS 0, SINR:0, 256 QAM may be on, delta_MCS off, maxMIMO_Layers -1, HARQ feedback enabled, num DLHARQ:16, num ULHARQ:16
[NR_MAC]   No RedCap configuration found
[GNB_APP]   sr_ProhibitTimer 0, sr_TransMax 64, sr_ProhibitTimer_v1700 0, t300 400, t301 400, t310 2000, n310 10, t311 3000, n311 1, t319 400
[NR_MAC]   Candidates per PDCCH aggregation level on UESS: L1: 0, L2: 2, L4: 0, L8: 0, L16: 0
[RRC]   Read in ServingCellConfigCommon (PhysCellId 0, ABSFREQSSB 660960, DLBand 78, ABSFREQPOINTA 660000, DLBW 217,RACH_TargetReceivedPower -118
[RRC]   absoluteFrequencySSB 660960 corresponds to 3914400000 Hz

Assertion (gscn >= start_gscn && gscn <= end_gscn) failed!
In check_ssb_raster() ../../../common/utils/nr/nr_common.c:415
GSCN 8134 corresponding to SSB frequency 3914400000 does not belong to GSCN range for band 78

Exiting execution
../../../common/utils/nr/nr_common.c:415 check_ssb_raster() Exiting OAI softmodem: _Assert_Exit_
user@IN-8KBKXD3:~/oai/cmake_targets/ran_build/build$
```

##### nr-ue
```bash
user@IN-8KBKXD3:~/oai/cmake_targets/ran_build/build$ sudo ./nr-uesoftmodem -O /etc/oai/nr-ue.conf --rfsim --sa --nokrnmod
CMDLINE: "./nr-uesoftmodem" "-O" "/etc/oai/nr-ue.conf" "--rfsim" "--sa" "--nokrnmod"
[CONFIG] function config_libconfig_init returned 0
[UTIL]   running in SA mode (no --phy-test, --do-ra, --nsa option present)
[UTIL]   threadCreate() for Tpool0_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool1_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool2_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool3_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool4_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool5_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool6_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool7_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool8_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool9_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool10_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool11_-1: creating thread with affinity ffffffff, priority 97
[OPT]   OPT disabled
[HW]   Version: Branch: develop Abrev. Hash: e82fde247a Date: Fri Jul 11 08:03:04 2025 +0000
[NR_RRC]   create TASK_RRC_NRUE
[UTIL]   threadCreate() for TASK_RRC_NRUE: creating thread with affinity ffffffff, priority 50
[UTIL]   threadCreate() for TASK_NAS_NRUE: creating thread with affinity ffffffff, priority 50
[SIM]   UICC simulation: IMSI=999700000000001, IMEISV=6754567890123413, Ki=465B5CE8B199B49FAA5F0A2EE238A6BC, OPc=E8ED289DEBA952E4283B54E88E6183CA, DNN=internet, SST=0x01, SD=0xffffff
[NR_MAC]   [UE0] Initializing MAC
[NR_MAC]   Initializing dl and ul config_request. num_slots = 20
[RLC]   Activated srb0 for UE 0
[UTIL]   threadCreate() for time source iq samples: creating thread with affinity ffffffff, priority 2
[UTIL]   time manager configuration: [time source: iq_samples] [mode: standalone] [server IP: 127.0.0.1} [server port: 7374] (server IP/port not used)
[PHY]   Set UE_fo_compensation 0, UE_scan_carrier 0, UE_no_timing_correction 0
, chest-freq 0, chest-time 0
[PHY]   Set UE nb_rx_antenna 1, nb_tx_antenna 1, threequarter_fs 0, ssb_start_subcarrier 516
[PHY]   SA init parameters. DL freq 0 UL offset 0 SSB numerology 1 N_RB_DL 106

Assertion (0) failed!
In get_freq_range_from_freq() ../../../common/utils/nr/nr_common.c:1444
Undefined Frequency Range for frequency 0 Hz

Exiting execution
../../../common/utils/nr/nr_common.c:1444 get_freq_range_from_freq() Exiting OAI softmodem: _Assert_Exit_
Aborted
user@IN-8KBKXD3:~/oai/cmake_targets/ran_build/build$
```

[[possible fix for error oai start comp]]


---
### **6. Testing End-to-End Connectivity**

Once the UE is connected, a new tunnel interface, `oaitun_ue1`, will be created.

1.  **Check for the Tunnel Interface:** In a new terminal, check the network interfaces. You should see `oaitun_ue1` with an IP address from the `10.45.0.0/16` subnet.
    ```bash
    ip a
    ```

2.  **Ping the Internet:** Use the tunnel interface to ping an external address like Google's DNS.
    ```bash
    ping -I oaitun_ue1 8.8.8.8
    ```
    A successful ping indicates that your end-to-end 5G network is fully operational on your single WSL instance.

### **Conclusion**

This guide has detailed the process of consolidating a complex, multi-component 5G testbed onto a single WSL Ubuntu 20.04 system. By patching the source code for compatibility, reconfiguring the network interfaces to use a single IP address, and running each component, you can successfully simulate a full 5G SA connection. This setup provides a powerful and accessible environment for developers and researchers to experiment with and test 5G network functions.