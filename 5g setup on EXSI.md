

| VM Name       | IP           |
| :------------ | :----------- |
| oai-nr-ue-vm1 | 172.17.42.81 |
| oai-du-vm2    | 172.17.42.82 |
| oai-cucp-vm3  | 172.17.42.83 |
| oai-du2-vm4   | 172.17.42.84 |
| oai-cuup-vm5  | 172.17.42.85 |
| open5gs-vm6   | 172.17.42.86 |

## open5gs build

SSH into the `open5gs-vm` and follow these steps.

1.  **Install Prerequisites:**
    ```bash
    sudo apt update
    sudo apt install nano iptables git
    sudo apt install -y software-properties-common curl gnupg git
    ```

2.  **Install MongoDB:**
    ```bash
    curl -fsSL https://pgp.mongodb.com/server-8.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg --dearmor
    echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg] https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
    sudo apt update
    sudo apt install -y mongodb-org
    sudo systemctl start mongod
    sudo systemctl enable mongod
    ```

3.  **Install Open5GS:**
    ```bash
    sudo add-apt-repository ppa:open5gs/latest
    sudo apt update
    sudo apt install open5gs -y
    ```

4.  **Configure Open5GS Components:**
Update the configuration files with the new IP address of your `open5gs-vm`.

*   **In `/etc/open5gs/amf.yaml`:**
    ```yaml
    #...
    amf:
      sbi: #...
      ngap:
        server:
          - address: 172.17.42.86 # open5gs vm ip
    #...
	
	
	# restart amf services
	sudo systemctl restart open5gs-amfd
	
	# amf logs can be found in /var/log/open5gs/amf.log
	sudo tail -f /var/log/open5gs/amf.log
    ```
*   **In `/etc/open5gs/upf.yaml`:**
    ```yaml
    #...
    upf:
      pfcp: #...
      gtpu:
        server:
          - address: 172.17.42.86 # open5gs vm ip
    #...
    	
	
	# restart upf services
	sudo systemctl restart open5gs-upfd
	
	# upf logs can be found in /var/log/open5gs/upf.log
	sudo tail -f /var/log/open5gs/upf.log
    ```
*   The `smf.yaml` and `nssf.yaml` files do not need IP changes for this basic setup. The default slice configurations are sufficient to start.


5.  **Enable NAT:**
    ```bash
    sudo sysctl -w net.ipv4.ip_forward=1
    sudo iptables -t nat -A POSTROUTING -o ens34 -j MASQUERADE
    sudo iptables -I FORWARD 1 -j ACCEPT
    ```


6.  **Setup and Add Subscriber in WebUI:**

*   **Install Node.js and dependencies:**
    ```bash
    sudo apt update
    curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
    sudo apt install nodejs -y
    ```
*   **Clone the WebUI repo and install packages:**
    ```bash
    cd ~
    git clone https://github.com/open5gs/open5gs
    cd open5gs/webui
    npm install
    ```
*   **Change host ip:** 
```bash
sudo nano /server/index.js

# chnage 
const _hostname = process.env.HOSTNAME || 'localhost';
# to
const _hostname = process.env.HOSTNAME || '172.17.42.86';
```
*  **Start the WebUI:**
    This command will run the user interface in the foreground.
    ```bash
    npm run dev
    ```
*   **Accessing the WebUI:**
    * The WebUI is now running on `172.17.42.86:9999`. 
    *  Now, open a browser on your local machine and go to `http://172.17.42.86:9999`.
    *   Login with default credentials: `admin` / `1423`.
    *   Add a new subscriber as per the document's instructions (IMSI, Key, OPC).
	    * IMSI: 999700000000001
	    * Subscriber Key: 465B5CE8B199B49FAA5F0A2EE238A6BC
	    * USIM Type: OPc
	    * Operator Key: E8ED289DEBA952E4283B54E88E6183CA


## oai component build
On **each of the five OAI VMs** (`oai-cucp-vm`,`oai-cuup-vm`, `oai-du-vm`, `oai-du1-vm`, `oai-nr-ue-vm`), perform the following steps. This build enables the telnet server required to trigger the handover.

```bash
# Run on all four OAI VMs
sudo apt update
sudo apt install git cmake ninja-build build-essential

cd ~
git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git
cd openairinterface5g
source oaienv
cd cmake_targets
# This command builds all necessary components with telnet support
./build_oai -I
./build_oai --ninja --nrUE --gNB --build-lib telnetsrv
```



## nr-ue.conf
```nr-ue.conf
uicc0 = {
  imsi = "999700000000001";
  key = "465B5CE8B199B49FAA5F0A2EE238A6BC";
  opc = "E8ED289DEBA952E4283B54E88E6183CA";
  dnn = "internet";
  nssai_sst = 1;
  nssai_sd = 0xFFFFFF;
}

rfsimulator: {
serveraddr = "server";
    serverport = 4043;
    options = (); #("saviq"); or/and "chanmod"
    modelname = "AWGN";
    IQfile = "/tmp/rfsimulator.iqs"
}

```
## oai-du.conf
```oai-du.conf
Active_gNBs = ( "oai-cu-cp");
Asn1_verbosity = "info";

gNBs =
(
 {
    gNB_ID = 0xe00;
    gNB_DU_ID = 0xe01;
    gNB_name  =  "oai-cu-cp";
    tracking_area_code  =  1;
    plmn_list = ({ mcc = 999; mnc = 70; mnc_length = 2; snssaiList =  ({ sst = 1, sd = 0x000001 },{ sst = 1, sd = 0xFFFFFF }) });
    nr_cellid = 12345678L;
    min_rxtxtime = 6;

    servingCellConfigCommon = (
    {
      physCellId = 1;
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
      ra_ResponseWindow = 5;
      ssb_perRACH_OccasionAndCB_PreamblesPerSSB_PR = 4;
      ssb_perRACH_OccasionAndCB_PreamblesPerSSB = 14;
      ra_ContentionResolutionTimer = 7;
      rsrp_ThresholdSSB = 19;
      prach_RootSequenceIndex_PR = 2;
      prach_RootSequenceIndex = 1;
      msg1_SubcarrierSpacing = 1,
      restrictedSetConfig = 0,
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
     }
  );

    SCTP :
    {
        SCTP_INSTREAMS  = 2;
        SCTP_OUTSTREAMS = 2;
    };
  }
);

MACRLCs = (
  {
    num_cc           = 1;
    tr_s_preference  = "local_L1";
    tr_n_preference  = "f1";
    local_n_address = "172.17.42.82";  # du2 vm ip
    remote_n_address = "172.17.42.83"; # cu-cp vm ip
    local_n_portc   = 500;
    local_n_portd   = 2153;
    remote_n_portc  = 501;
    remote_n_portd  = 2153;
    pusch_TargetSNRx10          = 200;
    pucch_TargetSNRx10          = 200;
  }
);

L1s = (
{
  num_cc = 1;
  tr_n_preference = "local_mac";
  prach_dtx_threshold = 200;
  pucch0_dtx_threshold = 150;
  ofdm_offset_divisor = 8;
}
);

RUs = (
    {
       local_rf       = "yes"
         nb_tx          = 1
         nb_rx          = 1
         att_tx         = 0
         att_rx         = 0;
         bands          = [78];
         max_pdschReferenceSignalPower = -27;
         max_rxgain                    = 114;
         eNB_instances  = [0];
         clock_src = "internal";
    }
);

rfsimulator: {
serveraddr = "172.17.42.81";
    serverport = 4043;
    options = ();
    modelname = "AWGN";
    IQfile = "/tmp/rfsimulator.iqs"
}

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
  modellist_rfsimu_1 = (
    {
      model_name     = "rfsimu_channel_enB0"
      type           = "AWGN";
      ploss_dB       = 20;
      noise_power_dB = -4;
      forgetfact     = 0;
      offset         = 0;
      ds_tdl         = 0;
    },
    {
      model_name     = "rfsimu_channel_ue0"
      type           = "AWGN";
      ploss_dB       = 20;
      noise_power_dB = -2;
      forgetfact     = 0;
      offset         = 0;
      ds_tdl         = 0;
    }
  );
};


```


## oai-du2.conf
```oai-du2.conf

Active_gNBs = ( "oai-cu-cp");
Asn1_verbosity = "info";

gNBs =
(
 {
    gNB_ID = 0xe00;
    gNB_DU_ID = 0xe02;
    gNB_name  =  "oai-cu-cp";
    tracking_area_code  =  2;
    plmn_list = ({ mcc = 999; mnc = 70; mnc_length = 2; snssaiList =  ({ sst = 1, sd = 0x000001 },{ sst = 1, sd = 0xFFFFFF }) });
    nr_cellid = 12345679L;
    min_rxtxtime = 6;

    servingCellConfigCommon = (
    {
      physCellId = 2;
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
      ra_ResponseWindow = 5; # changed from 4
      ssb_perRACH_OccasionAndCB_PreamblesPerSSB_PR = 4;
      ssb_perRACH_OccasionAndCB_PreamblesPerSSB = 14;
      ra_ContentionResolutionTimer = 7;
      rsrp_ThresholdSSB = 19;
      prach_RootSequenceIndex_PR = 2;
      prach_RootSequenceIndex = 1;
      msg1_SubcarrierSpacing = 1,
      restrictedSetConfig = 0,
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
     }
  );

    SCTP :
    {
        SCTP_INSTREAMS  = 2;
        SCTP_OUTSTREAMS = 2;
    };
  }
);

MACRLCs = (
  {
    num_cc           = 1;
    tr_s_preference  = "local_L1";
    tr_n_preference  = "f1";
    local_n_address = "172.17.42.84";  # du2 vm ip
    remote_n_address = "172.17.42.83"; # cu-cp vm ip
    local_n_portc   = 500;
    local_n_portd   = 2153;
    remote_n_portc  = 501;
    remote_n_portd  = 2153;
    pusch_TargetSNRx10          = 200;
    pucch_TargetSNRx10          = 200;
  }
);

L1s = (
{
  num_cc = 1;
  tr_n_preference = "local_mac";
  prach_dtx_threshold = 200;
  pucch0_dtx_threshold = 150;
  ofdm_offset_divisor = 8;
}
);

RUs = (
    {
       local_rf       = "yes"
         nb_tx          = 1
         nb_rx          = 1
         att_tx         = 0
         att_rx         = 0;
         bands          = [78];
         max_pdschReferenceSignalPower = -27;
         max_rxgain                    = 114;
         eNB_instances  = [0];
         clock_src = "internal";
    }
);

rfsimulator: {
serveraddr = "172.17.42.81"; # nr-ue vm ip
    serverport = 4043;
    options = ();
    modelname = "AWGN";
    IQfile = "/tmp/rfsimulator.iqs"
}

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
  modellist_rfsimu_1 = (
    {
      model_name     = "rfsimu_channel_enB0"
      type           = "AWGN";
      ploss_dB       = 20;
      noise_power_dB = -4;
      forgetfact     = 0;
      offset         = 0;
      ds_tdl         = 0;
    },
    {
      model_name     = "rfsimu_channel_ue0"
      type           = "AWGN";
      ploss_dB       = 20;
      noise_power_dB = -2;
      forgetfact     = 0;
      offset         = 0;
      ds_tdl         = 0;
    }
  );
};


```


## oai-cucp.conf
```oai-cucp.conf
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

    SCTP :
    {
        SCTP_INSTREAMS  = 5;
        SCTP_OUTSTREAMS = 5;
    };

    amf_ip_address = ({ ipv4 = "172.17.42.86"; });

    E1_INTERFACE =
    (
      {
        type = "cp";
        ipv4_cucp = "172.17.42.83"; # CU-CP IP
        port_cucp = 38462;
        ipv4_cuup = "172.17.42.85"; # CU-UP IP
        port_cuup = 38462;
      }
    )

    NETWORK_INTERFACES :
    {
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

log_config :
{
   global_log_level = "info";
   hw_log_level     = "info";
   phy_log_level    = "info";
   mac_log_level    = "info";
   rlc_log_level    = "debug";
   pdcp_log_level   = "info";
   rrc_log_level    = "info";
   f1ap_log_level   = "info";
   ngap_log_level   = "debug";
   sctp_log_level   = "info";
};



```

## oai-cuup.conf
```oai-cuup.conf
Active_gNBs = ( "oai-cu-cp");
# Asn1_verbosity, choice in: none, info, annoying
Asn1_verbosity = "none";

gNBs =
(
 {
    gNB_ID = 0xe00;
    gNB_CU_UP_ID = 0xe00;

#     cell_type =  "CELL_MACRO_GNB";

    gNB_name  =  "oai-cuup-sd1";

    // Tracking area code, 0x0000 and 0xfffe are reserved values
    tracking_area_code  =  1;
    plmn_list = ({ mcc = 999; mnc = 70; mnc_length = 2; snssaiList = ({ sst = 1, sd = 0x000001 },{ sst = 1, sd = 0xFFFFFF }) });


    tr_s_preference = "f1";

    local_s_address = "172.17.42.85";  #cu-up vm IP
    remote_s_address = "0.0.0.0"; #du vm IP
    local_s_portc   = 501;
    local_s_portd   = 2153;
    remote_s_portc  = 500;
    remote_s_portd  = 2153;

    # ------- SCTP definitions
    SCTP :
    {
        # Number of streams to use in input/output
        SCTP_INSTREAMS  = 5;
        SCTP_OUTSTREAMS = 5;
    };

    E1_INTERFACE =
    (
      {
        type = "cp";
        ipv4_cucp = "172.17.42.83"; # CU-CP IP
        port_cucp = 38462;
        ipv4_cuup = "172.17.42.85"; # CU-UP IP
        port_cuup = 38462;
      }
    )

    NETWORK_INTERFACES :
    {
        GNB_IPV4_ADDRESS_FOR_NG_AMF              = "172.17.42.86"; # AMF IP
        GNB_IPV4_ADDRESS_FOR_NGU                 = "172.17.42.85"; # CU-UP IP
        GNB_PORT_FOR_S1U                         = 2152; # Spec 2152
    };
  }
);

log_config : {
  global_log_level = "info";
  pdcp_log_level   = "info";
  f1ap_log_level   = "info";
  ngap_log_level   = "info";
};

# e2_agent = {
#   near_ric_ip_addr = "10.0.9.20";
#   sm_dir = "/usr/local/lib/flexric/"
# }



```


## start commands 
**1. Start Open5GS Services (on `open5gs-vm`)**

```bash
# MongoDB (if using local database)  
sudo systemctl start mongod

# Start Open5GS core components  
sudo systemctl start open5gs-mmed  
sudo systemctl start open5gs-sgwcd  
sudo systemctl start open5gs-smfd  
sudo systemctl start open5gs-amfd  
sudo systemctl start open5gs-upfd  
sudo systemctl start open5gs-pcfd  
sudo systemctl start open5gs-ausfd  
sudo systemctl start open5gs-nrfd  
sudo systemctl start open5gs-nssfd  
sudo systemctl start open5gs-bsfd  
sudo systemctl start open5gs-udmd  
sudo systemctl start open5gs-udrd  
sudo systemctl start open5gs-pcfd

# Verify that all services are running 
sudo systemctl status open5gs-*

# To check logs
sudo journalctl -u open5gs-amfd -n 200 --no-pager
sudo journalctl -u open5gs-upfd -n 200 --no-pager

```

**2. Enable NAT (on `open5gs-vm`)**

```bash
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -o ens34 -j MASQUERADE
sudo iptables -I FORWARD 1 -j ACCEPT
```

**3. Start the CU-CP (on `oai-cu-vm`)**
```bash
# On: oai-cucp-vm
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo -E ./nr-softmodem -O /etc/oai/oai-cucp.conf --sa --telnetsrv --telnetsrv.shrmod ci
```

**4. Start the CU-UP (on `oai-cu-vm`)**
```bash
# On: oai-cucp-vm (in a new terminal)
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo -E ./nr-cuup -O /etc/oai/oai-cuup.conf --telnetsrv --telnetsrv.shrmod ci

```

**5. Start the NR UE as the RF Server (on `oai-nr-ue-vm`)**  
This starts the UE in server mode, waiting for the DUs to connect.

```bash
# On: oai-nr-ue-vm
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --ssb 516 --rfsim -O /etc/oai/nr-ue.conf
```

**6. Start the Source DU (DU0) as a Client (on `oai-du0-vm`)**  
The DU will connect to the UE's RF server, and the UE will attach to the network.

```bash
# On: oai-du-vm
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O /etc/oai/oai-du.conf --rfsim --sa
```

**7. Start the Target DU (DU1) as a Client (on `oai-du1-vm`)**  
This DU also connects to the UE's RF server. The CU-CP log will show a second F1 setup.

```bash
# On: oai-cuup-vm
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O /etc/oai/oai-du2.conf --rfsim --sa
```

**8. Trigger the F1 Handover**  
Run this command from any machine.

```bash
echo ci trigger_f1_ho | nc 172.17.42.83 9090 && echo
```