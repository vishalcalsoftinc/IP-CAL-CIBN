

```cucp-service.yaml

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

```

```cuup-service.yaml

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
    - name: gtp-u
      port: 2152
      targetPort: 2152
      protocol: UDP
    - name: f1-u
      port: 2153
      targetPort: 2153
      protocol: UDP



```

```du-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: du-svc
  namespace: oai-test
  labels:
    app: oai-du
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
      port: 4043  # Default RF simulator port
      targetPort: 4043

```

```cucp-depl-cm.yaml
---

apiVersion: v1
kind: PersistentVolume
metadata:
  name: shared-xapp-libs
  namespace: oai-test
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  volumeMode: Filesystem
  local:
    path: /mnt/shared-xapp-libs/
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - ran


---

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: shared-xapp-libs
  namespace: oai-test
spec:
  storageClassName: "local-storage"
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
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
        image: oaisoftwarealliance/oai-gnb:develop
        securityContext:
          privileged: true
        env:
        - name: USE_ADDITIONAL_OPTIONS
          value: "--log_config.global_log_options level,nocolor,time --gNBs.[0].E1_INTERFACE.[0].ipv4_cucp 0.0.0.0 --gNBs.[0].local_s_address 0.0.0.0"
        - name: ASAN_OPTIONS
          value: detect_leaks=0
        volumeMounts:
        - mountPath: /opt/oai-gnb/etc/gnb.conf
          subPath: gnb.conf
          name: config-volume
        - name: shared-xapp-libs
          mountPath: /usr/local/lib/flexric/
        readinessProbe:
          exec:
            command: ["/bin/bash", "-c", "pgrep nr-softmodem"]
          initialDelaySeconds: 10
          periodSeconds: 10
        livenessProbe:
          exec:
            command: ["/bin/bash", "-c", "pgrep nr-softmodem"]
          initialDelaySeconds: 10
          periodSeconds: 30
      volumes:
      - name: config-volume
        configMap:
          name: oai-cucp-config
      - name: shared-xapp-libs
        persistentVolumeClaim:
          claimName: shared-xapp-libs
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: oai-cucp-config
  namespace: oai-test
data:
  gnb.conf: |
    Active_gNBs = ( "gNB-OAI");
    # Asn1_verbosity, choice in: none, info, annoying
    Asn1_verbosity = "debug";

    gNBs =
    (
     {
        ////////// Identification parameters:
        gNB_ID = 0xe00;

    #     cell_type =  "CELL_MACRO_GNB";

        gNB_name  =  "gNB-OAI";

        // Tracking area code, 0x0000 and 0xfffe are reserved values
        tracking_area_code  =  1;
        #plmn_list = ({ mcc = 999; mnc = 70; mnc_length = 2; snssaiList = ({ sd = 0x111111, sst = 1 }, { sd = 0x111112, sst = 2 }, { sd = 0x111113, sst = 3}, { sd = 0x01b207, sst = 1}) });
        plmn_list = ({ mcc = 999; mnc = 70; mnc_length = 2; snssaiList = ({ sst = 1 },  {sd = 0x000001, sst = 1 }, { sd = 0x000001, sst = 2 }, { sd = 0x000001, sst = 3}) });


        nr_cellid = 12345678L;

        tr_s_preference = "f1";

        local_s_address = "0.0.0.0";
        remote_s_address = "0.0.0.0"; # multiple DUs
        local_s_portc   = 501;
        local_s_portd   = 2153;
        remote_s_portc  = 500;
        remote_s_portd  = 2152;

        # ------- SCTP definitions
        SCTP :
        {
            # Number of streams to use in input/output
            SCTP_INSTREAMS  = 2;
            SCTP_OUTSTREAMS = 2;
        };


        ////////// AMF parameters:
        amf_ip_address = ({ ipv4 = "10.30.202.4"; });


        E1_INTERFACE =
        (
          {
            type = "cp";
            ipv4_cucp = "0.0.0.0";
            port_cucp = 38462;
            ipv4_cuup = "0.0.0.0"; # multiple CU-UPs
            port_cuup = 38462;
          }
        )

        NETWORK_INTERFACES :
        {
            GNB_IPV4_ADDRESS_FOR_NG_AMF              = "0.0.0.0/24";
        };
      }
    );

    security = {
      # preferred ciphering algorithms
      # the first one of the list that an UE supports in chosen
      # valid values: nea0, nea1, nea2, nea3
      ciphering_algorithms = ( "nea0" );

      # preferred integrity algorithms
      # the first one of the list that an UE supports in chosen
      # valid values: nia0, nia1, nia2, nia3
      integrity_algorithms = ( "nia2", "nia0" );

      # setting 'drb_ciphering' to "no" disables ciphering for DRBs, no matter
      # what 'ciphering_algorithms' configures; same thing for 'drb_integrity'
      drb_ciphering = "yes";
      drb_integrity = "no";
    };
    log_config : {
      global_log_level                      ="debug";
      hw_log_level                          ="debug";
      phy_log_level                         ="debug";
      mac_log_level                         ="debug";
      rlc_log_level                         ="debug";
      pdcp_log_level                        ="debug";
      rrc_log_level                         ="debug";
      f1ap_log_level                        ="debug";
    };

    #e2_agent = {
    #  near_ric_ip_addr = "10.12.0.191";
    #  e2AP_port = 36422;
    #  near_ri_port = 32222;
    #  #sm_dir = "/path/where/the/SMs/are/located/"
    #  sm_dir = "/usr/local/lib/flexric/"
    #};

```

```cuup-depl-cm.yaml
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
        image: oaisoftwarealliance/oai-nr-cuup:develop
        env:
        - name: USE_ADDITIONAL_OPTIONS
          value: "--log_config.global_log_options level,nocolor,time --gNBs.[0].E1_INTERFACE.[0].ipv4_cucp cucp-svc --gNBs.[0].E1_INTERFACE.[0].ipv4_cuup 0.0.0.0 --gNBs.[0].local_s_address 0.0.0.0 --gNBs.[0].remote_s_address 0.0.0.0"
        - name: ASAN_OPTIONS
          value: detect_leaks=0
        volumeMounts:
        - mountPath: /opt/oai-gnb/etc/gnb.conf
          subPath: gnb.conf
          name: config-volume
        readinessProbe:
          exec:
            command: ["/bin/bash", "-c", "pgrep nr-cuup"]
          initialDelaySeconds: 10
          periodSeconds: 10
        livenessProbe:
          exec:
            command: ["/bin/bash", "-c", "pgrep nr-cuup"]
          initialDelaySeconds: 10
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
    # Asn1_verbosity, choice in: none, info, annoying
    Asn1_verbosity = "debug";

    gNBs =
    (
     {
        ////////// Identification parameters:
        gNB_ID = 0xe00;
        gNB_CU_UP_ID = 0xe00;

    #     cell_type =  "CELL_MACRO_GNB";

        gNB_name  =  "gNB-OAI";

        // Tracking area code, 0x0000 and 0xfffe are reserved values
        tracking_area_code  =  1;
        #plmn_list = ({ mcc = 999; mnc = 70; mnc_length = 2; snssaiList =  ({ sd = 0x111111, sst = 1 }, { sd = 0x111112, sst = 2 }, { sd = 0x111113, sst = 3}, { sd = 0x01b207, sst = 1}) });
        plmn_list = ({ mcc = 999; mnc = 70; mnc_length = 2; snssaiList =  ({ sst = 1 }, { sd = 0x000001, sst = 1 }, { sd = 0x000001, sst = 2 }, { sd = 0x000001, sst = 3}) });


        tr_s_preference = "f1";

        local_s_address = "0.0.0.0";
        remote_s_address = "0.0.0.0";
        local_s_portc   = 501;
        local_s_portd   = 2153;
        remote_s_portc  = 500;
        remote_s_portd  = 2152;

        # ------- SCTP definitions
        SCTP :
        {
            # Number of streams to use in input/output
            SCTP_INSTREAMS  = 2;
            SCTP_OUTSTREAMS = 2;
        };

        E1_INTERFACE =
        (
          {
            type = "up";
            ipv4_cucp = "cucp-svc";
            port_cucp = 38462;
            ipv4_cuup = "0.0.0.0";
            port_cuup = 38462;
          }
        );

        NETWORK_INTERFACES :
        {
            GNB_IPV4_ADDRESS_FOR_NG_AMF              = "0.0.0.0/24";
            GNB_IPV4_ADDRESS_FOR_NGU                 = "0.0.0.0/24";
            GNB_PORT_FOR_NGU                         = 2152; # Spec 2152
        };
      }
    );

    log_config : {
      global_log_level = "debug";
      pdcp_log_level   = "debug";
      f1ap_log_level   = "debug";
      ngap_log_level   = "debug";
    };

    #e2_agent = {
    #  near_ric_ip_addr = "10.21.133.135";
    #  #sm_dir = "/path/where/the/SMs/are/located/"
    #  sm_dir = "/usr/local/lib/flexric/"
    #};

```

```du-depl-cm.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: oai-du
  namespace: oai-test
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
        # Equivalent to: ${REGISTRY:-oaisoftwarealliance}/${GNB_IMG:-oai-gnb}:${TAG:-develop}
        image: oaisoftwarealliance/oai-gnb:develop
        imagePullPolicy: IfNotPresent

        # Drop capabilities (like cap_drop: ALL in Docker Compose)
        
            
        securityContext:
          privileged: true
        env:
          - name: USE_ADDITIONAL_OPTIONS
            value: "--rfsim --log_config.global_log_options level,nocolor,time --MACRLCs.[0].local_n_address 0.0.0.0 --MACRLCs.[0].remote_n_address cucp-svc --MACRLCs.[0].local_n_address_f1u 0.0.0.0"
          - name: ASAN_OPTIONS
            value: "detect_leaks=0"

        # Health checks in K8s typically translate to liveness/readiness probes.
        livenessProbe:
          exec:
            command: ["/bin/bash", "-c", "pgrep nr-softmodem"]
          initialDelaySeconds: 10
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 5
        readinessProbe:
          exec:
            command: ["/bin/bash", "-c", "pgrep nr-softmodem"]
          initialDelaySeconds: 10
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 5

        # If you need to expose ports (not shown in snippet), add them here:
        # ports:
        #   - containerPort: 38472
        #     protocol: SCTP
        #   - containerPort: 9090
        #     protocol: TCP

        volumeMounts:
          - name: gnb-config
            mountPath: /opt/oai-gnb/etc/gnb.conf
            subPath: gnb.conf

      # Volumes definition
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
    # Asn1_verbosity, choice in: none, info, annoying
    Asn1_verbosity = "debug";

    gNBs =
    (
     {
        ////////// Identification parameters:
        gNB_ID = 0xe00;
        gNB_DU_ID = 0xe00;

    #     cell_type =  "CELL_MACRO_GNB";

        gNB_name  =  "gNB-OAI";

        // Tracking area code, 0x0000 and 0xfffe are reserved values
        tracking_area_code  =  1;
        #plmn_list = ({ mcc = 999; mnc = 70; mnc_length = 2; snssaiList =  ({ sd = 0x111111, sst = 1 }, { sd = 0x111112, sst = 2 }, { sd = 0x111113, sst = 3}, { sd = 0x01b207, sst = 1}) });
        plmn_list = ({ mcc = 999; mnc = 70; mnc_length = 2; snssaiList =  ({ sst = 1 }, { sd = 0x000001, sst = 1 }, { sd = 0x000001, sst = 2 }, { sd = 0x000001, sst = 3}) });


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
        local_n_address = "0.0.0.0";
        remote_n_address = "cuup-svc";
        local_n_portc   = 500;
        local_n_portd   = 2152;
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
      global_log_level = "debug";
      hw_log_level = "debug";
      phy_log_level = "debug";
      mac_log_level = "debug";
      rlc_log_level = "debug";
      f1ap_log_level = "debug";
    };

    #e2_agent = {
    #  near_ric_ip_addr = "10.0.2.81";
    #  #sm_dir = "/path/where/the/SMs/are/located/"
    #  sm_dir = "/usr/local/lib/flexric/"
    #};

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

```

```ue-depl-cm.yaml
---  
apiVersion: apps/v1
kind: Deployment
metadata:
  name: oai-nr-ue
  namespace: oai-test 
  labels:
    app: oai-nr-ue
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
          image: oaisoftwarealliance/oai-nr-ue:develop
          securityContext:
            privileged: true
          env:
            - name: USE_ADDITIONAL_OPTIONS
              value: "--ssb 516 --band 78 --rfsim -r 106 --numerology 1 -C 3619200000 --uicc0.imsi 999700000000010 --rfsimulator.serveraddr du-svc --log_config.global_log_options level,nocolor,time"
          volumeMounts:
            - name: nrue-conf
              mountPath: /opt/oai-nr-ue/etc/nr-ue.conf
              subPath: nrue.uicc.conf
          readinessProbe:
            exec:
              command:
                - /bin/bash
                - -c
                - "pgrep nr-uesoftmodem"
            initialDelaySeconds: 10
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 5
      volumes:
        - name: nrue-conf
          configMap:
            name: nrue-config
      restartPolicy: Always
      nodeSelector:
        kubernetes.io/os: linux
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nrue-config
  namespace: oai-test 
data:
  nrue.uicc.conf: |
    uicc0 = {
      imsi = "999700000000010";
      key = "465B5CE8B199B49FAA5F0A2EE238A6BC";
      opc= "E8ED289DEBA952E4283B54E88E6183CA";
      dnn= "internet";
      nssai_sst=1;
      #nssai_sd=0x000001;
    }

    position0 = {
        x = 0.0;
        y = 0.0;
        z = 6377900.0;
    }

    thread-pool = "-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1,-1"

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


```

