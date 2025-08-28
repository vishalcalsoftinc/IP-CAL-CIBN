
1. installed 


```oai-cu.conf
# CU (CU-CP / CU-UP consolidated) - single-host loopback example
Active_gNBs = ( "gNB-local-CU" );
Asn1_verbosity = "none";

gNBs =
(
 {
    gNB_ID = 0xe00;
    gNB_name  = "gNB-local-CU";
    tracking_area_code  = 1;
    plmn_list = ({ mcc = 999; mnc = 70; mnc_length = 2; snssaiList = ({ sst = 1, sd = 0xFFFFFF }) });
    nr_cellid = 12345678L;
    tr_s_preference = "f1";

    # CU local address (bind here)
    local_s_address = "127.0.0.11";
    # Example ports (control and data - keep stable)
    local_s_portc   = 38472;    # F1-C server control port (CU side)
    local_s_portd   = 2153;     # F1-U (GTP-U) for CU side (use 2153 to avoid conflict with UPF 2152)

    # when talking to Open5GS AMF on the same host:
    amf_ip_address = ({ ipv4 = "127.0.0.5"; });  # Open5GS AMF loopback

    NETWORK_INTERFACES :
    {
        GNB_IPV4_ADDRESS_FOR_NG_AMF = "127.0.0.11";  # CU N2 bind
        GNB_IPV4_ADDRESS_FOR_NGU    = "127.0.0.11";  # CU N3 bind (if CU-UP present)
        GNB_PORT_FOR_S1U            = 2152;         # standard GTP-U port for external GTP (UPF)
    };

    SCTP :
    {
        SCTP_INSTREAMS  = 5;
        SCTP_OUTSTREAMS = 5;
    };

    log_config :
    {
       global_log_level = "info";
       f1ap_log_level = "debug";
       ngap_log_level = "debug";
    };
  }
);

```

```oai-du0.conf
Active_gNBs = ( "du-rfsim");
# Asn1_verbosity, choice in: none, info, annoying
Asn1_verbosity = "none";

gNBs =
(
 {
    ////////// Identification parameters:
    gNB_ID = 0xe00;
    gNB_DU_ID = 0xe00;
    gNB_name  =  "du-rfsim";

    // Tracking area code, 0x0000 and 0xfffe are reserved values
    tracking_area_code  =  1;
    plmn_list = ({ mcc = 999; mnc = 70; mnc_length = 2; snssaiList = ({ sst = 1, sd = 0xFFFFFF }) });

    nr_cellid = 12345678L;

    ////////// Physical parameters:

    min_rxtxtime = 4;

    servingCellConfigCommon = (
    {
 #spCellConfigCommon

      physCellId                                               = 0;

#  downlinkConfigCommon
    #frequencyInfoDL
      # this is 3600 MHz + 43 PRBs@30kHz SCS (same as initial BWP)
      absoluteFrequencySSB                                      = 630048;
      dl_frequencyBand                                          = 78;
      # this is 3600 MHz
      dl_absoluteFrequencyPointA                                = 628776;
      #scs-SpecificCarrierList
        dl_offstToCarrier                                       = 0;
# subcarrierSpacing
# 0=kHz15, 1=kHz30, 2=kHz60, 3=kHz120
        dl_subcarrierSpacing                                    = 1;
        dl_carrierBandwidth                                     = 106;
     #initialDownlinkBWP
      #genericParameters
        # this is RBstart=27,L=48 (275*(L-1))+RBstart
        initialDLBWPlocationAndBandwidth                         = 28875;
# subcarrierSpacing
# 0=kHz15, 1=kHz30, 2=kHz60, 3=kHz120
        initialDLBWPsubcarrierSpacing                             = 1;
      #pdcch-ConfigCommon
        initialDLBWPcontrolResourceSetZero                        = 12;
        initialDLBWPsearchSpaceZero                               = 0;

  #uplinkConfigCommon
     #frequencyInfoUL
      ul_frequencyBand                                             = 78;
      #scs-SpecificCarrierList
      ul_offstToCarrier                                            = 0;
# subcarrierSpacing
# 0=kHz15, 1=kHz30, 2=kHz60, 3=kHz120
      ul_subcarrierSpacing                                         = 1;
      ul_carrierBandwidth                                          = 106;
      pMax                                                         = 20;
     #initialUplinkBWP
      #genericParameters
        initialULBWPlocationAndBandwidth                           = 28875;
# subcarrierSpacing
# 0=kHz15, 1=kHz30, 2=kHz60, 3=kHz120
        initialULBWPsubcarrierSpacing                              = 1;
      #rach-ConfigCommon
        #rach-ConfigGeneric
          prach_ConfigurationIndex                                 = 98;
#prach_msg1_FDM
#0 = one, 1=two, 2=four, 3=eight
          prach_msg1_FDM                                           = 0;
          prach_msg1_FrequencyStart                                = 0;
          zeroCorrelationZoneConfig                                = 13;
          preambleReceivedTargetPower                              = -96;
#preamblTransMax (0...10) = (3,4,5,6,7,8,10,20,50,100,200)
          preambleTransMax                                         = 6;
#powerRampingStep
# 0=dB0,1=dB2,2=dB4,3=dB6
        powerRampingStep                                           = 1;
#ra_ReponseWindow
#1,2,4,8,10,20,40,80
        ra_ResponseWindow                                          = 4;
#ssb_perRACH_OccasionAndCB_PreamblesPerSSB_PR
#1=oneeighth,2=onefourth,3=half,4=one,5=two,6=four,7=eight,8=sixteen
        ssb_perRACH_OccasionAndCB_PreamblesPerSSB_PR               = 4;
#one (0..15) 4,8,12,16,...60,64
        ssb_perRACH_OccasionAndCB_PreamblesPerSSB                  = 14;
#ra_ContentionResolutionTimer
#(0..7) 8,16,24,32,40,48,56,64
        ra_ContentionResolutionTimer                               = 7;
        rsrp_ThresholdSSB                                          = 19;
#prach-RootSequenceIndex_PR
#1 = 839, 2 = 139
        prach_RootSequenceIndex_PR                                 = 2;
        prach_RootSequenceIndex                                    = 1;
        # SCS for msg1, can only be 15 for 30 kHz < 6 GHz, takes precendence over the one derived from prach-ConfigIndex
        #
        msg1_SubcarrierSpacing                                     = 1,
# restrictedSetConfig
# 0=unrestricted, 1=restricted type A, 2=restricted type B
        restrictedSetConfig                                        = 0,

        msg3_DeltaPreamble                                         = 1;
        p0_NominalWithGrant                                        =-90;

# pucch-ConfigCommon setup :
# pucchGroupHopping
# 0 = neither, 1= group hopping, 2=sequence hopping
        pucchGroupHopping                                          = 0;
        hoppingId                                                  = 40;
        p0_nominal                                                 = -90;

      ssb_PositionsInBurst_Bitmap                                  = 1;

# ssb_periodicityServingCell
# 0 = ms5, 1=ms10, 2=ms20, 3=ms40, 4=ms80, 5=ms160, 6=spare2, 7=spare1
      ssb_periodicityServingCell                                   = 2;

# dmrs_TypeA_position
# 0 = pos2, 1 = pos3
      dmrs_TypeA_Position                                           = 0;

# subcarrierSpacing
# 0=kHz15, 1=kHz30, 2=kHz60, 3=kHz120
      subcarrierSpacing                                            = 1;


  #tdd-UL-DL-ConfigurationCommon
# subcarrierSpacing
# 0=kHz15, 1=kHz30, 2=kHz60, 3=kHz120
      referenceSubcarrierSpacing                                   = 1;
      # pattern1
      # dl_UL_TransmissionPeriodicity
      # 0=ms0p5, 1=ms0p625, 2=ms1, 3=ms1p25, 4=ms2, 5=ms2p5, 6=ms5, 7=ms10
      dl_UL_TransmissionPeriodicity                                = 6;
      nrofDownlinkSlots                                            = 7;
      nrofDownlinkSymbols                                          = 6;
      nrofUplinkSlots                                              = 2;
      nrofUplinkSymbols                                            = 4;

      ssPBCH_BlockPower                                            = -25;
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

MACRLCs = ({
    num_cc              = 1;
    tr_s_preference     = "local_L1";
    tr_n_preference     = "f1";

    # --- NETWORK bindings (edited for single-host loopbacks) ---
    local_n_address     = "127.0.0.12";   # DU0 local loopback (was 127.0.0.4)
    remote_n_address    = "127.0.0.11";   # CU loopback (was 127.0.0.3)

    # F1-C (SCTP) local/remote ports (unique per DU)
    local_n_portc       = 38473;
    remote_n_portc      = 38472;

    # F1-U (GTP-U) local/remote ports: use 2153 for F1-U to avoid Open5GS UPF 2152 conflict
    local_n_portd       = 2153;
    remote_n_portd      = 2153;

    pusch_FailureThres  = 1000;
});

L1s = ({
    num_cc = 1;
    tr_n_preference = "local_mac";
    prach_dtx_threshold = 200;
    pucch0_dtx_threshold = 150;
    ofdm_offset_divisor = 8; #set this to UINT_MAX for offset 0
});

RUs = ({

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
});

rfsimulator: {
  serveraddr = "127.0.0.14";   # UE RF-sim server loopback
  serverport = 4043;
  options = (); #("saviq"); or/and "chanmod"
  modelname = "AWGN";
  IQfile = "/tmp/rfsimulator.iqs"
}

log_config: {
  global_log_level = "info";
  hw_log_level     = "info";
  nr_phy_log_level = "info";
  nr_mac_log_level = "info";
  rlc_log_level    = "info";
  pdcp_log_level   = "info";
  rrc_log_level    = "info";
  f1ap_log_level   = "info";
  ngap_log_level   = "debug";
};

#e2_agent = {
#  near_ric_ip_addr = "127.0.0.1";
#  sm_dir = "/usr/local/lib/flexric/"
#};

```

```nr-ue.conf
# UE (nr-ue) settings
uicc0 = {
  imsi = "999700000000001";
  key = "465B5CE8B199B49FAA5F0A2EE238A6BC";
  opc = "E8ED289DEBA952E4283B54E88E6183CA";
  dnn = "internet";
  nssai_sst = 1;
  nssai_sd = 0xFFFFFF;
}

# RF simulator: the UE acts as the server for both DU clients
rfsimulator: {
  serveraddr = "server";    # keep as 'server' (nr-ue when started as --rfsim server)
  serverport = 4043;
  modelname = "AWGN";
  IQfile = "/tmp/rfsimulator.iqs";
}

```


```bash
sudo -E ./nr-softmodem -O /etc/oai/oai-cu.conf --sa --telnetsrv --telnetsrv.shrmod ci

sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3450720000 --rfsim -O /etc/oai/nr-ue.conf

sudo ./nr-softmodem -O /etc/oai/oai-du0.conf --rfsim --sa

sudo ./nr-softmodem -O /etc/oai/oai-du1.conf --rfsim --sa

echo ci trigger_f1_ho | nc 127.0.0.11 9090 && echo

```