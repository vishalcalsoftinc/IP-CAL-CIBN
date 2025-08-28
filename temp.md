
# cu logs
```bash
telcomaan@oai-cu-vm:~$ cd ~/openairinterface5g/cmake_targets/ran_build/build
telcomaan@oai-cu-vm:~/openairinterface5g/cmake_targets/ran_build/build$ sudo -E ./nr-softmodem -O /etc/oai/oai-cu.conf --sa --telnetsrv --telnetsrv.shrmod ci
CMDLINE: "./nr-softmodem" "-O" "/etc/oai/oai-cu.conf" "--sa" "--telnetsrv" "--telnetsrv.shrmod" "ci" 
[CONFIG] function config_libconfig_init returned 0
[LOADER] library libtelnetsrv_gnb.so is not loaded: libtelnetsrv_gnb.so: cannot open shared object file: No such file or directory
[TELNETSRV] Telnet server: module 0 = telnet added to shell
policy set to other, priority 0
Error 3: No such process trying to get nice value of thread 3711956672 
[TELNETSRV] 
Initializing telnet server...
[TELNETSRV] Telnet server: module 1 = softmodem added to shell
[TELNETSRV] couldn't find add_phy_cmds for module phy 
[TELNETSRV] Telnet server: module 2 = loader added to shell
[TELNETSRV] Telnet server: module 3 = measur added to shell
[TELNETSRV] Telnet server: module 4 = ci added to shell
[UTIL]   running in SA mode (no --phy-test, --do-ra, --nsa option present)
[OPT]   OPT disabled
[HW]   Version: Branch: develop Abrev. Hash: b763692792 Date: Fri Aug 1 04:40:24 2025 +0000
[GNB_APP]   Initialized RAN Context: RC.nb_nr_inst = 1, RC.nb_nr_macrlc_inst = 0, RC.nb_nr_L1_inst = 0, RC.nb_RU = 0, RC.nb_nr_CC[0] = 0
[GNB_APP]   F1AP: gNB_CU_id[0] 3584
[GNB_APP]   F1AP: gNB_CU_name[0] gNB-Eurecom-CU
[GNB_APP]   SDAP layer is disabled
[GNB_APP]   Data Radio Bearer count 1
[GNB_APP]   Parsed IPv4 address for NG AMF: 172.17.0.93
[UTIL]   threadCreate() for TASK_SCTP: creating thread with affinity ffffffff, priority 50
[X2AP]   X2AP is disabled.
[UTIL]   threadCreate() for TASK_NGAP: creating thread with affinity ffffffff, priority 50
[NGAP]   Starting NGAP layer
[UTIL]   threadCreate() for TASK_RRC_GNB: creating thread with affinity ffffffff, priority 50
[NGAP]   Registered new gNB[0] and macro gNB id 3584
[NGAP]   [gNB 0] check the amf registration state
[NR_RRC]   Entering main loop of NR_RRC message task
[GTPU]   Configuring GTPu
[GTPU]   SA mode 
[GTPU]   Configuring GTPu address : 172.17.0.93, port : 2152
[GTPU]   Initializing UDP for local address 172.17.0.93 with port 2152
[GTPU]   Created gtpu instance id: 95
[NR_RRC]   Accepting new CU-UP ID 3584 name gNB-Eurecom-CU (assoc_id -1)
[UTIL]   threadCreate() for TASK_GNB_APP: creating thread with affinity ffffffff, priority 50
[UTIL]   threadCreate() for TASK_CU_F1: creating thread with affinity ffffffff, priority 50
[F1AP]   Starting F1AP at CU
[F1AP]   F1AP_CU_SCTP_REQ(create socket) for 172.17.0.93 len 12
[UTIL]   threadCreate() for TASK_GTPV1_U: creating thread with affinity ffffffff, priority 50
[GTPU]   Initializing UDP for local address 172.17.0.93 with port 2153
[GTPU]   Created gtpu instance id: 96
[UTIL]   threadCreate() for time source realtime: creating thread with affinity ffffffff, priority 2
[UTIL]   time manager configuration: [time source: reatime] [mode: standalone] [server IP: 127.0.0.1} [server port: 7374] (server IP/port not used)
[NGAP]   Send NGSetupRequest to AMF
[NGAP]   3584 -> 0000e000
TYPE <CTRL-C> TO TERMINATE
[NGAP]   Served GUAMIs for AMF (no name) (assoc_id=3):
[NGAP]    GUAMI:
[NGAP]      PLMN: MCC=999, MNC=70
[NGAP]      AMF Region ID: 2
[NGAP]      AMF Set ID: 1
[NGAP]      AMF Pointer: 0
[NGAP]   Supported PLMN 0: MCC=999 MNC=70
[NGAP]   Supported slice (PLMN 0): SST=0x01 SD=000
[NGAP]   Received NGSetupResponse from AMF
[GNB_APP]   [gNB 0] Received NGAP_REGISTER_GNB_CNF: associated AMF 1
[F1AP]   CU Task Received SCTP_NEW_ASSOCIATION_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   CU Task Received SCTP_NEW_ASSOCIATION_RESP for instance 0: sending SCTP message via assoc_id 0
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[F1AP]   CU_handle_F1_SETUP_REQUEST
[NR_RRC]   Received F1 Setup Request from gNB_DU 3585 (oai-cu-cp) on assoc_id 4
[NR_RRC]   Accepting DU 3585 (oai-cu-cp), sending F1 Setup Response
[NR_RRC]   DU uses RRC version 17.3.0
[F1AP]   CU Task Received F1AP_SETUP_RESP for instance 0: sending SCTP message via assoc_id 4
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[F1AP]   [SCTP 4] CU_handle_gNB_DU_CONFIGURATION_UPDATE
[F1AP]   Sending F1AP_GNB_DU_CONFIGURATION_UPDATE ITTI message 
[NR_RRC]   cell PLMN 999.70 Cell ID 12345678 is in service
[F1AP]   CU Task Received F1AP_GNB_DU_CONFIGURATION_UPDATE_ACKNOWLEDGE for instance 0: sending SCTP message via assoc_id 4
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[NR_RRC]   Decoding CCCH: RNTI da06, payload_size 6
[NR_RRC]   [--] (cellID 0, UE ID 1 RNTI da06) Create UE context: CU UE ID 1 DU UE ID 55814 (rnti: da06, random ue id 142395bb77000000)
[RRC]   activate SRB 1 of UE 1
[NR_RRC]   [DL] (cellID bc614e, UE ID 1 RNTI da06) Send RRC Setup
[F1AP]   CU Task Received F1AP_DL_RRC_MESSAGE for instance 0: sending SCTP message via assoc_id 4
[F1AP]   CU send DL_RRC_MESSAGE_TRANSFER 
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[F1AP]   CU_handle_UL_RRC_MESSAGE_TRANSFER 
[F1AP]   UL RRC MESSAGE for SRB 1 in DCCH 
[F1AP]   [UE 1] calling pdcp_data_ind for SRB 1 with size 39 (DCCH) 
[NR_RRC]   [UL] (cellID bc614e, UE ID 1 RNTI da06) Received RRCSetupComplete (RRC_CONNECTED reached)
[NGAP]   Selected PLMN in the NG Initial UE Message: MCC 999, MNC 70
[NGAP]   UE 1: Chose AMF 'open5gs-amf0' (assoc_id 3) through selected PLMN MCC=999 MNC=70
[NGAP]   Create UE context (ID 1) for AMF 'open5gs-amf0' (assoc_id 3)
[NR_RRC]   [DL] (cellID bc614e, UE ID 1 RNTI da06) Send DL Information Transfer [42 bytes]
[F1AP]   CU Task Received F1AP_DL_RRC_MESSAGE for instance 0: sending SCTP message via assoc_id 4
[F1AP]   CU send DL_RRC_MESSAGE_TRANSFER 
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[F1AP]   CU_handle_UL_RRC_MESSAGE_TRANSFER 
[F1AP]   UL RRC MESSAGE for SRB 1 in DCCH 
[F1AP]   [UE 1] calling pdcp_data_ind for SRB 1 with size 30 (DCCH) 
[NR_RRC]   [UL] (cellID bc614e, UE ID 1 RNTI da06) Received RRC UL Information Transfer [24 bytes]
[NR_RRC]   [DL] (cellID bc614e, UE ID 1 RNTI da06) Send DL Information Transfer [19 bytes]
[F1AP]   CU Task Received F1AP_DL_RRC_MESSAGE for instance 0: sending SCTP message via assoc_id 4
[F1AP]   CU send DL_RRC_MESSAGE_TRANSFER 
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[F1AP]   CU_handle_UL_RRC_MESSAGE_TRANSFER 
[F1AP]   UL RRC MESSAGE for SRB 1 in DCCH 
[F1AP]   [UE 1] calling pdcp_data_ind for SRB 1 with size 66 (DCCH) 
[NR_RRC]   [UL] (cellID bc614e, UE ID 1 RNTI da06) Received RRC UL Information Transfer [60 bytes]
[NGAP]   NGAP_FIND_PROTOCOLIE_BY_ID ie is NULL (searching for ie: 110)
[NGAP]   NGAP_FIND_PROTOCOLIE_BY_ID ie is NULL (searching for ie: 71)
[NGAP]   AllowedNSSAI.list.count 1
[NGAP]   NGAP_FIND_PROTOCOLIE_BY_ID ie is NULL (searching for ie: 36)
[NR_RRC]   [--] (cellID bc614e, UE ID 1 RNTI da06) Selected security algorithms: ciphering 0, integrity 2
[NR_RRC]   [UE da06] Saved security key B7
[NR_RRC]   UE 1 Logical Channel DL-DCCH, Generate SecurityModeCommand (bytes 3)
[F1AP]   CU Task Received F1AP_DL_RRC_MESSAGE for instance 0: sending SCTP message via assoc_id 4
[F1AP]   CU send DL_RRC_MESSAGE_TRANSFER 
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[F1AP]   CU_handle_UL_RRC_MESSAGE_TRANSFER 
[F1AP]   UL RRC MESSAGE for SRB 1 in DCCH 
[F1AP]   [UE 1] calling pdcp_data_ind for SRB 1 with size 8 (DCCH) 
[NR_RRC]   [UL] (cellID bc614e, UE ID 1 RNTI da06) Received Security Mode Complete
[NR_RRC]   UE 1: Logical Channel DL-DCCH, Generate NR UECapabilityEnquiry (bytes 8, xid 1)
[F1AP]   CU Task Received F1AP_DL_RRC_MESSAGE for instance 0: sending SCTP message via assoc_id 4
[F1AP]   CU send DL_RRC_MESSAGE_TRANSFER 
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[F1AP]   CU_handle_UL_RRC_MESSAGE_TRANSFER 
[F1AP]   UL RRC MESSAGE for SRB 1 in DCCH 
[F1AP]   [UE 1] calling pdcp_data_ind for SRB 1 with size 20 (DCCH) 
[NR_RRC]   [UL] (cellID bc614e, UE ID 1 RNTI da06) Received UE capabilities
[NR_RRC]   Send message to ngap: NGAP_UE_CAPABILITIES_IND
[NR_RRC]   [DL] (cellID bc614e, UE ID 1 RNTI da06) Send DL Information Transfer [46 bytes]
[NR_RRC]   Send message to sctp: NGAP_InitialContextSetupResponse
[F1AP]   CU Task Received F1AP_DL_RRC_MESSAGE for instance 0: sending SCTP message via assoc_id 4
[F1AP]   CU send DL_RRC_MESSAGE_TRANSFER 
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[F1AP]   CU_handle_UL_RRC_MESSAGE_TRANSFER 
[F1AP]   UL RRC MESSAGE for SRB 1 in DCCH 
[F1AP]   [UE 1] calling pdcp_data_ind for SRB 1 with size 19 (DCCH) 
[NR_RRC]   [UL] (cellID bc614e, UE ID 1 RNTI da06) Received RRC UL Information Transfer [13 bytes]
[NR_RRC]   [DL] (cellID bc614e, UE ID 1 RNTI da06) Send DL Information Transfer [51 bytes]
[F1AP]   CU Task Received F1AP_DL_RRC_MESSAGE for instance 0: sending SCTP message via assoc_id 4
[F1AP]   CU send DL_RRC_MESSAGE_TRANSFER 
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[F1AP]   CU_handle_UL_RRC_MESSAGE_TRANSFER 
[F1AP]   UL RRC MESSAGE for SRB 1 in DCCH 
[F1AP]   [UE 1] calling pdcp_data_ind for SRB 1 with size 46 (DCCH) 
[NR_RRC]   [UL] (cellID bc614e, UE ID 1 RNTI da06) Received RRC UL Information Transfer [40 bytes]
[NGAP]   PDUSESSIONSetup initiating message
[NR_RRC]   UE 1: received PDU Session Resource Setup Request
[NR_RRC]   Bearer Context Setup: PDU Session ID=10, incoming TEID=0x00009551, Addr=172.17.0.95
[NR_RRC]   UE 1: configure DRB ID 1 for PDU session ID 10
[NR_RRC]   [--] (cellID bc614e, UE ID 1 RNTI da06) second best match: CU-UP ID 3584 matches SST 1
[NR_RRC]   [--] (cellID bc614e, UE ID 1 RNTI da06) selecting CU-UP ID 3584 based on exact NSSAI match (1:0xffffff)
[RRC]   UE 1 associating to CU-UP assoc_id -1 out of 1 CU-UPs
[E1AP]   UE 1: add PDU session ID 10 (1 bearers)
[GTPU]   [95] Created tunnel for UE ID 1, teid for incoming: 61a82d8a, teid for outgoing 9551 to remote IPv4: 172.17.0.95, IPv6 ::
[PDCP]   added drb 1 to UE ID 1
[SDAP]   Default DRB for the created SDAP entity: 1 
[GTPU]   [96] Created tunnel for UE ID 1, teid for incoming: 90e1d19e, teid for outgoing ffff to remote IPv4: 0.0.0.0, IPv6 ::
[RRC]   activate SRB 2 of UE 1
[RRC]   UE 1 trigger UE context setup request with 1 DRBs
[F1AP]   CU Task Received F1AP_UE_CONTEXT_SETUP_REQ for instance 0: sending SCTP message via assoc_id 4
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[RRC]   UE da06 replacing existing CellGroupConfig with new one received from DU
[E1AP]   UE 1: updating PDU session ID 10 (1 bearers)
[PDCP]   DRB 1 re-established
[GTPU]   [96] Tunnel Outgoing TEID updated to 2e0e6155 and address to 5c0011ac
[NR_RRC]   [DL] (cellID bc614e, UE ID 1 RNTI da06) Generate RRCReconfiguration (bytes 283, xid 0)
[F1AP]   CU Task Received F1AP_DL_RRC_MESSAGE for instance 0: sending SCTP message via assoc_id 4
[RRC]   UE 1: PDU session ID 10 modified 1 bearers
[F1AP]   CU send DL_RRC_MESSAGE_TRANSFER 
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[F1AP]   CU_handle_UL_RRC_MESSAGE_TRANSFER 
[F1AP]   UL RRC MESSAGE for SRB 1 in DCCH 
[F1AP]   [UE 1] calling pdcp_data_ind for SRB 1 with size 8 (DCCH) 
[NR_RRC]   [UL] (cellID bc614e, UE ID 1 RNTI da06) Received RRCReconfigurationComplete
[NR_RRC]   PDU Session Setup Response: ID=10, outgoing TEID=0x61a82d8a, Addr=172.17.0.93
[NR_RRC]   NGAP_PDUSESSION_SETUP_RESP: sending the message
[F1AP]   CU Task Received F1AP_UE_CONTEXT_MODIFICATION_REQ for instance 0: sending SCTP message via assoc_id 4
[NGAP]   Encoded PDU Session Transfer (10): TEID=0x61a82d8a, Addr=172.17.0.93
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[F1AP]   CU Task Received SCTP_NEW_ASSOCIATION_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   CU Task Received SCTP_NEW_ASSOCIATION_RESP for instance 0: sending SCTP message via assoc_id 0
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[F1AP]   CU_handle_F1_SETUP_REQUEST
[NR_RRC]   Received F1 Setup Request from gNB_DU 3586 (oai-cu-cp) on assoc_id 5
[NR_RRC]   Accepting DU 3586 (oai-cu-cp), sending F1 Setup Response
[NR_RRC]   DU uses RRC version 17.3.0
[F1AP]   CU Task Received F1AP_SETUP_RESP for instance 0: sending SCTP message via assoc_id 5
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[F1AP]   [SCTP 5] CU_handle_gNB_DU_CONFIGURATION_UPDATE
[F1AP]   Sending F1AP_GNB_DU_CONFIGURATION_UPDATE ITTI message 
[NR_RRC]   cell PLMN 999.70 Cell ID 12345679 is in service
[F1AP]   CU Task Received F1AP_GNB_DU_CONFIGURATION_UPDATE_ACKNOWLEDGE for instance 0: sending SCTP message via assoc_id 5
[TELNETSRV] Telnet client connected....
[TELNETSRV] Command received: readc 17 filled 17 "ci trigger_f1_ho"
[NR_RRC]   Handover triggered for UE 1/RNTI da06 towards DU 3586/assoc_id 5/PCI 1
[F1AP]   CU Task Received F1AP_UE_CONTEXT_SETUP_REQ for instance 0: sending SCTP message via assoc_id 5
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[RRC]   UE da06 replacing existing CellGroupConfig with new one received from DU
[E1AP]   UE 1: updating PDU session ID 10 (1 bearers)
[PDCP]   DRB 1 re-established
[GTPU]   [96] Tunnel Outgoing TEID updated to 77fddcba and address to 5e0011ac
[NR_RRC]   HO acknowledged: Send reconfiguration for UE 1/RNTI da06...
[F1AP]   CU Task Received F1AP_UE_CONTEXT_MODIFICATION_REQ for instance 0: sending SCTP message via assoc_id 4
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[PDCP]   SRB 2 re-established
[RRC]   UE 1: PDU session ID 10 modified 1 bearers
[NR_RRC]   UE 1 handover: update RNTI from da06 to 324d
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[F1AP]   CU_handle_UL_RRC_MESSAGE_TRANSFER 
[F1AP]   UL RRC MESSAGE for SRB 1 in DCCH 
[F1AP]   [UE 1] calling pdcp_data_ind for SRB 1 with size 8 (DCCH) 
[NR_RRC]   [UL] (cellID bc614f, UE ID 1 RNTI 324d) Received RRCReconfigurationComplete
[NR_RRC]   handover for UE 1/RNTI 324d complete!
[NR_RRC]   UE 1 Handover: trigger release on DU assoc_id 4
[F1AP]   CU Task Received F1AP_UE_CONTEXT_RELEASE_CMD for instance 0: sending SCTP message via assoc_id 4
[F1AP]   CU Task Received F1AP_UE_CONTEXT_MODIFICATION_REQ for instance 0: sending SCTP message via assoc_id 5
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[F1AP]   CU_handle_UL_RRC_MESSAGE_TRANSFER 
[F1AP]   UL RRC MESSAGE for SRB 2 in DCCH 
[F1AP]   [UE 1] calling pdcp_data_ind for SRB 2 with size 33 (DCCH) 
[NR_RRC]   [UL] (cellID bc614f, UE ID 1 RNTI 324d) Received RRC UL Information Transfer [27 bytes]
[NR_RRC]   [DL] (cellID bc614f, UE ID 1 RNTI 324d) Send DL Information Transfer [10 bytes]
[E1AP]   releasing UE 1
[GTPU]   [95] Deleted all tunnels for ue id 1 (1 tunnels deleted)
[GTPU]   [96] Deleted all tunnels for ue id 1 (1 tunnels deleted)
[F1AP]   CU Task Received F1AP_DL_RRC_MESSAGE for instance 0: sending SCTP message via assoc_id 5
[NR_RRC]   [DL] (cellID bc614f, UE ID 1 RNTI 324d) Send RRC Release
[RRC]   UE 1: received bearer release complete
[F1AP]   CU send DL_RRC_MESSAGE_TRANSFER 
[F1AP]   CU Task Received F1AP_UE_CONTEXT_RELEASE_CMD for instance 0: sending SCTP message via assoc_id 5
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[NR_RRC]   removed UE CU UE ID 1/RNTI 324d 
[NR_RRC]   [--] (cellID bc614f, UE ID 1 RNTI 324d) Remove UE context
[SCTP]   Received SCTP SHUTDOWN EVENT
[F1AP]   CU Task Received SCTP_NEW_ASSOCIATION_RESP for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Received SCTP state 1 for assoc_id 5, removing endpoint
[NR_RRC]   releasing DU ID 3586 (oai-cu-cp) on assoc_id 5
[SCTP]   Received SCTP SHUTDOWN EVENT
[F1AP]   CU Task Received SCTP_NEW_ASSOCIATION_RESP for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Received SCTP state 1 for assoc_id 4, removing endpoint
[NR_RRC]   releasing DU ID 3585 (oai-cu-cp) on assoc_id 4
^C
** Caught SIGTERM, shutting down
Returned from ITTI signal handler
Bye.

```


# ue logs 
```bash


telcomaan@oai-nr-ue-vm:~/openairinterface5g/cmake_targets/ran_build/build$ sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --ssb 516 --rfsim -O /etc/oai/nr-ue.conf
CMDLINE: "./nr-uesoftmodem" "-r" "106" "--numerology" "1" "--band" "78" "-C" "3619200000" "--ssb" "516" "--rfsim" "-O" "/etc/oai/nr-ue.conf" 
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
[OPT]   OPT disabled
[HW]   Version: Branch: develop Abrev. Hash: 61c769f8b5 Date: Wed Jul 23 15:13:27 2025 +0000
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
[PHY]   SA init parameters. DL freq 3619200000 UL offset 0 SSB numerology 1 N_RB_DL 106
[PHY]   Init: N_RB_DL 106, first_carrier_offset 1412, nb_prefix_samples 144,nb_prefix_samples0 176, ofdm_symbol_size 2048
[PHY]   samples_per_subframe 61440/per second 61440000, wCP 57344
[UTIL]   threadCreate() for SYNC__actor: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for DL__actor: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for DL__actor: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for DL__actor: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for DL__actor: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for UL__actor: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for UL__actor: creating thread with affinity ffffffff, priority 97
[PHY]   Initializing UE vars for gNB TXant 1, UE RXant 1
[PHY]   prs_config configuration NOT found..!! Skipped configuring UE for the PRS reception
[PHY]   HW: Configuring card 0, sample_rate 61440000.000000, tx/rx num_channels 1/1, duplex_mode TDD
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_freq 3619200000 Hz, rx_freq 3619200000 Hz, tune_offset 0
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_gain 0, rx_gain 110
[PHY]   HW: Configuring card 1, sample_rate 61440000.000000, tx/rx num_channels 1/1, duplex_mode TDD
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_freq 3619200000 Hz, rx_freq 3619200000 Hz, tune_offset 0
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_gain 0, rx_gain 110
[PHY]   HW: Configuring card 2, sample_rate 61440000.000000, tx/rx num_channels 1/1, duplex_mode TDD
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_freq 3619200000 Hz, rx_freq 3619200000 Hz, tune_offset 0
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_gain 0, rx_gain 110
[PHY]   HW: Configuring card 3, sample_rate 61440000.000000, tx/rx num_channels 1/1, duplex_mode TDD
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_freq 3619200000 Hz, rx_freq 3619200000 Hz, tune_offset 0
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_gain 0, rx_gain 110
[PHY]   HW: Configuring card 4, sample_rate 61440000.000000, tx/rx num_channels 1/1, duplex_mode TDD
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_freq 3619200000 Hz, rx_freq 3619200000 Hz, tune_offset 0
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_gain 0, rx_gain 110
[PHY]   HW: Configuring card 5, sample_rate 61440000.000000, tx/rx num_channels 1/1, duplex_mode TDD
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_freq 3619200000 Hz, rx_freq 3619200000 Hz, tune_offset 0
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_gain 0, rx_gain 110
[PHY]   HW: Configuring card 6, sample_rate 61440000.000000, tx/rx num_channels 1/1, duplex_mode TDD
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_freq 3619200000 Hz, rx_freq 3619200000 Hz, tune_offset 0
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_gain 0, rx_gain 110
[PHY]   HW: Configuring card 7, sample_rate 61440000.000000, tx/rx num_channels 1/1, duplex_mode TDD
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_freq 3619200000 Hz, rx_freq 3619200000 Hz, tune_offset 0
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_gain 0, rx_gain 110
[PHY]   Intializing UE Threads for instance 0 ...
[UTIL]   threadCreate() for UEthread_0: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for L1_UE_stats_0: creating thread with affinity ffffffff, priority 1
UE threads created by 4275
TYPE <CTRL-C> TO TERMINATE
[HW]   Running as server waiting opposite rfsimulators to connect
Initializing random number generator, seed 1074693117782859641
[HW]   [RRU] has loaded RFSIMULATOR device.
[HW]   No connected device, generating void samples...
Entering ITTI signals handler
TYPE <CTRL-C> TO TERMINATE
[PHY]   SSB position provided
[NR_PHY]   Starting sync detection
[PHY]   [UE thread Synch] Running Initial Synch 
[NR_PHY]   Starting cell search with center freq: 3619200000, bandwidth: 106. Scanning for 1 number of GSCN.
[NR_PHY]   Scanning GSCN: 0, with SSB offset: 516, SSB Freq: 0.000000
[PHY]   synch Failed: 
[PHY]   SSB position provided
[NR_PHY]   Starting sync detection
[PHY]   [UE thread Synch] Running Initial Synch 
[NR_PHY]   Starting cell search with center freq: 3619200000, bandwidth: 106. Scanning for 1 number of GSCN.
[NR_PHY]   Scanning GSCN: 0, with SSB offset: 516, SSB Freq: 0.000000
[PHY]   synch Failed: 
[PHY]   SSB position provided
[NR_PHY]   Starting sync detection
[PHY]   [UE thread Synch] Running Initial Synch 
[NR_PHY]   Starting cell search with center freq: 3619200000, bandwidth: 106. Scanning for 1 number of GSCN.
[NR_PHY]   Scanning GSCN: 0, with SSB offset: 516, SSB Freq: 0.000000
[PHY]   Initial sync: pbch decoded sucessfully, ssb index 0
[PHY]   pbch rx ok. rsrp:51 dB/RE, adjust_rxgain:-1 dB
[NR_PHY]   Cell Detected with GSCN: 0, SSB SC offset: 516, SSB Ref: 0.000000, PSS Corr peak: 99 dB, PSS Corr Average: 61
[PHY]   [UE0] In synch, rx_offset 30720 samples
[PHY]   [UE 0] Measured Carrier Frequency offset 7 Hz
[PHY]   Initial sync successful, PCI: 0
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_freq 3619200007 Hz, rx_freq 3619200007 Hz, tune_offset 0
[PHY]   Got synch: hw_slot_offset 2, carrier off 7 Hz, rxgain 0.000000 (DL 3619200007.000000 Hz, UL 3619200007.000000 Hz)
[PHY]   UE synchronized! decoded_frame_rx=610 UE->init_sync_frame=1 trashed_frames=56
[PHY]   Resynchronizing RX by 30720 samples
[HW]   received write reorder clear context
[NR_RRC]   SIB1 decoded
[NR_MAC]   TDD period index = 6, based on the sum of dl_UL_TransmissionPeriodicity from Pattern1 (5.000000 ms) and Pattern2 (0.000000 ms): Total = 5.000000 ms
[NR_MAC]   Set TDD configuration period to: 8 DL slots, 3 UL slots, 10 slots per period (NR_TDD_UL_DL_Pattern is 7 DL slots, 2 UL slots, 6 DL symbols, 4 UL symbols)
[NR_MAC]   Configured 1 TDD patterns (total slots: pattern1 = 10, pattern2 = 0)
[PHY]   N_TA_offset changed from 0 to 800
[MAC]   Initialization of 4-Step CBRA procedure
[NR_MAC]   PRACH scheduler: Selected RO Frame 669, Slot 19, Symbol 0, Fdm 0
[PHY]   PRACH [UE 0] in frame.slot 669.19, placing PRACH in position 2828, Msg1/MsgA-Preamble frequency start 0 (k1 0), preamble_offset 8, first_nonzero_root_idx 0, preambleIndex = 34
[PHY]   RAR-Msg2 decoded
[NR_MAC]   [UE 0][RAPROC][RA-RNTI 010b] Got BI RAR subPDU 5 ms
[NR_MAC]   [UE 0][RAPROC][RA-RNTI 010b] Got RAPID RAR subPDU
[NR_MAC]   [UE 0][RAPROC][670.7] Found RAR with the intended RAPID 34
[MAC]   received TA command 31
[NR_MAC]   [RAPROC][670.17] RA-Msg3 transmitted
[MAC]   [UE 0][671.11][RAPROC] 4-Step RA procedure succeeded. CBRA: Contention Resolution is successful.
[NR_RRC]   [UE0][RAPROC] Logical Channel DL-CCCH (SRB0), Received NR_RRCSetup
[RLC]   Added srb 1 to UE 0
[NR_RRC]   State = NR_RRC_CONNECTED
[NAS]   Generate Initial NAS Message: Registration Request
[NAS]   [UE 0] Received NR_NAS_CONN_ESTABLISH_IND: asCause 0
[NR_RRC]   [UE 0][RAPROC] Logical Channel UL-DCCH (SRB1), Generating RRCSetupComplete (bytes33)
[MAC]   [UE 0] Applying CellGroupConfig from gNodeB
[NR_MAC]   UE 0 RNTI da06 stats sfn: 768.8, cumulated bad DCI 0
    DL harq: 1/0
    Ul harq: 119/0 avg code rate 0.2, avg bit/symbol 2.5, avg per TB: (nb RBs 5.8, nb symbols 3.0)
[NAS]   [UE 0] Received NAS_DOWNLINK_DATA_IND type FGS_AUTHENTICATION_REQUEST with length 42
kausf:11 42 34 f9 96 3c 92 7b 74 3a 1f 4b 26 8e ca 45 da 24 eb 53 29 a7 dc 57 eb 4c 65 3b f bc cc 64 
kseaf:39 98 39 b2 7c 6 bc c 3f 31 c0 de 94 77 8d 98 44 7d fe 35 fb cf 48 b8 db fe 7a 7d 63 b2 4a ef 
kamf:19 61 d 39 ff 64 33 9f 4f 83 dd f6 87 c4 3 7d a4 76 8d 73 3e b1 91 a2 c7 22 23 9f 49 33 96 82 
[NAS]   [UE 0] Received NAS_DOWNLINK_DATA_IND type FGS_SECURITY_MODE_COMMAND with length 19
knas_int: 9a 89 57 a6 d4 2c c5 18 3f ad 8d 8a 54 29 1f 1f 
knas_enc: bc f 66 4f c e9 2c 5c 99 d8 ed a4 e4 45 1 3 
[NAS]   Generate Initial NAS Message: Registration Request
mac 5d cd 32 b 
[NR_RRC]   Received securityModeCommand (gNB 0)
[NR_RRC]   Receiving from SRB1 (DL-DCCH), Processing securityModeCommand
[NR_RRC]   Security algorithm is set to nea0
[NR_RRC]   Integrity protection algorithm is set to nia2
[NR_RRC]   deriving kRRCenc, kRRCint from KgNB=b7 64 dd 87 f2 12 f1 c8 d8 80 2f cd ad 6a a4 fe 65 92 f6 2e d1 95 38 2f 0b 4f 68 63 c9 0c 10 e4 
[NR_RRC]   Receiving from SRB1 (DL-DCCH), encoding securityModeComplete, rrc_TransactionIdentifier: 0
[NR_RRC]   securityModeComplete payload: 28 00 00 00 00 00 00 00 b0 4e 00 a0 ac 71 00 00 
[NR_RRC]   Received Capability Enquiry (gNB 0)
[NR_RRC]   Receiving from SRB1 (DL-DCCH), Processing UECapabilityEnquiry
<UE-NR-Capability>
    <accessStratumRelease><rel15/></accessStratumRelease>
    <pdcp-Parameters>
        <supportedROHC-Profiles>
            <profile0x0000><false/></profile0x0000>
            <profile0x0001><false/></profile0x0001>
            <profile0x0002><false/></profile0x0002>
            <profile0x0003><false/></profile0x0003>
            <profile0x0004><false/></profile0x0004>
            <profile0x0006><false/></profile0x0006>
            <profile0x0101><false/></profile0x0101>
            <profile0x0102><false/></profile0x0102>
            <profile0x0103><false/></profile0x0103>
            <profile0x0104><false/></profile0x0104>
        </supportedROHC-Profiles>
        <maxNumberROHC-ContextSessions><cs2/></maxNumberROHC-ContextSessions>
    </pdcp-Parameters>
    <phy-Parameters>
    </phy-Parameters>
    <rf-Parameters>
        <supportedBandListNR>
            <BandNR>
                <bandNR>1</bandNR>
            </BandNR>
        </supportedBandListNR>
    </rf-Parameters>
</UE-NR-Capability>
[PHY]   [RRC]UE NR Capability encoded, 10 bytes (86 bits)
[NR_RRC]   UECapabilityInformation Encoded 106 bits (14 bytes)
[NAS]   [UE 0] Received NAS_DOWNLINK_DATA_IND type FGS_REGISTRATION_ACCEPT with length 46
[NAS]   Received Registration Accept with result 3GPP
[NAS]   SMS not allowed in 5GS Registration Result
[NR_RRC]   5G-GUTI: AMF pointer 0, AMF Set ID 1, 5G-TMSI 3221226295 
mac 89 a a cb 
[NAS]   Send NAS_UPLINK_DATA_REQ message(RegistrationComplete)
mac b3 2f 9e 43 
[NAS]   Send NAS_UPLINK_DATA_REQ message(PduSessionEstablishRequest)
[NAS]   [UE 0] Received NAS_DOWNLINK_DATA_IND type FGS_CONFIGURATION_UPDATE_COMMAND with length 51
[NR_RRC]   unknown message type 84
[NR_RRC]   RRCReconfiguration includes radio Bearer Configuration
[PDCP]   added drb 1 to UE ID 0
[SDAP]   Default DRB for the created SDAP entity: 1 
[NR_RRC]   State = NR_RRC_CONNECTED
[RLC]   Added srb 2 to UE 0
[RLC]   Added drb 1 to UE 0
[RLC]   Added DRB to UE 0
[NR_RRC]   RRCReconfiguration includes Measurement Configuration
[NR_RRC]   Measurement gaps not yet supported!
[NR_RRC]   rrcReconfigurationComplete Encoded 10 bits (2 bytes)
[NR_RRC]    Logical Channel UL-DCCH (SRB1), Generating RRCReconfigurationComplete (bytes 2)
[MAC]   [UE 0] Applying CellGroupConfig from gNodeB
[NAS]   [UE 0] Received NAS_CONN_ESTABLI_CNF: errCode 1, length 68
[NAS]   Received PDU Session Establishment Accept, UE IPv4: 10.45.0.2
[OIP]   Interface oaitun_ue1 successfully configured, IPv4 10.45.0.2, IPv6 (null)
[UTIL]   threadCreate() for ue_tun_read_0_p10: creating thread with affinity ffffffff, priority 1
[NR_MAC]   UE 0 RNTI da06 stats sfn: 896.8, cumulated bad DCI 0
    DL harq: 18/0
    Ul harq: 174/0 avg code rate 0.3, avg bit/symbol 2.2, avg per TB: (nb RBs 6.0, nb symbols 3.5)
[NR_MAC]   UE 0 RNTI da06 stats sfn: 0.8, cumulated bad DCI 0
    DL harq: 19/0
    Ul harq: 186/0 avg code rate 0.3, avg bit/symbol 2.2, avg per TB: (nb RBs 5.9, nb symbols 4.1)
    .....
    [NR_RRC]   RRCReconfiguration includes radio Bearer Configuration
[PDCP]   SRB 2 re-established
[PDCP]   DRB 1 re-established
[NR_RRC]   State = NR_RRC_CONNECTED
[NR_RRC]   Processing reconfigurationWithSync
[NR_RRC]   RRCReconfiguration includes Measurement Configuration
[NR_RRC]   Measurement gaps not yet supported!
[NR_RRC]   rrcReconfigurationComplete Encoded 10 bits (2 bytes)
[NR_RRC]    Logical Channel UL-DCCH (SRB1), Generating RRCReconfigurationComplete (bytes 2)
[MAC]   [UE 0] Applying CellGroupConfig from gNodeB
[NR_MAC]   Received reconfigurationWithSync
[NR_MAC]   Configuring CRNTI 324d
[NR_MAC]   TDD period index = 6, based on the sum of dl_UL_TransmissionPeriodicity from Pattern1 (5.000000 ms) and Pattern2 (0.000000 ms): Total = 5.000000 ms
[NR_MAC]   Set TDD configuration period to: 8 DL slots, 3 UL slots, 10 slots per period (NR_TDD_UL_DL_Pattern is 7 DL slots, 2 UL slots, 6 DL symbols, 4 UL symbols)
[NR_MAC]   Configured 1 TDD patterns (total slots: pattern1 = 10, pattern2 = 0)
[PHY]   SSB position provided
[NR_PHY]   Starting re-sync detection for target Nid_cell 1
[PHY]   [UE thread Synch] Running Initial Synch 
[NR_PHY]   Starting cell search with center freq: 3619200000, bandwidth: 106. Scanning for 1 number of GSCN.
[NR_PHY]   Scanning GSCN: 0, with SSB offset: 516, SSB Freq: 0.000000
[PHY]   Initial sync: pbch decoded sucessfully, ssb index 0
[PHY]   pbch rx ok. rsrp:54 dB/RE, adjust_rxgain:-4 dB
[NR_PHY]   Cell Detected with GSCN: 0, SSB SC offset: 516, SSB Ref: 0.000000, PSS Corr peak: 99 dB, PSS Corr Average: 64
[PHY]   [UE0] In synch, rx_offset 365616 samples
[PHY]   [UE 0] Measured Carrier Frequency offset -76 Hz
[PHY]   Initial sync successful, PCI: 1
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_freq 3619199924 Hz, rx_freq 3619199924 Hz, tune_offset 0
[PHY]   Got synch: hw_slot_offset 24, carrier off -76 Hz, rxgain 0.000000 (DL 3619199924.000000 Hz, UL 3619199924.000000 Hz)
[PHY]   UE synchronized! decoded_frame_rx=344 UE->init_sync_frame=1 trashed_frames=6
[PHY]   Resynchronizing RX by 365616 samples
[HW]   received write reorder clear context
[MAC]   Initialization of 4-Step CFRA procedure
[NR_MAC]   PRACH scheduler: Selected RO Frame 353, Slot 19, Symbol 0, Fdm 0
[PHY]   PRACH [UE 0] in frame.slot 353.19, placing PRACH in position 2828, Msg1/MsgA-Preamble frequency start 0 (k1 0), preamble_offset 15, first_nonzero_root_idx 0, preambleIndex = 63
[PHY]   RAR-Msg2 decoded
[NR_MAC]   [UE 0][RAPROC][RA-RNTI 010b] Got BI RAR subPDU 5 ms
[NR_MAC]   [UE 0][RAPROC][RA-RNTI 010b] Got RAPID RAR subPDU
[NR_MAC]   [UE 0][RAPROC][354.7] Found RAR with the intended RAPID 63
[MAC]   received TA command 31
[MAC]   [UE 0][354.7][RAPROC] RA procedure succeeded. CFRA: RAR successfully received.
[NR_MAC]   UE 0 RNTI 324d stats sfn: 384.8, cumulated bad DCI 0
    DL harq: 202/0
    Ul harq: 2343/0 avg code rate 0.1, avg bit/symbol 2.0, avg per TB: (nb RBs 5.5, nb symbols 10.1)
[NR_MAC]   UE 0 RNTI 324d stats sfn: 512.8, cumulated bad DCI 0
    DL harq: 203/0
    Ul harq: 2451/0 avg code rate 0.1, avg bit/symbol 2.0, avg per TB: (nb RBs 5.5, nb symbols 9.8)
    ....
    ^C[NAS]   [UE 0] Received NAS_DEREGISTRATION_REQ
mac 41 2b d2 d2 
Press ^C again to trigger immediate shutdown
[NR_RRC]   [UE 0] Received RRC Release (gNB 0)
[NAS]   [UE 0] Received NAS_DOWNLINK_DATA_IND type FGS_DEREGISTRATION_ACCEPT_UE_ORIGINATING with length 10
[NAS]   received deregistration accept
[NR_RRC]   deprioritisationReq in RRCRelease not handled
[PDCP]   release DRB 1 of UE 0
[ITTI]   Queue for TASK_RRC_NRUE task size: 22 (last message NR_RRC_MAC_BCCH_DATA_IND)
[ITTI]   Queue for TASK_RRC_NRUE task size: 28 (last message NRRRC_FRAME_PROCESS)
[ITTI]   Queue for TASK_RRC_NRUE task size: 35 (last message NRRRC_FRAME_PROCESS)
[ITTI]   Queue for TASK_RRC_NRUE task size: 44 (last message NRRRC_FRAME_PROCESS)
[ITTI]   Queue for TASK_RRC_NRUE task size: 55 (last message NRRRC_FRAME_PROCESS)
[ITTI]   Queue for TASK_RRC_NRUE task size: 69 (last message NR_RRC_MAC_SYNC_IND)
[NR_MAC]   [UE 0] SR not served! SR counter 64 reached sr_MaxTransmissions 64
[NR_MAC]   Triggering new RA procedure for UE with RNTI 324d
[MAC]   Initialization of 4-Step CBRA procedure
[NR_MAC]   PRACH scheduler: Selected RO Frame 563, Slot 19, Symbol 0, Fdm 0
[PHY]   PRACH [UE 0] in frame.slot 563.19, placing PRACH in position 2828, Msg1/MsgA-Preamble frequency start 0 (k1 0), preamble_offset 5, first_nonzero_root_idx 0, preambleIndex = 20
[PHY]   RAR-Msg2 decoded
[NR_MAC]   [UE 0][RAPROC][RA-RNTI 010b] Got BI RAR subPDU 5 ms
[NR_MAC]   [UE 0][RAPROC][RA-RNTI 010b] Got RAPID RAR subPDU
[NR_MAC]   [UE 0][RAPROC][564.7] Found RAR with the intended RAPID 20
[NR_MAC]   [RAPROC][564.17] RA-Msg3 transmitted
[ITTI]   Queue for TASK_RRC_NRUE task size: 87 (last message NRRRC_FRAME_PROCESS)
[MAC]   [UE 0] Contention resolution failed
[NR_MAC]   PRACH scheduler: Selected RO Frame 571, Slot 19, Symbol 0, Fdm 0
[PHY]   PRACH [UE 0] in frame.slot 571.19, placing PRACH in position 2828, Msg1/MsgA-Preamble frequency start 0 (k1 0), preamble_offset 13, first_nonzero_root_idx 0, preambleIndex = 52
[PHY]   RAR-Msg2 decoded
[NR_MAC]   [UE 0][RAPROC][RA-RNTI 010b] Got BI RAR subPDU 5 ms
[NR_MAC]   [UE 0][RAPROC][RA-RNTI 010b] Got RAPID RAR subPDU
[NR_MAC]   [UE 0][RAPROC][572.7] Found RAR with the intended RAPID 52
[NR_MAC]   [RAPROC][572.17] RA-Msg3 transmitted
[MAC]   [UE 0] Contention resolution failed
[ITTI]   Queue for TASK_RRC_NRUE task size: 109 (last message NR_RRC_MAC_MSG3_IND)
[NR_MAC]   PRACH scheduler: Selected RO Frame 579, Slot 19, Symbol 8, Fdm 0
[PHY]   PRACH [UE 0] in frame.slot 579.19, placing PRACH in position 2828, Msg1/MsgA-Preamble frequency start 0 (k1 0), preamble_offset 10, first_nonzero_root_idx 0, preambleIndex = 43
[PHY]   RAR-Msg2 decoded
[NR_MAC]   [UE 0][RAPROC][RA-RNTI 0113] Got BI RAR subPDU 5 ms
[NR_MAC]   [UE 0][RAPROC][RA-RNTI 0113] Got RAPID RAR subPDU
[NR_MAC]   [UE 0][RAPROC][580.7] Found RAR with the intended RAPID 43
[NR_MAC]   [RAPROC][580.17] RA-Msg3 transmitted
[MAC]   [UE 0] Contention resolution failed
[NR_MAC]   PRACH scheduler: Selected RO Frame 587, Slot 19, Symbol 4, Fdm 0
[PHY]   PRACH [UE 0] in frame.slot 587.19, placing PRACH in position 2828, Msg1/MsgA-Preamble frequency start 0 (k1 0), preamble_offset 0, first_nonzero_root_idx 0, preambleIndex = 1
[PHY]   RAR-Msg2 decoded
[NR_MAC]   [UE 0][RAPROC][RA-RNTI 010f] Got BI RAR subPDU 5 ms
[NR_MAC]   [UE 0][RAPROC][RA-RNTI 010f] Got RAPID RAR subPDU
[NR_MAC]   [UE 0][RAPROC][588.7] Found RAR with the intended RAPID 1
[NR_MAC]   [RAPROC][588.17] RA-Msg3 transmitted
[ITTI]   Queue for TASK_RRC_NRUE task size: 137 (last message NR_RRC_MAC_BCCH_DATA_IND)
[TMR]   Queue for TASK_RRC_NRUE task contains 214 messages
[MAC]   [UE 0] Contention resolution failed
[NR_MAC]   PRACH scheduler: Selected RO Frame 635, Slot 19, Symbol 0, Fdm 0
[PHY]   PRACH [UE 0] in frame.slot 635.19, placing PRACH in position 2828, Msg1/MsgA-Preamble frequency start 0 (k1 0), preamble_offset 10, first_nonzero_root_idx 0, preambleIndex = 41
[PHY]   RAR-Msg2 decoded
[NR_MAC]   [UE 0][RAPROC][RA-RNTI 010b] Got BI RAR subPDU 5 ms
[NR_MAC]   [UE 0][RAPROC][RA-RNTI 010b] Got RAPID RAR subPDU
[NR_MAC]   [UE 0][RAPROC][636.7] Found RAR with the intended RAPID 41
[NR_MAC]   [RAPROC][636.17] RA-Msg3 transmitted
[MAC]   [UE 0] Contention resolution failed
^CReturned from ITTI signal handler
oai_exit=1
```

# du0 logs
```bash

telcomaan@oai-du0-vm:~/openairinterface5g/cmake_targets/ran_build/build$ sudo ./nr-softmodem -O /etc/oai/oai-du.conf --rfsim --sa
CMDLINE: "./nr-softmodem" "-O" "/etc/oai/oai-du.conf" "--rfsim" "--sa" 
[CONFIG] function config_libconfig_init returned 0
[UTIL]   running in SA mode (no --phy-test, --do-ra, --nsa option present)
[OPT]   OPT disabled
[HW]   Version: Branch: develop Abrev. Hash: b763692792 Date: Fri Aug 1 04:40:24 2025 +0000
[GNB_APP]   Initialized RAN Context: RC.nb_nr_inst = 1, RC.nb_nr_macrlc_inst = 1, RC.nb_nr_L1_inst = 1, RC.nb_RU = 1, RC.nb_nr_CC[0] = 1
[NR_PHY]   Initializing gNB RAN context: RC.nb_nr_L1_inst = 1 
[NR_PHY]   Registered with MAC interface module (0x5990f2725850)
[NR_PHY]   Initializing NR L1: RC.nb_nr_L1_inst = 1
[NR_PHY]   L1_RX_THREAD_CORE -1 (15)
[NR_PHY]   TX_AMP = 519 (-36 dBFS)
[PHY]   No prs_config configuration found..!!
[GNB_APP]   pdsch_AntennaPorts N1 1 N2 1 XP 1 pusch_AntennaPorts 1
[GNB_APP]   minTXRXTIME 6
[GNB_APP]   SIB1 TDA 1
[GNB_APP]   CSI-RS 0, SRS 0, SINR:0, 256 QAM may be on, delta_MCS off, maxMIMO_Layers -1, HARQ feedback enabled, num DLHARQ:16, num ULHARQ:16
[NR_MAC]   No RedCap configuration found
[GNB_APP]   sr_ProhibitTimer 0, sr_TransMax 64, sr_ProhibitTimer_v1700 0, t300 400, t301 400, t310 2000, n310 10, t311 3000, n311 1, t319 400
[NR_MAC]   Candidates per PDCCH aggregation level on UESS: L1: 0, L2: 2, L4: 0, L8: 0, L16: 0
[RRC]   Read in ServingCellConfigCommon (PhysCellId 0, ABSFREQSSB 641280, DLBand 78, ABSFREQPOINTA 640008, DLBW 106,RACH_TargetReceivedPower -96
[RRC]   absoluteFrequencySSB 641280 corresponds to 3619200000 Hz
[NR_MAC]   TDD period index = 6, based on the sum of dl_UL_TransmissionPeriodicity from Pattern1 (5.000000 ms) and Pattern2 (0.000000 ms): Total = 5.000000 ms
[UTIL]   threadCreate() for MAC_STATS: creating thread with affinity ffffffff, priority 2
[NR_MAC]   PUSCH Target 200, PUCCH Target 200, PUCCH Failure 10, PUSCH Failure 10
[NR_PHY]   Copying 0 blacklisted PRB to L1 context
[NR_MAC]   Set TX antenna number to 1, Set RX antenna number to 1 (num ssb 1: 80000000,0)
[NR_MAC]   TDD period index = 6, based on the sum of dl_UL_TransmissionPeriodicity from Pattern1 (5.000000 ms) and Pattern2 (0.000000 ms): Total = 5.000000 ms
[NR_MAC]   Set TDD configuration period to: 8 DL slots, 3 UL slots, 10 slots per period (NR_TDD_UL_DL_Pattern is 7 DL slots, 2 UL slots, 6 DL symbols, 4 UL symbols)
[NR_MAC]   Configured 1 TDD patterns (total slots: pattern1 = 10, pattern2 = 0)
[NR_PHY]   Set TDD Period Configuration: 2 periods per frame, 20 slots to be configured (8 DL, 3 UL)
[NR_PHY]   TDD period configuration: slot 0 is DOWNLINK
[NR_PHY]   TDD period configuration: slot 1 is DOWNLINK
[NR_PHY]   TDD period configuration: slot 2 is DOWNLINK
[NR_PHY]   TDD period configuration: slot 3 is DOWNLINK
[NR_PHY]   TDD period configuration: slot 4 is DOWNLINK
[NR_PHY]   TDD period configuration: slot 5 is DOWNLINK
[NR_PHY]   TDD period configuration: slot 6 is DOWNLINK
[NR_PHY]   TDD period configuration: slot 7 is FLEXIBLE: DDDDDDFFFFUUUU
[NR_PHY]   TDD period configuration: slot 8 is UPLINK
[NR_PHY]   TDD period configuration: slot 9 is UPLINK
DL frequency 3619200000: band 48, UL frequency 3619200000
[PHY]   DL frequency 3619200000 Hz, UL frequency 3619200000 Hz: band 48, uldl offset 0 Hz
[PHY]   Initializing frame parms for mu 1, N_RB 106, Ncp 0
[PHY]   Init: N_RB_DL 106, first_carrier_offset 1412, nb_prefix_samples 144,nb_prefix_samples0 176, ofdm_symbol_size 2048
[NR_RRC]   SIB1 freq: offsetToPointA 86
[GNB_APP]   F1AP: gNB idx 0 gNB_DU_id 3585, gNB_DU_name oai-cu-cp, TAC 1 MCC/MNC/length 999/70/2 cellID 12345678
[GNB_APP]   ngran_DU: Configuring Cell 0 for TDD
[GNB_APP]   SDAP layer is disabled
[GNB_APP]   Data Radio Bearer count 1
[UTIL]   threadCreate() for TASK_SCTP: creating thread with affinity ffffffff, priority 50
[X2AP]   X2AP is disabled.
[UTIL]   threadCreate() for TASK_GNB_APP: creating thread with affinity ffffffff, priority 50
[UTIL]   threadCreate() for TASK_DU_F1: creating thread with affinity ffffffff, priority 50
[F1AP]   Starting F1AP at DU
[F1AP]   F1-C DU IPaddr 172.17.0.92, connect to F1-C CU 172.17.0.93, binding GTP to 172.17.0.92
[UTIL]   threadCreate() for TASK_GTPV1_U: creating thread with affinity ffffffff, priority 50
[GTPU]   Initializing UDP for local address 172.17.0.92 with port 2153
[GTPU]   Created gtpu instance id: 94
[UTIL]   threadCreate() for time source iq samples: creating thread with affinity ffffffff, priority 2
[MAC]   received F1 Setup Response from CU gNB-Eurecom-CU
[MAC]   CU uses RRC version 17.3.0
[MAC]   Clearing the DU's UE states before, if any.
[MAC]   received gNB-DU configuration update acknowledge
[UTIL]   time manager configuration: [time source: iq_samples] [mode: standalone] [server IP: 127.0.0.1} [server port: 7374] (server IP/port not used)
[PHY]   RU clock source set as internal
[PHY]   number of L1 instances 1, number of RU 1, number of CPU cores 2
[PHY]   Initialized RU proc 0 (,synch_to_ext_device),
[PHY]   RU thread-pool core string -1,-1 (size 2)
[UTIL]   threadCreate() for Tpool0_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool1_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for ru_thread: creating thread with affinity ffffffff, priority 97
[PHY]   Starting RU 0 (,synch_to_ext_device) on cpu 0
[PHY]   Initializing frame parms for mu 1, N_RB 106, Ncp 0
[PHY]   Init: N_RB_DL 106, first_carrier_offset 1412, nb_prefix_samples 144,nb_prefix_samples0 176, ofdm_symbol_size 2048
[PHY]   fp->scs=30000
[PHY]   fp->ofdm_symbol_size=2048
[PHY]   fp->nb_prefix_samples0=176
[PHY]   fp->nb_prefix_samples=144
[PHY]   fp->slots_per_subframe=2
[PHY]   fp->samples_per_subframe_wCP=57344
[PHY]   fp->samples_per_frame_wCP=573440
[PHY]   fp->samples_per_subframe=61440
[PHY]   fp->samples_per_frame=614400
[PHY]   fp->dl_CarrierFreq=3619200000
[PHY]   fp->ul_CarrierFreq=3619200000
[PHY]   fp->Nid_cell=0
[PHY]   fp->first_carrier_offset=1412
[PHY]   fp->ssb_start_subcarrier=0
[PHY]   fp->Ncp=0
[PHY]   fp->N_RB_DL=106
[PHY]   fp->numerology_index=1
[PHY]   fp->nr_band=48
[PHY]   fp->ofdm_offset_divisor=8
[PHY]   fp->threequarter_fs=0
[PHY]   fp->sl_CarrierFreq=0
[PHY]   fp->N_RB_SL=0
[NR_PHY]   nb_tx_streams 1, nb_rx_streams 1, num_Beams_period 1
[PHY]   Setting RF config for N_RB 106, NB_RX 1, NB_TX 1
[PHY]   tune_offset 0 Hz, sample_rate 61440000 Hz
[PHY]   Channel 0: setting tx_gain offset 0, tx_freq 3619200000 Hz
[PHY]   Channel 0: setting rx_gain offset 114, rx_freq 3619200000 Hz
[HW]   Running as client: will connect to a rfsimulator server side
Initializing random number generator, seed 4739850180686543628
[HW]   [RAU] has loaded RFSIMULATOR device.
[PHY]   RU 0 Setting N_TA_offset to 800 samples (UL Freq 3600120, N_RB 106, mu 1)
[PHY]   Signaling main thread that RU 0 is ready, sl_ahead 6
[PHY]   L1 configured without analog beamforming
[PHY]   Attaching RU 0 antenna 0 to gNB antenna 0
[UTIL]   threadCreate() for Tpool0_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool1_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool2_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool3_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool4_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool5_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool6_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool7_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for L1_rx_thread: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for L1_tx_thread: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for L1_stats: creating thread with affinity ffffffff, priority 1
[PHY]   got sync (L1_stats_thread)
[PHY]   got sync (ru_thread)
[HW]   Trying to connect to 172.17.0.91:4043
TYPE <CTRL-C> TO TERMINATE
[HW]   Connection to 172.17.0.91:4043 established
[PHY]   RU 0 rf device ready
[PHY]   RU 0 RF started cpu_meas_enabled 0
[PHY]   Command line parameters for OAI UE: -C 3619200000 -r 106 --numerology 1 --ssb 516 
[NR_MAC]   Frame.Slot 384.0

[NR_PHY]   [RAPROC] 407.19 Initiating RA procedure with preamble 14, energy 43.4 dB (I0 0, thres 200), delay 0 start symbol 0 freq index 0
[NR_MAC]   407.19 UE RA-RNTI 010b TC-RNTI 9deb: initiating RA procedure
[NR_MAC]   UE 9deb: Msg3 scheduled at 408.17 (408.7 TDA 3) start 0 RBs 8
[NR_MAC]   UE 9deb: 408.7 Generating RA-Msg2 DCI, RA RNTI 0x10b, state 1, preamble_index(RAPID) 14, timing_offset = 0 (estimated distance 0.0 [m])
[NR_MAC]   408.7 Send RAR to RA-RNTI 010b
[NR_MAC]    408.17 PUSCH with TC_RNTI 0x9deb received correctly
[MAC]   [RAPROC] Received SDU for CCCH length 6 for UE 9deb
[RLC]   Activated srb0 for UE 40427
[RLC]   Added srb 1 to UE 40427
[NR_MAC]   Activating scheduling Msg4 for TC_RNTI 0x9deb (state WAIT_Msg3)
[NR_MAC]   UE 9deb Generate Msg4: feedback at  409.19, payload 161 bytes, next state nrRA_WAIT_Msg4_MsgB_ACK
[NR_MAC]    409.19 UE 9deb: Received Ack of Msg4. CBRA procedure succeeded (UE Connected)
[NR_MAC]   Adding new UE context with RNTI 0x9deb
[NR_MAC]   Frame.Slot 512.0
UE RNTI 9deb CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (12 meas)
UE 9deb: dlsch_rounds 2/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.09000 MCS (0) 0
UE 9deb: ulsch_rounds 129/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.03487 MCS (0) 4 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 50.5 dB
UE 9deb: MAC:    TX             17 RX            627 bytes
UE 9deb: LCID 1: TX              0 RX             40 bytes

[RLC]   Added srb 2 to UE 40427
[RLC]   Added drb 1 to UE 40427
[RLC]   Added DRB to UE 40427
[GTPU]   [94] Created tunnel for UE ID 40427, teid for incoming: 758e1a16, teid for outgoing 9a803e77 to remote IPv4: 172.17.0.93, IPv6 ::
[NR_MAC]   DU received confirmation of successful RRC Reconfiguration
[NR_MAC]   Frame.Slot 640.0
UE RNTI 9deb CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 9deb: dlsch_rounds 18/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.04783 MCS (0) 0
UE 9deb: ulsch_rounds 180/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.01094 MCS (0) 3 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 9deb: MAC:    TX            783 RX           1965 bytes
UE 9deb: LCID 1: TX            545 RX            349 bytes
UE 9deb: LCID 2: TX              0 RX              0 bytes
UE 9deb: LCID 4: TX              3 RX             64 bytes
.....

[NR_MAC]   gNB-DU received the TransmissionActionIndicator with Stop value for UE 9deb
[NR_PHY]   [RAPROC] 175.19 Initiating RA procedure with preamble 60, energy 23.7 dB (I0 0, thres 200), delay 20 start symbol 0 freq index 0
[NR_PHY]   [RAPROC] 175.19 Initiating RA procedure with preamble 61, energy 42.9 dB (I0 28, thres 200), delay 15 start symbol 4 freq index 0
[NR_MAC]   175.19 UE RA-RNTI 010b TC-RNTI 1c79: initiating RA procedure
[NR_MAC]   175.19 UE RA-RNTI 010f TC-RNTI 459d: initiating RA procedure
[NR_MAC]   UE 1c79: Msg3 scheduled at 176.17 (176.7 TDA 3) start 0 RBs 8
[NR_MAC]   UE 1c79: 176.7 Generating RA-Msg2 DCI, RA RNTI 0x10b, state 1, preamble_index(RAPID) 60, timing_offset = 20 (estimated distance 781.3 [m])
[NR_MAC]   176.7 Send RAR to RA-RNTI 010b
[NR_MAC]   UE 459d: Msg3 scheduled at 176.17 (176.7 TDA 3) start 8 RBs 8
[NR_MAC]   UE 459d: 176.7 Generating RA-Msg2 DCI, RA RNTI 0x10f, state 1, preamble_index(RAPID) 61, timing_offset = 15 (estimated distance 586.0 [m])
[NR_MAC]   176.7 Send RAR to RA-RNTI 010f
[NR_MAC]    17710: RA RNTI 1c79 CC_id 0 Scheduling retransmission of Msg3 in (177,17)
[NR_MAC]    17710: RA RNTI 459d CC_id 0 Scheduling retransmission of Msg3 in (177,17)
[NR_MAC]    17810: RA RNTI 1c79 CC_id 0 Scheduling retransmission of Msg3 in (178,17)
[NR_MAC]    17810: RA RNTI 459d CC_id 0 Scheduling retransmission of Msg3 in (178,17)
[NR_MAC]    17910: RA RNTI 1c79 CC_id 0 Scheduling retransmission of Msg3 in (179,17)
[NR_MAC]    17910: RA RNTI 459d CC_id 0 Scheduling retransmission of Msg3 in (179,17)
[NR_MAC]   UE 1c79 RA failed at state WAIT_Msg3 (Reached msg3 max harq rounds)
[NR_MAC]   Remove NR rnti 0x1c79
[NR_MAC]   UE 459d RA failed at state WAIT_Msg3 (Reached msg3 max harq rounds)
[NR_MAC]   Remove NR rnti 0x459d
[NR_MAC]   Frame.Slot 256.0
UE RNTI 9deb CU-UE-ID 1 out-of-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (5 meas)
UE 9deb: dlsch_rounds 90/1/1/0, dlsch_errors 0, pucch0_DTX 3, BLER 0.00002 MCS (0) 0
UE 9deb: ulsch_rounds 1113/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 9deb: MAC:    TX           2615 RX          17680 bytes
UE 9deb: LCID 1: TX            867 RX            349 bytes
UE 9deb: LCID 2: TX              0 RX              0 bytes
UE 9deb: LCID 4: TX             15 RX            320 bytes

[GTPU]   [94] Deleted all tunnels for ue id 40427 (1 tunnels deleted)
[RLC]   Remove UE 40427
[NR_MAC]   Remove NR rnti 0x9deb
[GTPU]   try to get a gtp-u not existing output
[NR_MAC]   Frame.Slot 384.0

[HW]   Lost socket

```


# du1 logs
```bash

telcomaan@oai-du1-vm:~/openairinterface5g/cmake_targets/ran_build/build$ sudo ./nr-softmodem -O /etc/oai/oai-du-ho.conf --rfsim --sa
CMDLINE: "./nr-softmodem" "-O" "/etc/oai/oai-du-ho.conf" "--rfsim" "--sa" 
[CONFIG] function config_libconfig_init returned 0
[UTIL]   running in SA mode (no --phy-test, --do-ra, --nsa option present)
[OPT]   OPT disabled
[HW]   Version: Branch: develop Abrev. Hash: b763692792 Date: Fri Aug 1 04:40:24 2025 +0000
[GNB_APP]   Initialized RAN Context: RC.nb_nr_inst = 1, RC.nb_nr_macrlc_inst = 1, RC.nb_nr_L1_inst = 1, RC.nb_RU = 1, RC.nb_nr_CC[0] = 1
[NR_PHY]   Initializing gNB RAN context: RC.nb_nr_L1_inst = 1 
[NR_PHY]   Registered with MAC interface module (0x5d920c1e0850)
[NR_PHY]   Initializing NR L1: RC.nb_nr_L1_inst = 1
[NR_PHY]   L1_RX_THREAD_CORE -1 (15)
[NR_PHY]   TX_AMP = 519 (-36 dBFS)
[PHY]   No prs_config configuration found..!!
[GNB_APP]   pdsch_AntennaPorts N1 1 N2 1 XP 1 pusch_AntennaPorts 1
[GNB_APP]   minTXRXTIME 6
[GNB_APP]   SIB1 TDA 1
[GNB_APP]   CSI-RS 0, SRS 0, SINR:0, 256 QAM may be on, delta_MCS off, maxMIMO_Layers -1, HARQ feedback enabled, num DLHARQ:16, num ULHARQ:16
[NR_MAC]   No RedCap configuration found
[GNB_APP]   sr_ProhibitTimer 0, sr_TransMax 64, sr_ProhibitTimer_v1700 0, t300 400, t301 400, t310 2000, n310 10, t311 3000, n311 1, t319 400
[NR_MAC]   Candidates per PDCCH aggregation level on UESS: L1: 0, L2: 2, L4: 0, L8: 0, L16: 0
[RRC]   Read in ServingCellConfigCommon (PhysCellId 1, ABSFREQSSB 641280, DLBand 78, ABSFREQPOINTA 640008, DLBW 106,RACH_TargetReceivedPower -96
[RRC]   absoluteFrequencySSB 641280 corresponds to 3619200000 Hz
[NR_MAC]   TDD period index = 6, based on the sum of dl_UL_TransmissionPeriodicity from Pattern1 (5.000000 ms) and Pattern2 (0.000000 ms): Total = 5.000000 ms
[UTIL]   threadCreate() for MAC_STATS: creating thread with affinity ffffffff, priority 2
[NR_MAC]   PUSCH Target 200, PUCCH Target 200, PUCCH Failure 10, PUSCH Failure 10
[NR_PHY]   Copying 0 blacklisted PRB to L1 context
[NR_MAC]   Set TX antenna number to 1, Set RX antenna number to 1 (num ssb 1: 80000000,0)
[NR_MAC]   TDD period index = 6, based on the sum of dl_UL_TransmissionPeriodicity from Pattern1 (5.000000 ms) and Pattern2 (0.000000 ms): Total = 5.000000 ms
[NR_MAC]   Set TDD configuration period to: 8 DL slots, 3 UL slots, 10 slots per period (NR_TDD_UL_DL_Pattern is 7 DL slots, 2 UL slots, 6 DL symbols, 4 UL symbols)
[NR_MAC]   Configured 1 TDD patterns (total slots: pattern1 = 10, pattern2 = 0)
[NR_PHY]   Set TDD Period Configuration: 2 periods per frame, 20 slots to be configured (8 DL, 3 UL)
[NR_PHY]   TDD period configuration: slot 0 is DOWNLINK
[NR_PHY]   TDD period configuration: slot 1 is DOWNLINK
[NR_PHY]   TDD period configuration: slot 2 is DOWNLINK
[NR_PHY]   TDD period configuration: slot 3 is DOWNLINK
[NR_PHY]   TDD period configuration: slot 4 is DOWNLINK
[NR_PHY]   TDD period configuration: slot 5 is DOWNLINK
[NR_PHY]   TDD period configuration: slot 6 is DOWNLINK
[NR_PHY]   TDD period configuration: slot 7 is FLEXIBLE: DDDDDDFFFFUUUU
[NR_PHY]   TDD period configuration: slot 8 is UPLINK
[NR_PHY]   TDD period configuration: slot 9 is UPLINK
DL frequency 3619200000: band 48, UL frequency 3619200000
[PHY]   DL frequency 3619200000 Hz, UL frequency 3619200000 Hz: band 48, uldl offset 0 Hz
[PHY]   Initializing frame parms for mu 1, N_RB 106, Ncp 0
[PHY]   Init: N_RB_DL 106, first_carrier_offset 1412, nb_prefix_samples 144,nb_prefix_samples0 176, ofdm_symbol_size 2048
[NR_RRC]   SIB1 freq: offsetToPointA 86
[GNB_APP]   F1AP: gNB idx 0 gNB_DU_id 3586, gNB_DU_name oai-cu-cp, TAC 1 MCC/MNC/length 999/70/2 cellID 12345679
[GNB_APP]   ngran_DU: Configuring Cell 0 for TDD
[GNB_APP]   SDAP layer is disabled
[GNB_APP]   Data Radio Bearer count 1
[UTIL]   threadCreate() for TASK_SCTP: creating thread with affinity ffffffff, priority 50
[X2AP]   X2AP is disabled.
[UTIL]   threadCreate() for TASK_GNB_APP: creating thread with affinity ffffffff, priority 50
[UTIL]   threadCreate() for TASK_DU_F1: creating thread with affinity ffffffff, priority 50
[F1AP]   Starting F1AP at DU
[F1AP]   F1-C DU IPaddr 172.17.0.94, connect to F1-C CU 172.17.0.93, binding GTP to 172.17.0.94
[UTIL]   threadCreate() for TASK_GTPV1_U: creating thread with affinity ffffffff, priority 50
[GTPU]   Initializing UDP for local address 172.17.0.94 with port 2153
[GTPU]   Created gtpu instance id: 94
[UTIL]   threadCreate() for time source iq samples: creating thread with affinity ffffffff, priority 2
[UTIL]   time manager configuration: [time source: iq_samples] [mode: standalone] [server IP: 127.0.0.1} [server port: 7374] (server IP/port not used)
[MAC]   received F1 Setup Response from CU gNB-Eurecom-CU
[MAC]   CU uses RRC version 17.3.0
[MAC]   Clearing the DU's UE states before, if any.
[MAC]   received gNB-DU configuration update acknowledge
[PHY]   RU clock source set as internal
[PHY]   number of L1 instances 1, number of RU 1, number of CPU cores 2
[PHY]   Initialized RU proc 0 (,synch_to_ext_device),
[PHY]   RU thread-pool core string -1,-1 (size 2)
[UTIL]   threadCreate() for Tpool0_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool1_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for ru_thread: creating thread with affinity ffffffff, priority 97
[PHY]   Starting RU 0 (,synch_to_ext_device) on cpu 1
[PHY]   Initializing frame parms for mu 1, N_RB 106, Ncp 0
[PHY]   Init: N_RB_DL 106, first_carrier_offset 1412, nb_prefix_samples 144,nb_prefix_samples0 176, ofdm_symbol_size 2048
[PHY]   fp->scs=30000
[PHY]   fp->ofdm_symbol_size=2048
[PHY]   fp->nb_prefix_samples0=176
[PHY]   fp->nb_prefix_samples=144
[PHY]   fp->slots_per_subframe=2
[PHY]   fp->samples_per_subframe_wCP=57344
[PHY]   fp->samples_per_frame_wCP=573440
[PHY]   fp->samples_per_subframe=61440
[PHY]   fp->samples_per_frame=614400
[PHY]   fp->dl_CarrierFreq=3619200000
[PHY]   fp->ul_CarrierFreq=3619200000
[PHY]   fp->Nid_cell=0
[PHY]   fp->first_carrier_offset=1412
[PHY]   fp->ssb_start_subcarrier=0
[PHY]   fp->Ncp=0
[PHY]   fp->N_RB_DL=106
[PHY]   fp->numerology_index=1
[PHY]   fp->nr_band=48
[PHY]   fp->ofdm_offset_divisor=8
[PHY]   fp->threequarter_fs=0
[PHY]   fp->sl_CarrierFreq=0
[PHY]   fp->N_RB_SL=0
[NR_PHY]   nb_tx_streams 1, nb_rx_streams 1, num_Beams_period 1
[PHY]   Setting RF config for N_RB 106, NB_RX 1, NB_TX 1
[PHY]   tune_offset 0 Hz, sample_rate 61440000 Hz
[PHY]   Channel 0: setting tx_gain offset 0, tx_freq 3619200000 Hz
[PHY]   Channel 0: setting rx_gain offset 114, rx_freq 3619200000 Hz
[HW]   Running as client: will connect to a rfsimulator server side
Initializing random number generator, seed 3107349158065691043
[HW]   [RAU] has loaded RFSIMULATOR device.
[PHY]   RU 0 Setting N_TA_offset to 800 samples (UL Freq 3600120, N_RB 106, mu 1)
[PHY]   Signaling main thread that RU 0 is ready, sl_ahead 6
[PHY]   L1 configured without analog beamforming
[PHY]   Attaching RU 0 antenna 0 to gNB antenna 0
[UTIL]   threadCreate() for Tpool0_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool1_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool2_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool3_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool4_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool5_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool6_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool7_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for L1_rx_thread: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for L1_tx_thread: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for L1_stats: creating thread with affinity ffffffff, priority 1
TYPE <CTRL-C> TO TERMINATE
[PHY]   got sync (ru_thread)
[PHY]   got sync (L1_stats_thread)
[HW]   Trying to connect to 172.17.0.91:4043
[HW]   Connection to 172.17.0.91:4043 established
[PHY]   RU 0 rf device ready
[PHY]   RU 0 RF started cpu_meas_enabled 0
[PHY]   Command line parameters for OAI UE: -C 3619200000 -r 106 --numerology 1 --ssb 516 
[NR_MAC]   Frame.Slot 512.0

[NR_MAC]   Frame.Slot 640.0
....

[NR_MAC]   Added new CFRA process for UE RNTI 324d with initial CellGroup
[RLC]   Activated srb0 for UE 12877
[RLC]   Added srb 1 to UE 12877
[RLC]   Added srb 2 to UE 12877
[RLC]   Added drb 1 to UE 12877
[RLC]   Added DRB to UE 12877
[GTPU]   [94] Created tunnel for UE ID 12877, teid for incoming: 77fddcba, teid for outgoing 90e1d19e to remote IPv4: 172.17.0.93, IPv6 ::
[NR_PHY]   [RAPROC] 353.19 Initiating RA procedure with preamble 63, energy 44.0 dB (I0 0, thres 200), delay 0 start symbol 0 freq index 0
[NR_MAC]   353.19 UE RA-RNTI 010b TC-RNTI 324d: initiating RA procedure
[NR_MAC]   UE 324d: Msg3 scheduled at 354.17 (354.7 TDA 3) start 0 RBs 8
[NR_MAC]   UE 324d: 354.7 Generating RA-Msg2 DCI, RA RNTI 0x10b, state 1, preamble_index(RAPID) 63, timing_offset = 0 (estimated distance 0.0 [m])
[NR_MAC]   354.7 Send RAR to RA-RNTI 010b
[NR_MAC]    354.17 PUSCH with TC_RNTI 0x324d received correctly
[NR_MAC]   (rnti 0x324d) CFRA procedure succeeded!
[NR_MAC]   cannot add LCID 1: already present, updating configuration
[NR_MAC]   cannot add LCID 2: already present, updating configuration
[NR_MAC]   cannot add LCID 4: already present, updating configuration
[NR_MAC]   Adding new UE context with RNTI 0x324d
[NR_MAC]   Frame.Slot 384.0
UE RNTI 324d CU-UE-ID 1 out-of-sync PH 0 dB PCMAX 0 dBm, average RSRP -44 (2 meas)
UE 324d: dlsch_rounds 0/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE 324d: ulsch_rounds 29/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.08100 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 50.5 dB
UE 324d: MAC:    TX              0 RX             84 bytes
UE 324d: LCID 1: TX              0 RX              0 bytes
UE 324d: LCID 2: TX              0 RX              0 bytes
UE 324d: LCID 4: TX              0 RX              0 bytes

[NR_MAC]   DU received confirmation of successful RRC Reconfiguration
[NR_MAC]   Frame.Slot 512.0
UE RNTI 324d CU-UE-ID 1 out-of-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (8 meas)
UE 324d: dlsch_rounds 1/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE 324d: ulsch_rounds 137/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.02059 MCS (0) 1 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 324d: MAC:    TX             21 RX            872 bytes
UE 324d: LCID 1: TX              3 RX             34 bytes
UE 324d: LCID 2: TX              0 RX              0 bytes
UE 324d: LCID 4: TX              0 RX              0 bytes
....




```


# amf logs
```bash

-- Boot cf3da29be3b04934a8a9c9e4209f3da2 --
Aug 14 06:09:05 open5gs-vm systemd[1]: Started open5gs-amfd.service - Open5GS AMF Daemon.
Aug 14 06:09:05 open5gs-vm open5gs-amfd[541]: Open5GS daemon v2.7.6
Aug 14 06:09:05 open5gs-vm open5gs-amfd[541]: 08/14 06:09:05.789: [app] INFO: Configuration: '/etc/open5gs/amf.yaml' (../lib/app/ogs-init.c:144)
Aug 14 06:09:05 open5gs-vm open5gs-amfd[541]: 08/14 06:09:05.789: [app] INFO: File Logging: '/var/log/open5gs/amf.log' (../lib/app/ogs-init.c:147)
Aug 14 06:09:05 open5gs-vm open5gs-amfd[541]: 08/14 06:09:05.856: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.200:7777] (../lib/sbi/context.c:507)
Aug 14 06:09:05 open5gs-vm open5gs-amfd[541]: 08/14 06:09:05.856: [metrics] INFO: metrics_server() [http://127.0.0.5]:9090 (../lib/metrics/prometheus/context.c:300)
Aug 14 06:09:05 open5gs-vm open5gs-amfd[541]: 08/14 06:09:05.857: [sbi] INFO: NF Service [namf-comm] (../lib/sbi/context.c:1994)
Aug 14 06:09:05 open5gs-vm open5gs-amfd[541]: 08/14 06:09:05.866: [sbi] INFO: nghttp2_server() [http://127.0.0.5]:7777 (../lib/sbi/nghttp2-server.c:439)
Aug 14 06:09:06 open5gs-vm open5gs-amfd[541]: 08/14 06:09:06.140: [amf] INFO: ngap_server() [172.17.0.95]:38412 (../src/amf/ngap-sctp.c:61)
Aug 14 06:09:06 open5gs-vm open5gs-amfd[541]: 08/14 06:09:06.140: [sctp] INFO: AMF initialize...done (../src/amf/app.c:33)
Aug 14 06:09:06 open5gs-vm open5gs-amfd[541]: 08/14 06:09:06.140: [sbi] WARNING: Couldn't connect to server (7): Failed to connect to 127.0.0.200 port 7777 after 0 ms: Couldn't connect to server (../lib/sbi/client.c:757)
Aug 14 06:09:06 open5gs-vm open5gs-amfd[541]: 08/14 06:09:06.140: [sbi] WARNING: ogs_sbi_client_handler() failed [-1] (../lib/sbi/path.c:62)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:16.869: [sbi] WARNING: [2edf6d08-78d5-41f0-b082-c5f898b0c456] Retry registration with NRF (../lib/sbi/nf-sm.c:256)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:16.887: [sbi] INFO: [2edf6d08-78d5-41f0-b082-c5f898b0c456] NF registered [Heartbeat:10s] (../lib/sbi/nf-sm.c:295)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:16.904: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.10:7777] (../lib/sbi/nnrf-handler.c:955)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:16.904: [sbi] INFO: [3578400e-78d5-41f0-b1e2-69ffbd192c93] Subscription created until 2025-08-15T06:09:16.892665+00:00 [duration:86400000000,validity:86400.000000,patch:43200.000000] (../lib/sbi/nnrf-handler.c:874)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:16.904: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.10:7777] (../lib/sbi/nnrf-handler.c:955)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:16.904: [sbi] INFO: [3578db40-78d5-41f0-b1e2-69ffbd192c93] Subscription created until 2025-08-15T06:09:16.896614+00:00 [duration:86400000000,validity:86400.000000,patch:43200.000000] (../lib/sbi/nnrf-handler.c:874)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:16.904: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.10:7777] (../lib/sbi/nnrf-handler.c:955)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:16.904: [sbi] INFO: [3578e34c-78d5-41f0-b1e2-69ffbd192c93] Subscription created until 2025-08-15T06:09:16.896777+00:00 [duration:86400000000,validity:86400.000000,patch:43200.000000] (../lib/sbi/nnrf-handler.c:874)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:16.914: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.10:7777] (../lib/sbi/nnrf-handler.c:955)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:16.914: [sbi] INFO: [3578e784-78d5-41f0-b1e2-69ffbd192c93] Subscription created until 2025-08-15T06:09:16.896902+00:00 [duration:86400000000,validity:86400.000000,patch:43200.000000] (../lib/sbi/nnrf-handler.c:874)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:16.914: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.10:7777] (../lib/sbi/nnrf-handler.c:955)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:16.914: [sbi] INFO: [3578ec02-78d5-41f0-b1e2-69ffbd192c93] Subscription created until 2025-08-15T06:09:16.897000+00:00 [duration:86400000000,validity:86400.000000,patch:43200.000000] (../lib/sbi/nnrf-handler.c:874)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:16.915: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.10:7777] (../lib/sbi/nnrf-handler.c:955)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:16.915: [sbi] INFO: [3578f1b6-78d5-41f0-b1e2-69ffbd192c93] Subscription created until 2025-08-15T06:09:16.897151+00:00 [duration:86400000000,validity:86400.000000,patch:43200.000000] (../lib/sbi/nnrf-handler.c:874)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:16.917: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.10:7777] (../lib/sbi/nnrf-handler.c:955)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:16.917: [sbi] INFO: [3578f670-78d5-41f0-b1e2-69ffbd192c93] Subscription created until 2025-08-15T06:09:16.897265+00:00 [duration:86400000000,validity:86400.000000,patch:43200.000000] (../lib/sbi/nnrf-handler.c:874)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:16.936: [sbi] INFO: [2f55647c-78d5-41f0-b2d8-a914c1f72e57] (NRF-profile-get) NF registered (../lib/sbi/nf-sm.c:81)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:16.936: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.12:80] (../lib/sbi/context.c:2374)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:16.936: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../lib/sbi/context.c:2113)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:16.936: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../lib/sbi/context.c:2113)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:16.936: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../lib/sbi/context.c:2113)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:16.942: [sbi] INFO: [2f29b7dc-78d5-41f0-a37e-6f5b9d79766f] (NRF-profile-get) NF registered (../lib/sbi/nf-sm.c:81)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:16.942: [sbi] INFO: Setup NF EndPoint(addr) [127.0.1.250:7777] (../lib/sbi/context.c:2374)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:16.946: [sbi] INFO: [2fef06a4-78d5-41f0-ad66-6d5fcc288bc7] (NRF-profile-get) NF registered (../lib/sbi/nf-sm.c:81)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:16.946: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.4:80] (../lib/sbi/context.c:2374)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:16.946: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../lib/sbi/context.c:2113)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:16.947: [sbi] INFO: [2ed25e7e-78d5-41f0-b22b-2fc5d8fb335f] (NRF-profile-get) NF registered (../lib/sbi/nf-sm.c:81)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:16.947: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.11:80] (../lib/sbi/context.c:2374)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:16.947: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.11:7777] (../lib/sbi/context.c:2113)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:17.128: [sbi] INFO: [2f09e2fe-78d5-41f0-a580-57ea83526c32] (NRF-notify) NF registered (../lib/sbi/nnrf-handler.c:1140)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:17.128: [sbi] INFO: [2f09e2fe-78d5-41f0-a580-57ea83526c32] (NRF-notify) NF Profile updated [type:NSSF] (../lib/sbi/nnrf-handler.c:1154)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:17.128: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.14:80] (../lib/sbi/context.c:2374)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:17.128: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.14:7777] (../lib/sbi/context.c:2113)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:17.219: [sbi] INFO: [359ec10c-78d5-41f0-b59d-f966836054ac] (NRF-notify) NF registered (../lib/sbi/nnrf-handler.c:1140)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:17.219: [sbi] INFO: [359ec10c-78d5-41f0-b59d-f966836054ac] (NRF-notify) NF Profile updated [type:PCF] (../lib/sbi/nnrf-handler.c:1154)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:17.219: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.13:80] (../lib/sbi/context.c:2374)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:17.219: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../lib/sbi/context.c:2113)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal systemd[1]: Reloading open5gs-amfd.service - Open5GS AMF Daemon...
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal systemd[1]: Reloaded open5gs-amfd.service - Open5GS AMF Daemon.
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:09:17.827: [app] INFO: SIGHUP received (../src/main.c:59)
Aug 14 06:19:03 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:19:03.007: [amf] INFO: gNB-N2 accepted[172.17.0.93]:36162 in ng-path module (../src/amf/ngap-sctp.c:113)
Aug 14 06:19:03 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:19:03.010: [amf] INFO: gNB-N2 accepted[172.17.0.93] in master_sm module (../src/amf/amf-sm.c:894)
Aug 14 06:19:03 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:19:03.047: [amf] INFO: [Added] Number of gNBs is now 1 (../src/amf/context.c:1277)
Aug 14 06:19:03 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:19:03.048: [amf] INFO: gNB-N2[172.17.0.93] max_num_of_ostreams : 5 (../src/amf/amf-sm.c:941)
Aug 14 06:23:41 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:23:41.908: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:437)
Aug 14 06:23:41 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:23:41.908: [amf] INFO: [Added] Number of gNB-UEs is now 1 (../src/amf/context.c:2789)
Aug 14 06:23:41 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:23:41.908: [amf] INFO:     RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[1] TAC[1] CellID[0xe0000] (../src/amf/ngap-handler.c:598)
Aug 14 06:23:41 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:23:41.912: [amf] INFO: [suci-0-999-70-0000-0-0-0000000001] Unknown UE by SUCI (../src/amf/context.c:1906)
Aug 14 06:23:41 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:23:41.912: [amf] INFO: [Added] Number of AMF-UEs is now 1 (../src/amf/context.c:1682)
Aug 14 06:23:41 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:23:41.912: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:1339)
Aug 14 06:23:41 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:23:41.914: [gmm] INFO: [suci-0-999-70-0000-0-0-0000000001]    SUCI (../src/amf/gmm-handler.c:183)
Aug 14 06:23:41 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:23:41.915: [sbi] INFO: [2ed25e7e-78d5-41f0-b22b-2fc5d8fb335f] Setup NF Instance [type:AUSF] (../lib/sbi/path.c:307)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:23:42.003: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.11:7777] (../src/amf/nausf-handler.c:130)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:23:42.268: [sbi] INFO: [2f55647c-78d5-41f0-b2d8-a914c1f72e57] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:23:42.274: [sbi] INFO: [2f55647c-78d5-41f0-b2d8-a914c1f72e57] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:23:42.291: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/amf/nudm-handler.c:355)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:23:42.294: [sbi] INFO: [359ec10c-78d5-41f0-b59d-f966836054ac] Setup NF Instance [type:PCF] (../lib/sbi/path.c:307)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:23:42.301: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/amf/npcf-handler.c:143)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:23:42.513: [gmm] INFO: [imsi-999700000000001] Registration complete (../src/amf/gmm-sm.c:2698)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:23:42.513: [amf] INFO: [imsi-999700000000001] Configuration update command (../src/amf/nas-path.c:609)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:23:42.513: [gmm] INFO:     UTC [2025-08-14T06:23:42] Timezone[0]/DST[0] (../src/amf/gmm-build.c:551)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:23:42.513: [gmm] INFO:     LOCAL [2025-08-14T06:23:42] Timezone[0]/DST[0] (../src/amf/gmm-build.c:556)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:23:42.516: [amf] INFO: [Added] Number of AMF-Sessions is now 1 (../src/amf/context.c:2810)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:23:42.516: [gmm] INFO: UE SUPI[imsi-999700000000001] DNN[internet] S_NSSAI[SST:1 SD:0xffffff] smContextRef[NULL] smContextResourceURI[NULL] (../src/amf/gmm-handler.c:1383)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:23:42.516: [amf] INFO: [2fef06a4-78d5-41f0-ad66-6d5fcc288bc7] Setup NF Instance [type:SMF] (../src/amf/context.c:2433)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:23:42.516: [gmm] INFO: SMF Instance [2fef06a4-78d5-41f0-ad66-6d5fcc288bc7] (../src/amf/gmm-handler.c:1424)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:23:42.532: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/amf/nsmf-handler.c:144)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:23:42.585: [amf] INFO: [imsi-999700000000001:10:11][0:0:NULL] /nsmf-pdusession/v1/sm-contexts/{smContextRef}/modify (../src/amf/nsmf-handler.c:947)
Aug 14 06:26:40 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:26:40.936: [gmm] INFO: [imsi-999700000000001] Deregistration request (../src/amf/gmm-sm.c:1623)
Aug 14 06:26:40 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:26:40.936: [gmm] INFO: [suci-0-999-70-0000-0-0-0000000001]    SUCI (../src/amf/gmm-handler.c:890)
Aug 14 06:26:40 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:26:40.946: [amf] INFO: [imsi-999700000000001:10] Release SM context [204] (../src/amf/amf-sm.c:581)
Aug 14 06:26:40 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:26:40.946: [amf] INFO: [imsi-999700000000001:10] Release SM Context [state:1] (../src/amf/nsmf-handler.c:1168)
Aug 14 06:26:40 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:26:40.946: [amf] INFO: [Removed] Number of AMF-Sessions is now 0 (../src/amf/context.c:2817)
Aug 14 06:26:41 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:26:41.164: [amf] INFO: UE Context Release [Action:2] (../src/amf/ngap-handler.c:1733)
Aug 14 06:26:41 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:26:41.164: [amf] INFO:     RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[1] (../src/amf/ngap-handler.c:1734)
Aug 14 06:26:41 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:26:41.164: [amf] INFO:     SUCI[suci-0-999-70-0000-0-0-0000000001] (../src/amf/ngap-handler.c:1738)
Aug 14 06:26:41 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:26:41.164: [amf] INFO: [Removed] Number of gNB-UEs is now 0 (../src/amf/context.c:2796)
Aug 14 06:26:58 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:26:58.067: [amf] INFO: gNB-N2[172.17.0.93] connection refused!!! (../src/amf/amf-sm.c:954)
Aug 14 06:26:58 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:26:58.067: [amf] INFO: [Removed] Number of gNBs is now 0 (../src/amf/context.c:1305)
Aug 14 06:38:05 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:38:05.442: [amf] INFO: gNB-N2 accepted[172.17.0.93]:42478 in ng-path module (../src/amf/ngap-sctp.c:113)
Aug 14 06:38:05 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:38:05.443: [amf] INFO: gNB-N2 accepted[172.17.0.93] in master_sm module (../src/amf/amf-sm.c:894)
Aug 14 06:38:05 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:38:05.460: [amf] INFO: [Added] Number of gNBs is now 1 (../src/amf/context.c:1277)
Aug 14 06:38:05 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:38:05.460: [amf] INFO: gNB-N2[172.17.0.93] max_num_of_ostreams : 5 (../src/amf/amf-sm.c:941)
Aug 14 06:39:41 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:39:41.167: [gmm] WARNING: [imsi-999700000000001] Mobile Reachable Timer Expired (../src/amf/gmm-sm.c:195)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:41:32.569: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:437)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:41:32.569: [amf] INFO: [Added] Number of gNB-UEs is now 1 (../src/amf/context.c:2789)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:41:32.569: [amf] INFO:     RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[2] TAC[1] CellID[0xe0000] (../src/amf/ngap-handler.c:598)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:41:32.569: [amf] INFO: [suci-0-999-70-0000-0-0-0000000001] known UE by SUCI (../src/amf/context.c:1904)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:41:32.569: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:1339)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:41:32.569: [gmm] INFO: [suci-0-999-70-0000-0-0-0000000001]    SUCI (../src/amf/gmm-handler.c:183)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:41:32.577: [amf] WARNING: UnRef NF EndPoint(addr) [127.0.0.11:7777] (../src/amf/nausf-handler.c:130)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:41:32.577: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.11:7777] (../src/amf/nausf-handler.c:130)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:41:32.679: [amf] WARNING: UnRef NF EndPoint(addr) [127.0.0.12:7777] (../src/amf/nudm-handler.c:355)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:41:32.679: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/amf/nudm-handler.c:355)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:41:32.685: [amf] WARNING: UnRef NF EndPoint(addr) [127.0.0.13:7777] (../src/amf/npcf-handler.c:143)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:41:32.685: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/amf/npcf-handler.c:143)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:41:32.772: [gmm] INFO: [imsi-999700000000001] Registration complete (../src/amf/gmm-sm.c:2698)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:41:32.772: [amf] INFO: [imsi-999700000000001] Configuration update command (../src/amf/nas-path.c:609)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:41:32.772: [gmm] INFO:     UTC [2025-08-14T06:41:32] Timezone[0]/DST[0] (../src/amf/gmm-build.c:551)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:41:32.772: [gmm] INFO:     LOCAL [2025-08-14T06:41:32] Timezone[0]/DST[0] (../src/amf/gmm-build.c:556)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:41:32.772: [amf] INFO: [Added] Number of AMF-Sessions is now 1 (../src/amf/context.c:2810)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:41:32.772: [gmm] INFO: UE SUPI[imsi-999700000000001] DNN[internet] S_NSSAI[SST:1 SD:0xffffff] smContextRef[NULL] smContextResourceURI[NULL] (../src/amf/gmm-handler.c:1383)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:41:32.772: [amf] INFO: [2fef06a4-78d5-41f0-ad66-6d5fcc288bc7] Setup NF Instance [type:SMF] (../src/amf/context.c:2433)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:41:32.772: [gmm] INFO: SMF Instance [2fef06a4-78d5-41f0-ad66-6d5fcc288bc7] (../src/amf/gmm-handler.c:1424)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:41:32.780: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/amf/nsmf-handler.c:144)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:41:32.824: [amf] INFO: [imsi-999700000000001:10:11][0:0:NULL] /nsmf-pdusession/v1/sm-contexts/{smContextRef}/modify (../src/amf/nsmf-handler.c:947)
Aug 14 06:42:54 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:42:54.551: [amf] INFO: [imsi-999700000000001:10:13][0:0:NULL] /nsmf-pdusession/v1/sm-contexts/{smContextRef}/modify (../src/amf/nsmf-handler.c:947)
Aug 14 06:42:54 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:42:54.552: [amf] INFO: UE Context Release [Action:2] (../src/amf/ngap-handler.c:1733)
Aug 14 06:42:54 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:42:54.552: [amf] INFO:     RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[2] (../src/amf/ngap-handler.c:1734)
Aug 14 06:42:54 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:42:54.552: [amf] INFO:     SUCI[suci-0-999-70-0000-0-0-0000000001] (../src/amf/ngap-handler.c:1738)
Aug 14 06:42:54 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/14 06:42:54.552: [amf] INFO: [Removed] Number of gNB-UEs is now 0 (../src/amf/context.c:2796)

```


# upf logs 
```bash

-- Boot cf3da29be3b04934a8a9c9e4209f3da2 --
Aug 14 06:09:06 open5gs-vm systemd[1]: Started open5gs-upfd.service - Open5GS UPF Daemon.
Aug 14 06:09:06 open5gs-vm open5gs-upfd[596]: Open5GS daemon v2.7.6
Aug 14 06:09:06 open5gs-vm open5gs-upfd[596]: 08/14 06:09:06.806: [app] INFO: Configuration: '/etc/open5gs/upf.yaml' (../lib/app/ogs-init.c:144)
Aug 14 06:09:06 open5gs-vm open5gs-upfd[596]: 08/14 06:09:06.806: [app] INFO: File Logging: '/var/log/open5gs/upf.log' (../lib/app/ogs-init.c:147)
Aug 14 06:09:07 open5gs-vm open5gs-upfd[596]: 08/14 06:09:07.812: [metrics] INFO: metrics_server() [http://127.0.0.7]:9090 (../lib/metrics/prometheus/context.c:300)
Aug 14 06:09:07 open5gs-vm open5gs-upfd[596]: 08/14 06:09:07.824: [pfcp] INFO: pfcp_server() [127.0.0.7]:8805 (../lib/pfcp/path.c:30)
Aug 14 06:09:07 open5gs-vm open5gs-upfd[596]: 08/14 06:09:07.824: [gtp] INFO: gtp_server() [172.17.0.95]:2152 (../lib/gtp/path.c:30)
Aug 14 06:09:07 open5gs-vm open5gs-upfd[596]: 08/14 06:09:07.824: [app] INFO: UPF initialize...done (../src/upf/app.c:31)
Aug 14 06:09:08 open5gs-vm open5gs-upfd[596]: 08/14 06:09:08.362: [upf] INFO: PFCP associated [127.0.0.4]:8805 (../src/upf/pfcp-sm.c:168)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 08/14 06:09:16.689: [app] INFO: SIGHUP received (../src/main.c:59)
Aug 14 06:09:16 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal systemd[1]: Reloading open5gs-upfd.service - Open5GS UPF Daemon...
Aug 14 06:09:16 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal systemd[1]: Reloaded open5gs-upfd.service - Open5GS UPF Daemon.
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 08/14 06:23:42.542: [upf] INFO: [Added] Number of UPF-Sessions is now 1 (../src/upf/context.c:212)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 08/14 06:23:42.543: [gtp] INFO: gtp_connect() [127.0.0.4]:2152 (../lib/gtp/path.c:60)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 08/14 06:23:42.543: [upf] INFO: UE F-SEID[UP:0x487 CP:0xfba] APN[internet] PDN-Type[1] IPv4[10.45.0.2] IPv6[] (../src/upf/context.c:498)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 08/14 06:23:42.580: [gtp] INFO: gtp_connect() [172.17.0.93]:2152 (../lib/gtp/path.c:60)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 08/14 06:23:42.639: [upf] ERROR: Invalid packet [IP version:6, Packet Length:48] (../src/upf/gtp-path.c:622)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0000: 60000000 00083aff fe800000 00000000   `.....:.........
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0010: 226ab6ab 466c3ada ff020000 00000000   "j..Fl:.........
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0020: 00000000 00000002 850022db 00000000   ..........".....
Aug 14 06:23:47 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 08/14 06:23:47.248: [upf] ERROR: Invalid packet [IP version:6, Packet Length:48] (../src/upf/gtp-path.c:622)
Aug 14 06:23:47 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0000: 60000000 00083aff fe800000 00000000   `.....:.........
Aug 14 06:23:47 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0010: 226ab6ab 466c3ada ff020000 00000000   "j..Fl:.........
Aug 14 06:23:47 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0020: 00000000 00000002 850022db 00000000   ..........".....
Aug 14 06:23:55 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 08/14 06:23:55.432: [upf] ERROR: Invalid packet [IP version:6, Packet Length:48] (../src/upf/gtp-path.c:622)
Aug 14 06:23:55 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0000: 60000000 00083aff fe800000 00000000   `.....:.........
Aug 14 06:23:55 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0010: 226ab6ab 466c3ada ff020000 00000000   "j..Fl:.........
Aug 14 06:23:55 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0020: 00000000 00000002 850022db 00000000   ..........".....
Aug 14 06:24:11 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 08/14 06:24:11.295: [upf] ERROR: Invalid packet [IP version:6, Packet Length:48] (../src/upf/gtp-path.c:622)
Aug 14 06:24:11 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0000: 60000000 00083aff fe800000 00000000   `.....:.........
Aug 14 06:24:11 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0010: 226ab6ab 466c3ada ff020000 00000000   "j..Fl:.........
Aug 14 06:24:11 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0020: 00000000 00000002 850022db 00000000   ..........".....
Aug 14 06:24:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 08/14 06:24:42.577: [upf] ERROR: Invalid packet [IP version:6, Packet Length:48] (../src/upf/gtp-path.c:622)
Aug 14 06:24:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0000: 60000000 00083aff fe800000 00000000   `.....:.........
Aug 14 06:24:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0010: 226ab6ab 466c3ada ff020000 00000000   "j..Fl:.........
Aug 14 06:24:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0020: 00000000 00000002 850022db 00000000   ..........".....
Aug 14 06:25:50 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 08/14 06:25:50.760: [upf] ERROR: Invalid packet [IP version:6, Packet Length:48] (../src/upf/gtp-path.c:622)
Aug 14 06:25:50 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0000: 60000000 00083aff fe800000 00000000   `.....:.........
Aug 14 06:25:50 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0010: 226ab6ab 466c3ada ff020000 00000000   "j..Fl:.........
Aug 14 06:25:50 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0020: 00000000 00000002 850022db 00000000   ..........".....
Aug 14 06:26:40 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 08/14 06:26:40.941: [upf] INFO: [Removed] Number of UPF-sessions is now 0 (../src/upf/context.c:256)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 08/14 06:41:32.786: [upf] INFO: [Added] Number of UPF-Sessions is now 1 (../src/upf/context.c:212)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 08/14 06:41:32.787: [upf] INFO: UE F-SEID[UP:0x73e CP:0xe1a] APN[internet] PDN-Type[1] IPv4[10.45.0.3] IPv6[] (../src/upf/context.c:498)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 08/14 06:41:32.870: [upf] ERROR: Invalid packet [IP version:6, Packet Length:48] (../src/upf/gtp-path.c:622)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0000: 60000000 00083aff fe800000 00000000   `.....:.........
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0010: 57225b55 01aff790 ff020000 00000000   W"[U............
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0020: 00000000 00000002 8500d17f 00000000   ................
Aug 14 06:41:37 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 08/14 06:41:37.005: [upf] ERROR: Invalid packet [IP version:6, Packet Length:48] (../src/upf/gtp-path.c:622)
Aug 14 06:41:37 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0000: 60000000 00083aff fe800000 00000000   `.....:.........
Aug 14 06:41:37 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0010: 57225b55 01aff790 ff020000 00000000   W"[U............
Aug 14 06:41:37 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0020: 00000000 00000002 8500d17f 00000000   ................
Aug 14 06:41:44 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 08/14 06:41:44.994: [upf] ERROR: Invalid packet [IP version:6, Packet Length:48] (../src/upf/gtp-path.c:622)
Aug 14 06:41:44 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0000: 60000000 00083aff fe800000 00000000   `.....:.........
Aug 14 06:41:44 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0010: 57225b55 01aff790 ff020000 00000000   W"[U............
Aug 14 06:41:44 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0020: 00000000 00000002 8500d17f 00000000   ................
Aug 14 06:42:02 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 08/14 06:42:02.185: [upf] ERROR: Invalid packet [IP version:6, Packet Length:48] (../src/upf/gtp-path.c:622)
Aug 14 06:42:02 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0000: 60000000 00083aff fe800000 00000000   `.....:.........
Aug 14 06:42:02 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0010: 57225b55 01aff790 ff020000 00000000   W"[U............
Aug 14 06:42:02 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0020: 00000000 00000002 8500d17f 00000000   ................
Aug 14 06:42:34 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 08/14 06:42:34.162: [upf] ERROR: Invalid packet [IP version:6, Packet Length:48] (../src/upf/gtp-path.c:622)
Aug 14 06:42:34 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0000: 60000000 00083aff fe800000 00000000   `.....:.........
Aug 14 06:42:34 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0010: 57225b55 01aff790 ff020000 00000000   W"[U............
Aug 14 06:42:34 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-upfd[596]: 0020: 00000000 00000002 8500d17f 00000000   ................

```



# smf logs
```bash 

-- Boot cf3da29be3b04934a8a9c9e4209f3da2 --
Aug 14 06:09:06 open5gs-vm systemd[1]: Started open5gs-smfd.service - Open5GS SMF Daemon.
Aug 14 06:09:06 open5gs-vm open5gs-smfd[588]: Open5GS daemon v2.7.6
Aug 14 06:09:06 open5gs-vm open5gs-smfd[588]: 08/14 06:09:06.586: [app] INFO: Configuration: '/etc/open5gs/smf.yaml' (../lib/app/ogs-init.c:144)
Aug 14 06:09:06 open5gs-vm open5gs-smfd[588]: 08/14 06:09:06.586: [app] INFO: File Logging: '/var/log/open5gs/smf.log' (../lib/app/ogs-init.c:147)
Aug 14 06:09:07 open5gs-vm open5gs-smfd[588]: 08/14 06:09:07.705: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.200:7777] (../lib/sbi/context.c:507)
Aug 14 06:09:07 open5gs-vm open5gs-smfd[588]: 08/14 06:09:07.721: [metrics] INFO: metrics_server() [http://127.0.0.4]:9090 (../lib/metrics/prometheus/context.c:300)
Aug 14 06:09:08 open5gs-vm open5gs-smfd[588]: 08/14 06:09:08.350: [app] INFO: Polling freeDiameter stats every 60000000 usecs (../lib/diameter/common/stats.c:77)
Aug 14 06:09:08 open5gs-vm open5gs-smfd[588]: 08/14 06:09:08.350: [gtp] INFO: gtp_server() [127.0.0.4]:2123 (../lib/gtp/path.c:30)
Aug 14 06:09:08 open5gs-vm open5gs-smfd[588]: 08/14 06:09:08.350: [gtp] INFO: gtp_server() [127.0.0.4]:2152 (../lib/gtp/path.c:30)
Aug 14 06:09:08 open5gs-vm open5gs-smfd[588]: 08/14 06:09:08.350: [pfcp] INFO: pfcp_server() [127.0.0.4]:8805 (../lib/pfcp/path.c:30)
Aug 14 06:09:08 open5gs-vm open5gs-smfd[588]: 08/14 06:09:08.361: [sbi] INFO: NF Service [nsmf-pdusession] (../lib/sbi/context.c:1994)
Aug 14 06:09:08 open5gs-vm open5gs-smfd[588]: 08/14 06:09:08.361: [sbi] INFO: nghttp2_server() [http://127.0.0.4]:7777 (../lib/sbi/nghttp2-server.c:439)
Aug 14 06:09:08 open5gs-vm open5gs-smfd[588]: 08/14 06:09:08.362: [app] INFO: SMF initialize...done (../src/smf/app.c:31)
Aug 14 06:09:08 open5gs-vm open5gs-smfd[588]: 08/14 06:09:08.365: [smf] INFO: PFCP associated [127.0.0.7]:8805 (../src/smf/pfcp-sm.c:188)
Aug 14 06:09:08 open5gs-vm open5gs-smfd[588]: 08/14 06:09:08.373: [sbi] INFO: [2fef06a4-78d5-41f0-ad66-6d5fcc288bc7] NF registered [Heartbeat:10s] (../lib/sbi/nf-sm.c:295)
Aug 14 06:09:08 open5gs-vm open5gs-smfd[588]: 08/14 06:09:08.386: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.10:7777] (../lib/sbi/nnrf-handler.c:955)
Aug 14 06:09:08 open5gs-vm open5gs-smfd[588]: 08/14 06:09:08.386: [sbi] INFO: [30657d2a-78d5-41f0-b1e2-69ffbd192c93] Subscription created until 2025-08-15T06:09:08.381033+00:00 [duration:86400000000,validity:86400.000000,patch:43200.000000] (../lib/sbi/nnrf-handler.c:874)
Aug 14 06:09:08 open5gs-vm open5gs-smfd[588]: 08/14 06:09:08.386: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.10:7777] (../lib/sbi/nnrf-handler.c:955)
Aug 14 06:09:08 open5gs-vm open5gs-smfd[588]: 08/14 06:09:08.386: [sbi] INFO: [306581da-78d5-41f0-b1e2-69ffbd192c93] Subscription created until 2025-08-15T06:09:08.381157+00:00 [duration:86400000000,validity:86400.000000,patch:43200.000000] (../lib/sbi/nnrf-handler.c:874)
Aug 14 06:09:08 open5gs-vm open5gs-smfd[588]: 08/14 06:09:08.386: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.10:7777] (../lib/sbi/nnrf-handler.c:955)
Aug 14 06:09:08 open5gs-vm open5gs-smfd[588]: 08/14 06:09:08.386: [sbi] INFO: [306576ae-78d5-41f0-b1e2-69ffbd192c93] Subscription created until 2025-08-15T06:09:08.380902+00:00 [duration:86400000000,validity:86400.000000,patch:43200.000000] (../lib/sbi/nnrf-handler.c:874)
Aug 14 06:09:08 open5gs-vm open5gs-smfd[588]: 08/14 06:09:08.395: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.10:7777] (../lib/sbi/nnrf-handler.c:955)
Aug 14 06:09:08 open5gs-vm open5gs-smfd[588]: 08/14 06:09:08.395: [sbi] INFO: [30658716-78d5-41f0-b1e2-69ffbd192c93] Subscription created until 2025-08-15T06:09:08.381290+00:00 [duration:86400000000,validity:86400.000000,patch:43200.000000] (../lib/sbi/nnrf-handler.c:874)
Aug 14 06:09:08 open5gs-vm open5gs-smfd[588]: 08/14 06:09:08.396: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.10:7777] (../lib/sbi/nnrf-handler.c:955)
Aug 14 06:09:08 open5gs-vm open5gs-smfd[588]: 08/14 06:09:08.396: [sbi] INFO: [30658c34-78d5-41f0-b1e2-69ffbd192c93] Subscription created until 2025-08-15T06:09:08.381416+00:00 [duration:86400000000,validity:86400.000000,patch:43200.000000] (../lib/sbi/nnrf-handler.c:874)
Aug 14 06:09:08 open5gs-vm open5gs-smfd[588]: 08/14 06:09:08.414: [sbi] INFO: [2f55647c-78d5-41f0-b2d8-a914c1f72e57] (NRF-profile-get) NF registered (../lib/sbi/nf-sm.c:81)
Aug 14 06:09:08 open5gs-vm open5gs-smfd[588]: 08/14 06:09:08.414: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.12:80] (../lib/sbi/context.c:2374)
Aug 14 06:09:08 open5gs-vm open5gs-smfd[588]: 08/14 06:09:08.414: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../lib/sbi/context.c:2113)
Aug 14 06:09:08 open5gs-vm open5gs-smfd[588]: 08/14 06:09:08.414: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../lib/sbi/context.c:2113)
Aug 14 06:09:08 open5gs-vm open5gs-smfd[588]: 08/14 06:09:08.414: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../lib/sbi/context.c:2113)
Aug 14 06:09:08 open5gs-vm open5gs-smfd[588]: 08/14 06:09:08.419: [sbi] INFO: [2f29b7dc-78d5-41f0-a37e-6f5b9d79766f] (NRF-profile-get) NF registered (../lib/sbi/nf-sm.c:81)
Aug 14 06:09:08 open5gs-vm open5gs-smfd[588]: 08/14 06:09:08.419: [sbi] INFO: Setup NF EndPoint(addr) [127.0.1.250:7777] (../lib/sbi/context.c:2374)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:09:16.881: [sbi] INFO: [2edf6d08-78d5-41f0-b082-c5f898b0c456] (NRF-notify) NF registered (../lib/sbi/nnrf-handler.c:1140)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:09:16.881: [sbi] INFO: [2edf6d08-78d5-41f0-b082-c5f898b0c456] (NRF-notify) NF Profile updated [type:AMF] (../lib/sbi/nnrf-handler.c:1154)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:09:16.881: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.5:80] (../lib/sbi/context.c:2374)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:09:16.881: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.5:7777] (../lib/sbi/context.c:2113)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:09:17.218: [sbi] INFO: [359ec10c-78d5-41f0-b59d-f966836054ac] (NRF-notify) NF registered (../lib/sbi/nnrf-handler.c:1140)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:09:17.218: [sbi] INFO: [359ec10c-78d5-41f0-b59d-f966836054ac] (NRF-notify) NF Profile updated [type:PCF] (../lib/sbi/nnrf-handler.c:1154)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:09:17.218: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.13:80] (../lib/sbi/context.c:2374)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:09:17.218: [sbi] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../lib/sbi/context.c:2113)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:09:17.317: [app] INFO: SIGHUP received (../src/main.c:59)
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal systemd[1]: Reloading open5gs-smfd.service - Open5GS SMF Daemon...
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal systemd[1]: Reloaded open5gs-smfd.service - Open5GS SMF Daemon.
Aug 14 06:09:17 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:09:17.756: [diam] INFO: CONNECTED TO 'pcrf.localdomain' (SCTP,soc#12): (../lib/diameter/common/logger.c:81)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:23:42.519: [smf] INFO: [Added] Number of SMF-UEs is now 1 (../src/smf/context.c:1033)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:23:42.521: [smf] INFO: [Added] Number of SMF-Sessions is now 1 (../src/smf/context.c:3203)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:23:42.521: [smf] INFO: Setup NF EndPoint(addr) [127.0.0.5:7777] (../src/smf/nsmf-handler.c:274)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:23:42.523: [sbi] INFO: [2f55647c-78d5-41f0-b2d8-a914c1f72e57] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:23:42.530: [smf] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/smf/nudm-handler.c:461)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:23:42.531: [sbi] INFO: [359ec10c-78d5-41f0-b59d-f966836054ac] Setup NF Instance [type:PCF] (../lib/sbi/path.c:307)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:23:42.541: [smf] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/smf/npcf-handler.c:367)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:23:42.541: [smf] INFO: UE SUPI[imsi-999700000000001] DNN[internet] IPv4[10.45.0.2] IPv6[] (../src/smf/npcf-handler.c:586)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:23:42.544: [gtp] INFO: gtp_connect() [172.17.0.95]:2152 (../lib/gtp/path.c:60)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:23:42.546: [sbi] INFO: [2edf6d08-78d5-41f0-b082-c5f898b0c456] Setup NF Instance [type:AMF] (../lib/sbi/path.c:307)
Aug 14 06:23:42 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:23:42.580: [sbi] INFO: [2f55647c-78d5-41f0-b2d8-a914c1f72e57] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)
Aug 14 06:26:40 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:26:40.945: [smf] INFO: Removed Session: UE IMSI:[imsi-999700000000001] DNN:[internet:10] IPv4:[10.45.0.2] IPv6:[] (../src/smf/context.c:1701)
Aug 14 06:26:40 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:26:40.946: [smf] INFO: [Removed] Number of SMF-Sessions is now 0 (../src/smf/context.c:3211)
Aug 14 06:26:40 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:26:40.946: [smf] INFO: [Removed] Number of SMF-UEs is now 0 (../src/smf/context.c:1097)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:41:32.774: [smf] INFO: [Added] Number of SMF-UEs is now 1 (../src/smf/context.c:1033)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:41:32.774: [smf] INFO: [Added] Number of SMF-Sessions is now 1 (../src/smf/context.c:3203)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:41:32.774: [smf] INFO: Setup NF EndPoint(addr) [127.0.0.5:7777] (../src/smf/nsmf-handler.c:274)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:41:32.774: [sbi] INFO: [2f55647c-78d5-41f0-b2d8-a914c1f72e57] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:41:32.779: [smf] INFO: Setup NF EndPoint(addr) [127.0.0.12:7777] (../src/smf/nudm-handler.c:461)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:41:32.779: [sbi] INFO: [359ec10c-78d5-41f0-b59d-f966836054ac] Setup NF Instance [type:PCF] (../lib/sbi/path.c:307)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:41:32.786: [smf] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/smf/npcf-handler.c:367)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:41:32.786: [smf] INFO: UE SUPI[imsi-999700000000001] DNN[internet] IPv4[10.45.0.3] IPv6[] (../src/smf/npcf-handler.c:586)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:41:32.788: [sbi] INFO: [2edf6d08-78d5-41f0-b082-c5f898b0c456] Setup NF Instance [type:AMF] (../lib/sbi/path.c:307)
Aug 14 06:41:32 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-smfd[588]: 08/14 06:41:32.822: [sbi] INFO: [2f55647c-78d5-41f0-b2d8-a914c1f72e57] Setup NF Instance [type:UDM] (../lib/sbi/path.c:307)

```




# Ping
```bash

telcomaan@oai-nr-ue-vm:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1460 qdisc mq state UP group default qlen 1000
    link/ether 42:01:ac:11:00:5b brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.91/32 metric 100 scope global dynamic ens4
       valid_lft 2686sec preferred_lft 2686sec
    inet6 fe80::4001:acff:fe11:5b/64 scope link 
       valid_lft forever preferred_lft forever
3: oaitun_ue1: <POINTOPOINT,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none 
    inet 10.45.0.2/24 scope global oaitun_ue1
       valid_lft forever preferred_lft forever
    inet6 fe80::226a:b6ab:466c:3ada/64 scope link stable-privacy 
       valid_lft forever preferred_lft forever
telcomaan@oai-nr-ue-vm:~$ ping -I oaitun_ue1 8.8.8.8
PING 8.8.8.8 (8.8.8.8) from 10.45.0.2 oaitun_ue1: 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=114 time=242 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=114 time=18.9 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=114 time=14.0 ms

--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
^Crtt min/avg/max/mdev = 13.958/91.778/242.430/106.546 ms

```