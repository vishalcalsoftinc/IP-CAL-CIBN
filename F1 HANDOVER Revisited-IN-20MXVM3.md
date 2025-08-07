Of course. This is an excellent and efficient modification to your testbed. By consolidating the CU-CP and CU-UP onto a single VM, you free up a machine to act as the second DU without needing to provision a new one.

Here is a revised, step-by-step guide that reflects your new architecture. It details the necessary configuration changes and execution steps to perform the F1 handover.

---

### **Guide: Implementing F1 Handover with a Consolidated CU**

This guide adapts the official OAI handover procedure to your updated GCP environment, where `oai-cucp-vm` runs both the CU-CP and CU-UP, and `oai-cuup-vm` is repurposed as the second DU.

#### **Assumed IP Address Configuration**

Based on your previous setup, we will assume the following IP address mapping:

*   `oai-nr-ue-vm`: `172.17.0.91`
*   `oai-du-vm` (Source DU-0): `172.17.0.92`
*   `oai-cucp-vm` (CU-CP & CU-UP): `172.17.0.93`
*   `oai-cuup-vm` (Target DU-1): `172.17.0.94`
*   `open5gs-vm`: `172.17.0.95`

---

### **Part 1: OAI Installation and Re-Building with Telnet Support**

To trigger the handover, the OAI binaries must be rebuilt with telnet support. This needs to be done on all OAI nodes.

1.  **SSH into your four OAI VMs:** `oai-cucp-vm`, `oai-du-vm`, `oai-cuup-vm`, and `oai-nr-ue-vm`.

2.  On **each of the four VMs**, ensure the OAI repository is cloned and then run the build command. This command builds the gNB (for CU and DU roles), the nrUE, and crucially enables the telnet server library.

    ```bash
    # Run on oai-cucp-vm, oai-du-vm, oai-cuup-vm, oai-nr-ue-vm
    sudo apt update
    sudo apt install -y git
    cd ~
    git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git
    cd openairinterface5g
    source oaienv
    cd cmake_targets
    ./build_oai --ninja --nrUE --gNB --build-lib telnetsrv
    ```

---

### **Part 2: Creating Handover-Specific Configurations**

Now, we will create or modify the configuration files on each node to reflect the new architecture.

#### **2.1. Consolidated CU-UP Configuration (on `oai-cucp-vm`)**

Create the corresponding CU-UP configuration on the same VM.

1.  Create the file `/etc/oai/oai-cuup.conf` and copy the content from the cuup vm.
#### **2.3. First DU (DU0) Configuration (on `oai-du-vm`)**

Create a new handover-specific configuration for the first DU. The `remote_n_address` now points to the consolidated CU VM (`172.17.0.93`).

1.  Create the file `/etc/oai/oai-du.conf`:
    ```bash
    # On oai-du-vm
    nano /etc/oai/oai-du.conf
    ```

2.  Paste the following content. This DU uses **PCI 0**.
 
```conf
Active_gNBs = ( "oai-cu-cp");
# Asn1_verbosity, choice in: none, info, annoying
Asn1_verbosity = "info";

gNBs =
(
 {
    ////////// Identification parameters:
    gNB_ID = 0xe00;
    gNB_DU_ID = 0xe01;

#     cell_type =  "CELL_MACRO_GNB";

    gNB_name  =  "oai-cu-cp";

    // Tracking area code, 0x0000 and 0xfffe are reserved values
    tracking_area_code  =  1;
    #plmn_list = ({ mcc = 208; mnc = 99; mnc_length = 2; snssaiList = ({ sst = 1 }, { sst = 2 }, { sst = 3 } ) });
    plmn_list = ({ mcc = 999; mnc = 70; mnc_length = 2; snssaiList =  ({ sst = 1, sd = 0x000001 },{ sst = 1, sd = 0xFFFFFF }) });


    nr_cellid = 12345678L;

    ////////// Physical parameters:

    min_rxtxtime                                              = 6;

    servingCellConfigCommon = (
    {
 #spCellConfigCommon

      physCellId                                                    = 0;

#  downlinkConfigCommon
    #frequencyInfoDL
      # this is 3600 MHz + 43 PRBs@30kHz SCS (same as initial BWP)
      absoluteFrequencySSB                                          = 641280;
      dl_frequencyBand                                                 = 78;
      # this is 3600 MHz
      dl_absoluteFrequencyPointA                                       = 640008;
      #scs-SpecificCarrierList
        dl_offstToCarrier                                              = 0;
# subcarrierSpacing
# 0=kHz15, 1=kHz30, 2=kHz60, 3=kHz120
        dl_subcarrierSpacing                                           = 1;
        dl_carrierBandwidth                                            = 106;
     #initialDownlinkBWP
      #genericParameters
        # this is RBstart=27,L=48 (275*(L-1))+RBstart
        initialDLBWPlocationAndBandwidth                               = 28875; # 6366 12925 12956 28875 12952
# subcarrierSpacing
# 0=kHz15, 1=kHz30, 2=kHz60, 3=kHz120
        initialDLBWPsubcarrierSpacing                                           = 1;
      #pdcch-ConfigCommon
        initialDLBWPcontrolResourceSetZero                              = 12;
        initialDLBWPsearchSpaceZero                                             = 0;

  #uplinkConfigCommon
     #frequencyInfoUL
      ul_frequencyBand                                                 = 78;
      #scs-SpecificCarrierList
      ul_offstToCarrier                                              = 0;
# subcarrierSpacing
# 0=kHz15, 1=kHz30, 2=kHz60, 3=kHz120
      ul_subcarrierSpacing                                           = 1;
      ul_carrierBandwidth                                            = 106;
      pMax                                                          = 20;
     #initialUplinkBWP
      #genericParameters
        initialULBWPlocationAndBandwidth                            = 28875;
# subcarrierSpacing
# 0=kHz15, 1=kHz30, 2=kHz60, 3=kHz120
        initialULBWPsubcarrierSpacing                                           = 1;
      #rach-ConfigCommon
        #rach-ConfigGeneric
          prach_ConfigurationIndex                                  = 98;
#prach_msg1_FDM
#0 = one, 1=two, 2=four, 3=eight
          prach_msg1_FDM                                            = 0;
          prach_msg1_FrequencyStart                                 = 0;
          zeroCorrelationZoneConfig                                 = 13;
          preambleReceivedTargetPower                               = -96;
#preamblTransMax (0...10) = (3,4,5,6,7,8,10,20,50,100,200)
          preambleTransMax                                          = 6;
#powerRampingStep
# 0=dB0,1=dB2,2=dB4,3=dB6
        powerRampingStep                                            = 1;
#ra_ReponseWindow
#1,2,4,8,10,20,40,80
        ra_ResponseWindow                                           = 4;
#ssb_perRACH_OccasionAndCB_PreamblesPerSSB_PR
#1=oneeighth,2=onefourth,3=half,4=one,5=two,6=four,7=eight,8=sixteen
        ssb_perRACH_OccasionAndCB_PreamblesPerSSB_PR                = 4;
#one (0..15) 4,8,12,16,...60,64
        ssb_perRACH_OccasionAndCB_PreamblesPerSSB                   = 14;
#ra_ContentionResolutionTimer
#(0..7) 8,16,24,32,40,48,56,64
        ra_ContentionResolutionTimer                                = 7;
        rsrp_ThresholdSSB                                           = 19;
#prach-RootSequenceIndex_PR
#1 = 839, 2 = 139
        prach_RootSequenceIndex_PR                                  = 2;
        prach_RootSequenceIndex                                     = 1;
        # SCS for msg1, can only be 15 for 30 kHz < 6 GHz, takes precendence over the one derived from prach-ConfigIndex
        #
        msg1_SubcarrierSpacing                                      = 1,
# restrictedSetConfig
# 0=unrestricted, 1=restricted type A, 2=restricted type B
        restrictedSetConfig                                         = 0,

        msg3_DeltaPreamble                                          = 1;
        p0_NominalWithGrant                                         =-90;

# pucch-ConfigCommon setup :
# pucchGroupHopping
# 0 = neither, 1= group hopping, 2=sequence hopping
        pucchGroupHopping                                           = 0;
        hoppingId                                                   = 40;
        p0_nominal                                                  = -90;

      ssb_PositionsInBurst_Bitmap                                   = 1;

# ssb_periodicityServingCell
# 0 = ms5, 1=ms10, 2=ms20, 3=ms40, 4=ms80, 5=ms160, 6=spare2, 7=spare1
      ssb_periodicityServingCell                                    = 2;

# dmrs_TypeA_position
# 0 = pos2, 1 = pos3
      dmrs_TypeA_Position                                           = 0;

# subcarrierSpacing
# 0=kHz15, 1=kHz30, 2=kHz60, 3=kHz120
      subcarrierSpacing                                             = 1;


  #tdd-UL-DL-ConfigurationCommon
# subcarrierSpacing
# 0=kHz15, 1=kHz30, 2=kHz60, 3=kHz120
      referenceSubcarrierSpacing                                    = 1;
      # pattern1
      # dl_UL_TransmissionPeriodicity
      # 0=ms0p5, 1=ms0p625, 2=ms1, 3=ms1p25, 4=ms2, 5=ms2p5, 6=ms5, 7=ms10
      dl_UL_TransmissionPeriodicity                                 = 6;
      nrofDownlinkSlots                                             = 7;
      nrofDownlinkSymbols                                           = 6;
      nrofUplinkSlots                                               = 2;
      nrofUplinkSymbols                                             = 4;

      ssPBCH_BlockPower                                             = -25;
     }

  );


    # ------- SCTP definitions
    SCTP :
    {
        # Number of streams to use in input/output
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
    local_n_address = "172.17.0.92";   #du vm IP
    remote_n_address = "172.17.0.93";    #  "172.17.0.94";  #cu-up vm IP
    local_n_portc   = 500;
    local_n_portd   = 2153;
    remote_n_portc  = 38472;
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
  ofdm_offset_divisor = 8; #set this to UINT_MAX for offset 0
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
serveraddr = "server";
    serverport = 4043;
    options = (); #("saviq"); or/and "chanmod"
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

#/* configuration for channel modelisation */
#/* To be included in main config file when */
#/* channel modelisation is used (rfsimulator with chanmod options enabled) */
channelmod = {
  max_chan = 10;
  modellist = "modellist_rfsimu_1";
  modellist_rfsimu_1 = (
    { # DL, modify on UE side
      model_name     = "rfsimu_channel_enB0"
      type           = "AWGN";
      ploss_dB       = 20;
      noise_power_dB = -4;
      forgetfact     = 0;
      offset         = 0;
      ds_tdl         = 0;
    },
    { # UL, modify on gNB side
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

#e2_agent = {
 # near_ric_ip_addr = "10.0.9.20";
  #sm_dir = "/usr/local/lib/flexric/"
#}

```

#### **2.4. Second DU (DU1) Configuration (on `oai-cuup-vm`)**

Create the configuration for the second DU on the repurposed `oai-cuup-vm`.

1.  Create the file `/etc/oai/oai-du-ho.conf`:
    ```bash
    # On oai-cuup-vm
    nano /etc/oai/oai-du-ho.conf
    ```

2.  Paste the following content. This DU uses its own IP (`172.17.0.94`), a different `gNB_DU_ID`, and **PCI 1**.

```conf 


```

---

### **Part 3: Executing the Handover Test**

Follow this startup sequence precisely. Each command requires its own SSH terminal.

**1. (Verify) Core Network**
Ensure the Open5GS services are active on `open5gs-vm`.

**2. Start the CU-CP (with Telnet)**
On `oai-cucp-vm`, start the CU-CP using the new handover configuration.

```bash
# In Terminal 1 for oai-cucp-vm
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo -E ./nr-softmodem -O /etc/oai/oai-cucp-ho.conf --sa --       telnetsrv --telnetsrv.shrmod ci
```
The CU-CP will start and wait for E1 and F1 connections.

**3. Start the CU-UP**
In a **second terminal connected to the same `oai-cucp-vm`**, start the CU-UP.

```bash
# In Terminal 2 for oai-cucp-vm
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo -E ./nr-cuup -O /etc/oai/oai-cuup-ho.conf --telnetsrv --telnetsrv.shrmod ci
```
You should see the E1 link between the CU-CP and CU-UP being established in the logs of both terminals.

**4. Start the First DU (DU0)**
On `oai-du-vm`, start the first DU.

```bash
# On oai-du-vm
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O ~/oai-du.conf --rfsim --sa
```
The CU-CP log will confirm the F1 SETUP from DU ID `0xe00`.

**5. Start the NR UE**
On `oai-nr-ue-vm`, start the UE. It will act as the RF simulator server.

```bash
# On oai-nr-ue-vm
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --ssb 516 --rfsim --rfsimulator.serveraddr 172.17.0.92 -O /etc/oai/nr-ue.conf 
```
Wait for the UE to attach to DU0 (PCI 0). You will see logs for RRC connection, Registration Accept, and PDU session establishment.

**6. Start the Second DU (DU1)**
On `oai-cuup-vm` (the repurposed VM), start the second DU.

```bash
# On oai-cuup-vm
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O ~/oai-du0-ho.conf --rfsim --sa
```
The CU-CP log will now show a *second* F1 SETUP, this time from DU ID `0xe01`. The CU is now aware of both DUs and ready for handover.

**7. Trigger the F1 Handover**
From any terminal, use `netcat` to send the trigger command to the telnet server running on the CU-CP.

```bash
echo ci trigger_f1_ho | nc 172.17.0.93 9090 && echo
```

### **Part 4: Verification**

If the handover is successful, you will see the following activity in your terminals:

*   **`oai-cucp-vm`:** The CU-CP log will show "Handover Request," sending an `RRCReconfiguration` to the UE, and then receiving "Handover Request Acknowledge" from the target DU (DU1).
*   **`oai-du-vm` (Source DU0):** Logs will indicate the release of the UE context.
*   **`oai-cuup-vm` (Target DU1):** Logs will show it receives the HANDOVER REQUEST from the CU and establishes a connection with the UE.
*   **`oai-nr-ue-vm`:** The UE will log the reception of the `RRCReconfiguration` containing mobility control information and will begin synchronization with the new cell (PCI 1).