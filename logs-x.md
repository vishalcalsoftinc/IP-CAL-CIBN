[2025-08-28 11:03:37.844] [amf_sbi] [info] Receive Update NF Instance Request, handling ...

[2025-08-28 11:03:37.844] [amf_sbi] [info] Send HTTP message to http://oai-nrf:8080/nnrf-nfm/v1/nf-instances/ad3dea39-51e9-443c-9607-6b89ff53c3df

[2025-08-28 11:03:37.844] [amf_sbi] [info] HTTP message Body: [{"op":"replace","path":"/nfStatus","value":"REGISTERED"}]

[2025-08-28 11:03:37.847] [amf_sbi] [info] Get response with HTTP code (204)

[2025-08-28 11:03:37.847] [amf_sbi] [info] Could not get JSON content from the response

[2025-08-28 11:03:47.847] [amf_sbi] [info] Receive Update NF Instance Request, handling ...

[2025-08-28 11:03:47.847] [amf_sbi] [info] Send HTTP message to http://oai-nrf:8080/nnrf-nfm/v1/nf-instances/ad3dea39-51e9-443c-9607-6b89ff53c3df

[2025-08-28 11:03:47.847] [amf_sbi] [info] HTTP message Body: [{"op":"replace","path":"/nfStatus","value":"REGISTERED"}]

[2025-08-28 11:03:47.850] [amf_sbi] [info] Get response with HTTP code (204)

[2025-08-28 11:03:47.851] [amf_sbi] [info] Could not get JSON content from the response

[2025-08-28 11:03:57.620] [amf_app] [info]

|------------------------------------------------------------------------------------------------------------------------------------------------------------|

|----------------------------------------------------------------------gNBs' Information---------------------------------------------------------------------|

| Index | Status | Global Id | gNB Name | PLMN |

| - | - | - | - | - |

|------------------------------------------------------------------------------------------------------------------------------------------------------------|

|-----------------------------------------------------------------------------------------------------------------------------------------------------------|

|---------------------------------------------------------------------UEs' Information----------------------------------------------------------------------|

| Index | 5GMM State | IMSI | GUTI | RAN UE NGAP ID | AMF UE NGAP ID | PLMN | Cell Id |

| - | - | - | - | - | - | - | - |

|-----------------------------------------------------------------------------------------------------------------------------------------------------------|

[2025-08-28 11:03:57.851] [amf_sbi] [info] Receive Update NF Instance Request, handling ...

[2025-08-28 11:03:57.851] [amf_sbi] [info] Send HTTP message to http://oai-nrf:8080/nnrf-nfm/v1/nf-instances/ad3dea39-51e9-443c-9607-6b89ff53c3df

[2025-08-28 11:03:57.851] [amf_sbi] [info] HTTP message Body: [{"op":"replace","path":"/nfStatus","value":"REGISTERED"}]

[2025-08-28 11:03:57.854] [amf_sbi] [info] Get response with HTTP code (204)

[2025-08-28 11:03:57.854] [amf_sbi] [info] Could not get JSON content from the response

[2025-08-28 11:04:01.540] [sctp] [info] ----------------------

[2025-08-28 11:04:01.540] [sctp] [info] Local addresses:

[2025-08-28 11:04:01.541] [sctp] [info] - IPv4 Addr: 192.168.70.137

[2025-08-28 11:04:01.542] [sctp] [info] ----------------------

[2025-08-28 11:04:01.542] [sctp] [info] Peer addresses:

[2025-08-28 11:04:01.542] [sctp] [info] - IPv4 Addr: 172.17.0.93

[2025-08-28 11:04:01.542] [sctp] [info] ----------------------

[2025-08-28 11:04:01.542] [sctp] [info] [Assoc_id 11, Socket 8] Received a message (length 67) from port 39115, on stream 0, PPID 60

[2025-08-28 11:04:01.550] [amf_n2] [info] Received NGSetupRequest message, handling

[2025-08-28 11:04:07.855] [amf_sbi] [info] Receive Update NF Instance Request, handling ...

[2025-08-28 11:04:07.855] [amf_sbi] [info] Send HTTP message to http://oai-nrf:8080/nnrf-nfm/v1/nf-instances/ad3dea39-51e9-443c-9607-6b89ff53c3df

[2025-08-28 11:04:07.855] [amf_sbi] [info] HTTP message Body: [{"op":"replace","path":"/nfStatus","value":"REGISTERED"}]

[2025-08-28 11:04:07.858] [amf_sbi] [info] Get response with HTTP code (204)

[2025-08-28 11:04:07.858] [amf_sbi] [info] Could not get JSON content from the response

[2025-08-28 11:04:16.578] [sctp] [info] [Assoc_id 11, Socket 8] Received a message (length 76) from port 39115, on stream 1, PPID 60

[2025-08-28 11:04:16.578] [amf_n2] [info] Received Initial UE Message, handling

[2025-08-28 11:04:16.579] [amf_n2] [warning] No UE NGAP context with ran_ue_ngap_id 1, gnb_id 57344

[2025-08-28 11:04:16.579] [amf_app] [warning] No existing UE context associated with key app_ue_ranid_1:amfid_1

[2025-08-28 11:04:16.579] [amf_n1] [info] Received UL_NAS_DATA_IND

[2025-08-28 11:04:16.579] [amf_n1] [warning] No NAS context with amf_ue_ngap_id 1

[2025-08-28 11:04:16.579] [amf_n1] [warning] No existing nas_context with amf_ue_ngap_id 1

[2025-08-28 11:04:16.580] [amf_n1] [info] Associating SUPI (imsi-999700000000001) with NAS context

[2025-08-28 11:04:16.582] [amf_n1] [warning] No Optional IE 5GMMCapability available

[2025-08-28 11:04:16.582] [amf_sbi] [info] Receive UE Authentication Request message, handling ...

[2025-08-28 11:04:16.582] [amf_sbi] [info] Send HTTP message to http://oai-ausf:8080/nausf-auth/v1/ue-authentications

[2025-08-28 11:04:16.582] [amf_sbi] [info] HTTP message Body: {"servingNetworkName":"5G:mnc070.mcc999.3gppnetwork.org","supiOrSuci":"999700000000001"}

[2025-08-28 11:04:16.627] [amf_sbi] [info] Get response with HTTP code (201)

[2025-08-28 11:04:16.628] [amf_n1] [info] Links is: http://192.168.70.134:8080/nausf-auth/v1/ue-authentications/7eacb3390538800008e43cc01678b2d9/5g-aka-confirmation

[2025-08-28 11:04:16.628] [amf_n2] [info] Received Downlink NAS Transport message, handling

[2025-08-28 11:04:16.658] [sctp] [info] [Assoc_id 11, Socket 8] Received a message (length 64) from port 39115, on stream 1, PPID 60

[2025-08-28 11:04:16.658] [amf_n2] [info] Received Uplink NAS Transport message, handling

[2025-08-28 11:04:16.659] [amf_n1] [info] Received UL_NAS_DATA_IND

[2025-08-28 11:04:16.659] [amf_n1] [info] Found nas_context (0x7fd810000e50) with amf_ue_ngap_id (1)

[2025-08-28 11:04:16.660] [amf_n1] [info] resStar_s (59B0AD774FC99E8F2AE747C1450CECFD)

[2025-08-28 11:04:16.660] [amf_sbi] [info] Receive UE Authentication Confirmation message, handling ...

[2025-08-28 11:04:16.660] [amf_sbi] [info] Send HTTP message to http://192.168.70.134:8080/nausf-auth/v1/ue-authentications/7eacb3390538800008e43cc01678b2d9/5g-aka-confirmation

[2025-08-28 11:04:16.660] [amf_sbi] [info] HTTP message Body: {"resStar":"59B0AD774FC99E8F2AE747C1450CECFD"}

[2025-08-28 11:04:16.689] [amf_sbi] [info] Get response with HTTP code (200)

[2025-08-28 11:04:16.690] [amf_n2] [info] Received Downlink NAS Transport message, handling

[2025-08-28 11:04:16.720] [sctp] [info] [Assoc_id 11, Socket 8] Received a message (length 100) from port 39115, on stream 1, PPID 60

[2025-08-28 11:04:16.720] [amf_n2] [info] Received Uplink NAS Transport message, handling

[2025-08-28 11:04:16.721] [amf_n1] [info] Received UL_NAS_DATA_IND

[2025-08-28 11:04:16.722] [amf_sbi] [info] Receive Slice Selection Subscription Data Retrieval Request, handling ...

[2025-08-28 11:04:16.722] [amf_sbi] [info] Send HTTP message to http://oai-udm:8080/nudm-sdm/v1/999700000000001/nssai?plmn-id={"mcc":"999","mnc":"70"}

[2025-08-28 11:04:16.722] [amf_sbi] [info] HTTP message Body:

[2025-08-28 11:04:16.735] [amf_sbi] [info] Get response with HTTP code (500)

[2025-08-28 11:04:16.741] [amf_n1] [info] UE (IMSI 999700000000001, GUTI 9997001004100000001, current RAN ID 1, current AMF ID 1) has been registered to the network

[2025-08-28 11:04:16.741] [amf_n2] [info] Received Initial Context Setup Request message, handling

[2025-08-28 11:04:16.742] [amf_app] [info]

|------------------------------------------------------------------------------------------------------------------------------------------------------------|

|----------------------------------------------------------------------gNBs' Information---------------------------------------------------------------------|

| Index | Status | Global Id | gNB Name | PLMN |

| 1 | Connected | 0xE000 | gNB-Eurecom-CU | 999,70 |

|------------------------------------------------------------------------------------------------------------------------------------------------------------|

|-----------------------------------------------------------------------------------------------------------------------------------------------------------|

|---------------------------------------------------------------------UEs' Information----------------------------------------------------------------------|

| Index | 5GMM State | IMSI | GUTI | RAN UE NGAP ID | AMF UE NGAP ID | PLMN | Cell Id |

| 1 | 5GMM-REGISTERED | 999700000000001 | 9997001004100000001| 0x01 | 0x01 | 999,70 | 0xE00000 |

|-----------------------------------------------------------------------------------------------------------------------------------------------------------|

[2025-08-28 11:04:16.809] [sctp] [info] [Assoc_id 11, Socket 8] Received a message (length 38) from port 39115, on stream 1, PPID 60

[2025-08-28 11:04:16.809] [amf_n2] [info] Received UE Radio Capability Indication message, handling

[2025-08-28 11:04:16.809] [sctp] [info] [Assoc_id 11, Socket 8] Received a message (length 19) from port 39115, on stream 1, PPID 60

[2025-08-28 11:04:16.832] [sctp] [info] [Assoc_id 11, Socket 8] Received a message (length 53) from port 39115, on stream 1, PPID 60

[2025-08-28 11:04:16.832] [sctp] [info] [Assoc_id 11, Socket 8] Received a message (length 80) from port 39115, on stream 1, PPID 60

[2025-08-28 11:04:16.832] [amf_n2] [info] Received Uplink NAS Transport message, handling

[2025-08-28 11:04:16.832] [amf_n2] [info] Received Uplink NAS Transport message, handling

[2025-08-28 11:04:16.832] [amf_n1] [info] Received UL_NAS_DATA_IND

[2025-08-28 11:04:16.833] [amf_n1] [info] Received UL_NAS_DATA_IND

[2025-08-28 11:04:16.833] [amf_sbi] [info] Running ITTI_SMF_PDU_SESSION_CREATE_SM_CTX

[2025-08-28 11:04:16.833] [amf_sbi] [info] Find ue_context in amf_app using UE Context Key: app_ue_ranid_1:amfid_1

[2025-08-28 11:04:16.833] [amf_app] [warning] No PDU Session Context with PDU Session ID 10

[2025-08-28 11:04:16.833] [amf_sbi] [info] Send HTTP message to http://oai-nrf:8080/nnrf-disc/v1/nf-instances?target-nf-type=SMF&requester-nf-type=AMF

[2025-08-28 11:04:16.833] [amf_sbi] [info] HTTP message Body:

[2025-08-28 11:04:16.836] [amf_sbi] [info] Get response with HTTP code (200)

[2025-08-28 11:04:16.853] [amf_sbi] [info] JSON part {"smfServiceInstanceId":"7123e73f-926e-45e8-a953-bb49a5434a5f"}

[2025-08-28 11:04:16.853] [amf_sbi] [info] Location of the created SMF context: /nsmf-pdusession/v1/sm-contexts/1

[2025-08-28 11:04:16.863] [amf_server] [info] ue_context_id imsi-999700000000001

[2025-08-28 11:04:16.864] [amf_server] [info] Procedure n1-n2-messages

[2025-08-28 11:04:16.866] [amf_app] [info] Handle ITTI N1N2 Message Transfer Request

[2025-08-28 11:04:16.866] [amf_n1] [info] Received DOWNLINK_NAS_TRANSFER

[2025-08-28 11:04:16.867] [amf_n2] [info] Received PDU Session Resource Setup Request message, handling

[2025-08-28 11:04:16.896] [sctp] [info] [Assoc_id 11, Socket 8] Received a message (length 40) from port 39115, on stream 1, PPID 60

[2025-08-28 11:04:16.896] [amf_sbi] [info] Receive Nsmf_PDUSessionUpdateSMContext, handling ...

[2025-08-28 11:04:16.903] [amf_sbi] [info] JSON part {"cause":255}

[2025-08-28 11:04:17.620] [amf_app] [info]

|------------------------------------------------------------------------------------------------------------------------------------------------------------|

|----------------------------------------------------------------------gNBs' Information---------------------------------------------------------------------|

| Index | Status | Global Id | gNB Name | PLMN |

| 1 | Connected | 0xE000 | gNB-Eurecom-CU | 999,70 |

|------------------------------------------------------------------------------------------------------------------------------------------------------------|

|-----------------------------------------------------------------------------------------------------------------------------------------------------------|

|---------------------------------------------------------------------UEs' Information----------------------------------------------------------------------|

| Index | 5GMM State | IMSI | GUTI | RAN UE NGAP ID | AMF UE NGAP ID | PLMN | Cell Id |

| 1 | 5GMM-REGISTERED | 999700000000001 | 9997001004100000001| 0x01 | 0x01 | 999,70 | 0xE00000 |

|-----------------------------------------------------------------------------------------------------------------------------------------------------------|

[2025-08-28 11:04:17.859] [amf_sbi] [info] Receive Update NF Instance Request, handling ...

[2025-08-28 11:04:17.859] [amf_sbi] [info] Send HTTP message to http://oai-nrf:8080/nnrf-nfm/v1/nf-instances/ad3dea39-51e9-443c-9607-6b89ff53c3df

[2025-08-28 11:04:17.859] [amf_sbi] [info] HTTP message Body: [{"op":"replace","path":"/nfStatus","value":"REGISTERED"}]

[2025-08-28 11:04:17.861] [amf_sbi] [info] Get response with HTTP code (204)

[2025-08-28 11:04:17.862] [amf_sbi] [info] Could not get JSON content from the response

[2025-08-28 11:04:27.862] [amf_sbi] [info] Receive Update NF Instance Request, handling ...

[2025-08-28 11:04:27.862] [amf_sbi] [info] Send HTTP message to http://oai-nrf:8080/nnrf-nfm/v1/nf-instances/ad3dea39-51e9-443c-9607-6b89ff53c3df

[2025-08-28 11:04:27.862] [amf_sbi] [info] HTTP message Body: [{"op":"replace","path":"/nfStatus","value":"REGISTERED"}]

[2025-08-28 11:04:27.865] [amf_sbi] [info] Get response with HTTP code (204)

[2025-08-28 11:04:27.865] [amf_sbi] [info] Could not get JSON content from the response

[2025-08-28 11:04:37.620] [amf_app] [info]

|------------------------------------------------------------------------------------------------------------------------------------------------------------|

|----------------------------------------------------------------------gNBs' Information---------------------------------------------------------------------|

| Index | Status | Global Id | gNB Name | PLMN |

| 1 | Connected | 0xE000 | gNB-Eurecom-CU | 999,70 |

|------------------------------------------------------------------------------------------------------------------------------------------------------------|

|-----------------------------------------------------------------------------------------------------------------------------------------------------------|

|---------------------------------------------------------------------UEs' Information----------------------------------------------------------------------|

| Index | 5GMM State | IMSI | GUTI | RAN UE NGAP ID | AMF UE NGAP ID | PLMN | Cell Id |

| 1 | 5GMM-REGISTERED | 999700000000001 | 9997001004100000001| 0x01 | 0x01 | 999,70 | 0xE00000 |

|-----------------------------------------------------------------------------------------------------------------------------------------------------------|

[2025-08-28 11:04:37.866] [amf_sbi] [info] Receive Update NF Instance Request, handling ...

[2025-08-28 11:04:37.866] [amf_sbi] [info] Send HTTP message to http://oai-nrf:8080/nnrf-nfm/v1/nf-instances/ad3dea39-51e9-443c-9607-6b89ff53c3df

[2025-08-28 11:04:37.866] [amf_sbi] [info] HTTP message Body: [{"op":"replace","path":"/nfStatus","value":"REGISTERED"}]

[2025-08-28 11:04:37.868] [amf_sbi] [info] Get response with HTTP code (204)

[2025-08-28 11:04:37.868] [amf_sbi] [info] Could not get JSON content from the response

[2025-08-28 11:04:47.869] [amf_sbi] [info] Receive Update NF Instance Request, handling ...

[2025-08-28 11:04:47.869] [amf_sbi] [info] Send HTTP message to http://oai-nrf:8080/nnrf-nfm/v1/nf-instances/ad3dea39-51e9-443c-9607-6b89ff53c3df

[2025-08-28 11:04:47.869] [amf_sbi] [info] HTTP message Body: [{"op":"replace","path":"/nfStatus","value":"REGISTERED"}]

[2025-08-28 11:04:47.872] [amf_sbi] [info] Get response with HTTP code (204)

[2025-08-28 11:04:47.873] [amf_sbi] [info] Could not get JSON content from the response

[2025-08-28 11:04:57.620] [amf_app] [info]

|------------------------------------------------------------------------------------------------------------------------------------------------------------|

|----------------------------------------------------------------------gNBs' Information---------------------------------------------------------------------|

| Index | Status | Global Id | gNB Name | PLMN |

| 1 | Connected | 0xE000 | gNB-Eurecom-CU | 999,70 |

|------------------------------------------------------------------------------------------------------------------------------------------------------------|

|-----------------------------------------------------------------------------------------------------------------------------------------------------------|

|---------------------------------------------------------------------UEs' Information----------------------------------------------------------------------|

| Index | 5GMM State | IMSI | GUTI | RAN UE NGAP ID | AMF UE NGAP ID | PLMN | Cell Id |

| 1 | 5GMM-REGISTERED | 999700000000001 | 9997001004100000001| 0x01 | 0x01 | 999,70 | 0xE00000 |

|-----------------------------------------------------------------------------------------------------------------------------------------------------------|

[2025-08-28 11:04:57.873] [amf_sbi] [info] Receive Update NF Instance Request, handling ...

[2025-08-28 11:04:57.873] [amf_sbi] [info] Send HTTP message to http://oai-nrf:8080/nnrf-nfm/v1/nf-instances/ad3dea39-51e9-443c-9607-6b89ff53c3df

[2025-08-28 11:04:57.873] [amf_sbi] [info] HTTP message Body: [{"op":"replace","path":"/nfStatus","value":"REGISTERED"}]

[2025-08-28 11:04:57.876] [amf_sbi] [info] Get response with HTTP code (204)

[2025-08-28 11:04:57.876] [amf_sbi] [info] Could not get JSON content from the response

[2025-08-28 11:05:07.876] [amf_sbi] [info] Receive Update NF Instance Request, handling ...

[2025-08-28 11:05:07.877] [amf_sbi] [info] Send HTTP message to http://oai-nrf:8080/nnrf-nfm/v1/nf-instances/ad3dea39-51e9-443c-9607-6b89ff53c3df

[2025-08-28 11:05:07.877] [amf_sbi] [info] HTTP message Body: [{"op":"replace","path":"/nfStatus","value":"REGISTERED"}]

[2025-08-28 11:05:07.880] [amf_sbi] [info] Get response with HTTP code (204)

[2025-08-28 11:05:07.880] [amf_sbi] [info] Could not get JSON content from the response

[2025-08-28 11:05:17.620] [amf_app] [info]

|------------------------------------------------------------------------------------------------------------------------------------------------------------|

|----------------------------------------------------------------------gNBs' Information---------------------------------------------------------------------|

| Index | Status | Global Id | gNB Name | PLMN |

| 1 | Connected | 0xE000 | gNB-Eurecom-CU | 999,70 |

|------------------------------------------------------------------------------------------------------------------------------------------------------------|

|-----------------------------------------------------------------------------------------------------------------------------------------------------------|

|---------------------------------------------------------------------UEs' Information----------------------------------------------------------------------|

| Index | 5GMM State | IMSI | GUTI | RAN UE NGAP ID | AMF UE NGAP ID | PLMN | Cell Id |

| 1 | 5GMM-REGISTERED | 999700000000001 | 9997001004100000001| 0x01 | 0x01 | 999,70 | 0xE00000 |

|-----------------------------------------------------------------------------------------------------------------------------------------------------------|

[2025-08-28 11:05:17.880] [amf_sbi] [info] Receive Update NF Instance Request, handling ...

[2025-08-28 11:05:17.880] [amf_sbi] [info] Send HTTP message to http://oai-nrf:8080/nnrf-nfm/v1/nf-instances/ad3dea39-51e9-443c-9607-6b89ff53c3df

[2025-08-28 11:05:17.880] [amf_sbi] [info] HTTP message Body: [{"op":"replace","path":"/nfStatus","value":"REGISTERED"}]

[2025-08-28 11:05:17.884] [amf_sbi] [info] Get response with HTTP code (204)

[2025-08-28 11:05:17.884] [amf_sbi] [info] Could not get JSON content from the response

[2025-08-28 11:05:27.884] [amf_sbi] [info] Receive Update NF Instance Request, handling ...

[2025-08-28 11:05:27.884] [amf_sbi] [info] Send HTTP message to http://oai-nrf:8080/nnrf-nfm/v1/nf-instances/ad3dea39-51e9-443c-9607-6b89ff53c3df

[2025-08-28 11:05:27.884] [amf_sbi] [info] HTTP message Body: [{"op":"replace","path":"/nfStatus","value":"REGISTERED"}]

[2025-08-28 11:05:27.887] [amf_sbi] [info] Get response with HTTP code (204)

[2025-08-28 11:05:27.888] [amf_sbi] [info] Could not get JSON content from the response

telcomaan@oai-5gc-vm:~/oai-cn5g-fed/docker-compose$ docker logs oai-smf

[2025-08-28 11:00:58.078] [smf_app] [start] Options parsed

[2025-08-28 11:00:58.079] [config ] [info] Reading NF configuration from /openair-smf/etc/config.yaml

[2025-08-28 11:00:58.126] [config ] [debug] Unknown NF upf in configuration. Ignored

[2025-08-28 11:00:58.126] [config ] [debug] Unknown NF udr in configuration. Ignored

[2025-08-28 11:00:58.126] [config ] [debug] Unknown NF ausf in configuration. Ignored

[2025-08-28 11:00:58.143] [config ] [debug] Validating configuration of log_level

[2025-08-28 11:00:58.159] [config ] [info] ==== OPENAIRINTERFACE smf vBranch: HEAD Abrev. Hash: 531b6d87 Date: Tue Jul 30 07:16:24 2024 +0000 ====

[2025-08-28 11:00:58.160] [config ] [info] Basic Configuration:

[2025-08-28 11:00:58.160] [config ] [info] - log_level..................................: info

[2025-08-28 11:00:58.160] [config ] [info] - register_nf................................: Yes

[2025-08-28 11:00:58.160] [config ] [info] - http_version...............................: 2

[2025-08-28 11:00:58.160] [config ] [info] - HTTP Request Timeout.......................: 3000 (ms)

[2025-08-28 11:00:58.160] [config ] [info] SMF Config:

[2025-08-28 11:00:58.160] [config ] [info] - host.......................................: oai-smf

[2025-08-28 11:00:58.160] [config ] [info] - sbi

[2025-08-28 11:00:58.160] [config ] [info] + URL......................................: http://oai-smf:8080

[2025-08-28 11:00:58.160] [config ] [info] + API Version..............................: v1

[2025-08-28 11:00:58.160] [config ] [info] + IPv4 Address ............................: 192.168.70.136

[2025-08-28 11:00:58.160] [config ] [info] - n4

[2025-08-28 11:00:58.160] [config ] [info] + Port.....................................: 8805

[2025-08-28 11:00:58.160] [config ] [info] + IPv4 Address ............................: 192.168.70.136

[2025-08-28 11:00:58.160] [config ] [info] + MTU......................................: 1500

[2025-08-28 11:00:58.160] [config ] [info] + Interface name: .........................: eth0

[2025-08-28 11:00:58.160] [config ] [info] supported_features:

[2025-08-28 11:00:58.160] [config ] [info] + use_local_subscription_info..............: Yes

[2025-08-28 11:00:58.160] [config ] [info] + use_local_pcc_rules......................: Yes

[2025-08-28 11:00:58.160] [config ] [info] + use_external_ausf........................: No

[2025-08-28 11:00:58.160] [config ] [info] + use_external_udm.........................: No

[2025-08-28 11:00:58.160] [config ] [info] + use_external_nssf........................: No

[2025-08-28 11:00:58.160] [config ] [info] - ue_mtu.....................................: 1500

[2025-08-28 11:00:58.160] [config ] [info] - p-cscf_ipv4................................: 127.0.0.1

[2025-08-28 11:00:58.160] [config ] [info] - p-cscf_ipv6................................: fe80::7915:f408:1787:db8b

[2025-08-28 11:00:58.160] [config ] [info] UPF List:

[2025-08-28 11:00:58.160] [config ] [info] + oai-upf

[2025-08-28 11:00:58.160] [config ] [info] + host...................................: oai-upf

[2025-08-28 11:00:58.160] [config ] [info] + port...................................: 8805

[2025-08-28 11:00:58.160] [config ] [info] + enable_usage_reporting.................: No

[2025-08-28 11:00:58.160] [config ] [info] + enable_dl_pdr_in_session_establishment.: No

[2025-08-28 11:00:58.160] [config ] [info] Local Subscription Infos:

[2025-08-28 11:00:58.160] [config ] [info] - local_subscription_info

[2025-08-28 11:00:58.160] [config ] [info] + dnn....................................: internet

[2025-08-28 11:00:58.160] [config ] [info] + ssc_mode...............................: 1

[2025-08-28 11:00:58.160] [config ] [info] + snssai:

[2025-08-28 11:00:58.160] [config ] [info] - sst..................................: 1

[2025-08-28 11:00:58.160] [config ] [info] - sd...................................: FFFFFF

[2025-08-28 11:00:58.160] [config ] [info] + qos_profile:

[2025-08-28 11:00:58.160] [config ] [info] - 5qi..................................: 9

[2025-08-28 11:00:58.160] [config ] [info] - priority.............................: 1

[2025-08-28 11:00:58.160] [config ] [info] - arp_priority.........................: 1

[2025-08-28 11:00:58.160] [config ] [info] - arp_preempt_vulnerability............: NOT_PREEMPTABLE

[2025-08-28 11:00:58.160] [config ] [info] - arp_preempt_capability...............: NOT_PREEMPT

[2025-08-28 11:00:58.160] [config ] [info] - session_ambr_dl......................: 400Mbps

[2025-08-28 11:00:58.160] [config ] [info] - session_ambr_ul......................: 200Mbps

[2025-08-28 11:00:58.160] [config ] [info] + smf_info:

[2025-08-28 11:00:58.160] [config ] [info] - snssai_smf_info_item:

[2025-08-28 11:00:58.160] [config ] [info] + snssai:

[2025-08-28 11:00:58.160] [config ] [info] - sst..................................: 1

[2025-08-28 11:00:58.160] [config ] [info] - sd...................................: FFFFFF

[2025-08-28 11:00:58.160] [config ] [info] + dnns:

[2025-08-28 11:00:58.160] [config ] [info] - dnn..................................: internet

[2025-08-28 11:00:58.160] [config ] [info] Peer NF Configuration:

[2025-08-28 11:00:58.160] [config ] [info] nrf:

[2025-08-28 11:00:58.160] [config ] [info] - host.....................................: oai-nrf

[2025-08-28 11:00:58.160] [config ] [info] - sbi

[2025-08-28 11:00:58.160] [config ] [info] + URL....................................: http://oai-nrf:8080

[2025-08-28 11:00:58.160] [config ] [info] + API Version............................: v1

[2025-08-28 11:00:58.160] [config ] [info] DNNs:

[2025-08-28 11:00:58.160] [config ] [info] - DNN:

[2025-08-28 11:00:58.160] [config ] [info] + DNN......................................: internet

[2025-08-28 11:00:58.160] [config ] [info] + PDU session type.........................: IPV4

[2025-08-28 11:00:58.160] [config ] [info] + IPv4 subnet..............................: 12.1.2.0/24

[2025-08-28 11:00:58.160] [config ] [info] + DNS Settings:

[2025-08-28 11:00:58.160] [config ] [info] - primary_dns_ipv4.......................: 172.21.3.100

[2025-08-28 11:00:58.160] [config ] [info] - primary_dns_ipv6.......................: 2001:4860:4860::8888

[2025-08-28 11:00:58.160] [config ] [info] - secondary_dns_ipv4.....................: 8.8.8.8

[2025-08-28 11:00:58.160] [config ] [info] - secondary_dns_ipv6.....................: 2001:4860:4860::8888

[2025-08-28 11:00:58.160] [itti ] [start] Starting...

[2025-08-28 11:00:58.161] [itti ] [start] Started

[2025-08-28 11:00:58.161] [smf_sbi] [info] HTTP Client successfully initiated on interface eth0 with timeout 3000 ms, HTTP version 2

[2025-08-28 11:00:58.161] [async ] [start] Starting...

[2025-08-28 11:00:58.161] [itti ] [info] Starting timer_manager_task

[2025-08-28 11:00:58.161] [itti ] [warning] Could not set schedparam to ITTI task 0, err=1

[2025-08-28 11:00:58.161] [async ] [warning] Could not set schedparam to ITTI task 1, err=1

[2025-08-28 11:00:58.162] [async ] [start] Started

[2025-08-28 11:00:58.162] [smf_app] [start] Starting...

[2025-08-28 11:00:58.162] [smf_app] [info] Apply config...

[2025-08-28 11:00:58.163] [smf_app] [info] Applied config internet

[2025-08-28 11:00:58.163] [smf_app] [info] PAA Ipv4: 12.1.2.2

[2025-08-28 11:00:58.163] [smf_app] [info] Applied config

[2025-08-28 11:00:58.165] [pfcp ] [info] pfcp_l4_stack created listening to 192.168.70.136:8805

[2025-08-28 11:00:58.167] [udp ] [warning] Could not set schedparam to ITTI task 6, err=1

[2025-08-28 11:00:58.167] [udp ] [warning] Could not set schedparam to ITTI task 6, err=1

[2025-08-28 11:00:58.167] [smf_n4 ] [start] Starting...

[2025-08-28 11:00:58.169] [smf_n4 ] [start] Started

[2025-08-28 11:00:58.169] [smf_sbi] [start] Starting...

[2025-08-28 11:00:58.171] [smf_sbi] [start] Started

[2025-08-28 11:00:58.171] [smf_app] [start] Started

[2025-08-28 11:00:58.187] [smf_api] [info] HTTP2 server being started

[2025-08-28 11:00:59.205] [smf_api] [info] NFStatusNotifyApiImpl, received a NF status notification...

[2025-08-28 11:00:59.205] [smf_app] [info] Handle a NF status notification from NRF (HTTP version 2)

[2025-08-28 11:00:59.217] [smf_n4 ] [info] handle_receive(53 bytes)

[2025-08-28 11:00:59.217] [smf_n4 ] [info] Received N4 ASSOCIATION SETUP RESPONSE from an UPF

[2025-08-28 11:00:59.217] [smf_n4 ] [info] Received N4 ASSOCIATION SETUP RESPONSE

[2025-08-28 11:00:59.220] [smf_app] [info] Successfully added UPF node: 192.168.70.138

[2025-08-28 11:01:04.216] [smf_n4 ] [info] TIME-OUT event timer id 2

[2025-08-28 11:01:09.220] [smf_n4 ] [info] TIME-OUT event timer id 3

[2025-08-28 11:01:09.220] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2319886528 starting

[2025-08-28 11:01:09.222] [smf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:01:11.217] [smf_app] [info] TIME-OUT event timer id 4

[2025-08-28 11:01:14.224] [smf_n4 ] [info] TIME-OUT event timer id 7

[2025-08-28 11:01:19.222] [smf_n4 ] [info] TIME-OUT event timer id 8

[2025-08-28 11:01:19.223] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2319886528 starting

[2025-08-28 11:01:19.223] [smf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:01:21.217] [smf_app] [info] TIME-OUT event timer id 9

[2025-08-28 11:01:24.223] [smf_n4 ] [info] TIME-OUT event timer id 12

[2025-08-28 11:01:29.224] [smf_n4 ] [info] TIME-OUT event timer id 13

[2025-08-28 11:01:29.224] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2319886528 starting

[2025-08-28 11:01:29.224] [smf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:01:31.218] [smf_app] [info] TIME-OUT event timer id 14

[2025-08-28 11:01:34.224] [smf_n4 ] [info] TIME-OUT event timer id 17

[2025-08-28 11:01:39.225] [smf_n4 ] [info] TIME-OUT event timer id 18

[2025-08-28 11:01:39.225] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2319886528 starting

[2025-08-28 11:01:39.225] [smf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:01:41.218] [smf_app] [info] TIME-OUT event timer id 19

[2025-08-28 11:01:44.225] [smf_n4 ] [info] TIME-OUT event timer id 22

[2025-08-28 11:01:49.226] [smf_n4 ] [info] TIME-OUT event timer id 23

[2025-08-28 11:01:49.226] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2319886528 starting

[2025-08-28 11:01:49.226] [smf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:01:51.219] [smf_app] [info] TIME-OUT event timer id 24

[2025-08-28 11:01:54.226] [smf_n4 ] [info] TIME-OUT event timer id 27

[2025-08-28 11:01:59.227] [smf_n4 ] [info] TIME-OUT event timer id 28

[2025-08-28 11:01:59.227] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2319886528 starting

[2025-08-28 11:01:59.227] [smf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:02:01.219] [smf_app] [info] TIME-OUT event timer id 29

[2025-08-28 11:02:04.227] [smf_n4 ] [info] TIME-OUT event timer id 32

[2025-08-28 11:02:09.228] [smf_n4 ] [info] TIME-OUT event timer id 33

[2025-08-28 11:02:09.228] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2319886528 starting

[2025-08-28 11:02:09.228] [smf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:02:11.219] [smf_app] [info] TIME-OUT event timer id 34

[2025-08-28 11:02:14.228] [smf_n4 ] [info] TIME-OUT event timer id 37

[2025-08-28 11:02:19.228] [smf_n4 ] [info] TIME-OUT event timer id 38

[2025-08-28 11:02:19.228] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2319886528 starting

[2025-08-28 11:02:19.229] [smf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:02:21.219] [smf_app] [info] TIME-OUT event timer id 39

[2025-08-28 11:02:24.229] [smf_n4 ] [info] TIME-OUT event timer id 42

[2025-08-28 11:02:29.230] [smf_n4 ] [info] TIME-OUT event timer id 43

[2025-08-28 11:02:29.230] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2319886528 starting

[2025-08-28 11:02:29.232] [smf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:02:31.220] [smf_app] [info] TIME-OUT event timer id 44

[2025-08-28 11:02:34.231] [smf_n4 ] [info] TIME-OUT event timer id 47

[2025-08-28 11:02:39.234] [smf_n4 ] [info] TIME-OUT event timer id 48

[2025-08-28 11:02:39.234] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2319886528 starting

[2025-08-28 11:02:39.234] [smf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:02:41.220] [smf_app] [info] TIME-OUT event timer id 49

[2025-08-28 11:02:44.234] [smf_n4 ] [info] TIME-OUT event timer id 52

[2025-08-28 11:02:49.234] [smf_n4 ] [info] TIME-OUT event timer id 53

[2025-08-28 11:02:49.234] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2319886528 starting

[2025-08-28 11:02:49.235] [smf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:02:51.220] [smf_app] [info] TIME-OUT event timer id 54

[2025-08-28 11:02:54.235] [smf_n4 ] [info] TIME-OUT event timer id 57

[2025-08-28 11:02:59.235] [smf_n4 ] [info] TIME-OUT event timer id 58

[2025-08-28 11:02:59.236] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2319886528 starting

[2025-08-28 11:02:59.236] [smf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:03:01.221] [smf_app] [info] TIME-OUT event timer id 59

[2025-08-28 11:03:04.236] [smf_n4 ] [info] TIME-OUT event timer id 62

[2025-08-28 11:03:09.236] [smf_n4 ] [info] TIME-OUT event timer id 63

[2025-08-28 11:03:09.236] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2319886528 starting

[2025-08-28 11:03:09.237] [smf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:03:11.221] [smf_app] [info] TIME-OUT event timer id 64

[2025-08-28 11:03:14.237] [smf_n4 ] [info] TIME-OUT event timer id 67

[2025-08-28 11:03:19.237] [smf_n4 ] [info] TIME-OUT event timer id 68

[2025-08-28 11:03:19.237] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2319886528 starting

[2025-08-28 11:03:19.238] [smf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:03:21.221] [smf_app] [info] TIME-OUT event timer id 69

[2025-08-28 11:03:24.237] [smf_n4 ] [info] TIME-OUT event timer id 72

[2025-08-28 11:03:29.238] [smf_n4 ] [info] TIME-OUT event timer id 73

[2025-08-28 11:03:29.238] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2319886528 starting

[2025-08-28 11:03:29.238] [smf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:03:31.221] [smf_app] [info] TIME-OUT event timer id 74

[2025-08-28 11:03:34.238] [smf_n4 ] [info] TIME-OUT event timer id 77

[2025-08-28 11:03:39.239] [smf_n4 ] [info] TIME-OUT event timer id 78

[2025-08-28 11:03:39.239] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2319886528 starting

[2025-08-28 11:03:39.240] [smf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:03:41.222] [smf_app] [info] TIME-OUT event timer id 79

[2025-08-28 11:03:44.239] [smf_n4 ] [info] TIME-OUT event timer id 82

[2025-08-28 11:03:49.240] [smf_n4 ] [info] TIME-OUT event timer id 83

[2025-08-28 11:03:49.240] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2319886528 starting

[2025-08-28 11:03:49.241] [smf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:03:51.222] [smf_app] [info] TIME-OUT event timer id 84

[2025-08-28 11:03:54.241] [smf_n4 ] [info] TIME-OUT event timer id 87

[2025-08-28 11:03:59.241] [smf_n4 ] [info] TIME-OUT event timer id 88

[2025-08-28 11:03:59.242] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2319886528 starting

[2025-08-28 11:03:59.242] [smf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:04:01.222] [smf_app] [info] TIME-OUT event timer id 89

[2025-08-28 11:04:04.242] [smf_n4 ] [info] TIME-OUT event timer id 92

[2025-08-28 11:04:09.242] [smf_n4 ] [info] TIME-OUT event timer id 93

[2025-08-28 11:04:09.242] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2319886528 starting

[2025-08-28 11:04:09.243] [smf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:04:11.222] [smf_app] [info] TIME-OUT event timer id 94

[2025-08-28 11:04:14.243] [smf_n4 ] [info] TIME-OUT event timer id 97

[2025-08-28 11:04:16.837] [smf_api] [info] Received a SM context create request from AMF.

[2025-08-28 11:04:16.837] [smf_api] [info] Handle PDU Session Create SM Context Request.

[2025-08-28 11:04:16.840] [smf_app] [warning] No SelMode available

[2025-08-28 11:04:16.840] [smf_app] [info] Handle a PDU Session Create SM Context Request from an AMF (HTTP version 2)

[2025-08-28 11:04:16.840] [smf_n1 ] [info] Decode NAS message from N1 SM Container.

[2025-08-28 11:04:16.845] [smf_app] [info] Handle a PDU Session Create SM Context Request message from AMF, SUPI 999700000000001, - snssai:

+ sst........................................: 1

+ sd.........................................: FFFFFF

[2025-08-28 11:04:16.847] [smf_app] [info] Inserted DNN Subscription, key: 4294967041 dnn internet

- snssai:

+ sst........................................: 1

+ sd.........................................: FFFFFF

[2025-08-28 11:04:16.847] [smf_app] [info] Handle a PDU Session Create SM Context Request message from AMF (HTTP version 2)

[2025-08-28 11:04:16.847] [smf_app] [info] Find PDU Session with ID 10

[2025-08-28 11:04:16.847] [smf_app] [info] Add PDU Session with Id 10

[2025-08-28 11:04:16.848] [smf_n7 ] [warning] Local PCC rules are not supported yet

[2025-08-28 11:04:16.848] [smf_n7 ] [info] PCF SM Policy Association Creation was not successful. Continue using default rules

[2025-08-28 11:04:16.848] [smf_app] [info] Find a DNN Subscription with key: 4294967041, map size 1 and

- snssai:

+ sst........................................: 1

+ sd.........................................: FFFFFF

[2025-08-28 11:04:16.848] [smf_app] [info] Find DNN configuration with DNN internet

[2025-08-28 11:04:16.849] [smf_app] [info] PAA, Ipv4 Address: 12.1.2.2

[2025-08-28 11:04:16.849] [smf_app] [info] Create a procedure to process this message.

[2025-08-28 11:04:16.849] [smf_app] [info] Perform a procedure - Create SM Context Request

[2025-08-28 11:04:16.849] [smf_app] [info] Get default QoS for a PDU Session, key 1

[2025-08-28 11:04:16.849] [smf_app] [info] Find a DNN Subscription with key: 4294967041, map size 1 and

- snssai:

+ sst........................................: 1

+ sd.........................................: FFFFFF

[2025-08-28 11:04:16.849] [smf_app] [info] Find DNN configuration with DNN internet

[2025-08-28 11:04:16.849] [smf_app] [info] Find PDU Session with ID 10

[2025-08-28 11:04:16.849] [smf_app] [info] Verifying if UPF edge serves network

[2025-08-28 11:04:16.849] [smf_app] [info] Successfully added UPF node: 192.168.70.138

[2025-08-28 11:04:16.849] [smf_app] [info] Verifying if UPF edge serves network

[2025-08-28 11:04:16.849] [smf_app] [info] UPF selection was successful.

[2025-08-28 11:04:16.851] [smf_app] [info] Sending ITTI message 37itti_n4_session_establishment_request to task TASK_SMF_N4

[2025-08-28 11:04:16.853] [smf_n4 ] [info] handle_receive(70 bytes)

[2025-08-28 11:04:16.853] [smf_n1 ] [info] Create N1 SM Container, PDU Session Establishment Accept

[2025-08-28 11:04:16.853] [smf_n1 ] [info] PDU_SESSION_ESTABLISHMENT_ACCEPT, encode starting...

[2025-08-28 11:04:16.853] [smf_app] [info] Find a DNN Subscription with key: 4294967041, map size 1 and

- snssai:

+ sst........................................: 1

+ sd.........................................: FFFFFF

[2025-08-28 11:04:16.853] [smf_app] [info] Find DNN configuration with DNN internet

[2025-08-28 11:04:16.853] [smf_app] [info] Get default QoS Flow Description (PDU session type 1)

[2025-08-28 11:04:16.853] [smf_n1 ] [info] Encode PDU Session Establishment Accept

[2025-08-28 11:04:16.854] [smf_n2 ] [info] Create N2 SM Information, PDU Session Resource Setup Request Transfer

[2025-08-28 11:04:16.854] [smf_n2 ] [info] QoS parameters: QFI 1, Priority level 1, ARP priority level 1

[2025-08-28 11:04:16.856] [smf_app] [info] Find a DNN Subscription with key: 4294967041, map size 1 and

- snssai:

+ sst........................................: 1

+ sd.........................................: FFFFFF

[2025-08-28 11:04:16.856] [smf_app] [info] Find DNN configuration with DNN internet

[2025-08-28 11:04:16.856] [smf_n2 ] [info] QoS parameters: QFI 1, ARP priority level 1, qos_flow.qos_profile.arp.preempt_cap NOT_PREEMPT, qos_flow.qos_profile.arp.preempt_vuln NOT_PREEMPTABLE

[2025-08-28 11:04:16.859] [smf_app] [info] Sending ITTI message N11_SESSION_CREATE_SM_CONTEXT_RESPONSE to task TASK_SMF_APP

[2025-08-28 11:04:16.867] [smf_app] [info] Process N1N2MessageTransfer Response

[2025-08-28 11:04:16.867] [smf_app] [info] PDU_SESSION_ESTABLISHMENT_UE_REQUESTED

[2025-08-28 11:04:16.867] [smf_app] [info] Update PDU Session Status

[2025-08-28 11:04:16.867] [smf_app] [info] Find PDU Session with ID 10

[2025-08-28 11:04:16.867] [smf_app] [info] Set PDU Session Status to PDU_SESSION_ESTABLISHMENT_PENDING

[2025-08-28 11:04:16.868] [smf_app] [info] Set PDU Session Status to PDU_SESSION_ESTABLISHMENT_PENDING

[2025-08-28 11:04:16.868] [smf_app] [info] Update UpCnx_State

[2025-08-28 11:04:16.868] [smf_app] [info] Find PDU Session with ID 10

[2025-08-28 11:04:16.868] [smf_app] [info] Set upCnxState to UPCNX_STATE_DEACTIVATED

[2025-08-28 11:04:16.868] [smf_app] [info] Set PDU Session UpCnxState to UPCNX_STATE_DEACTIVATED

[2025-08-28 11:04:16.897] [smf_api] [info] Received a SM context update request from AMF.

[2025-08-28 11:04:16.897] [smf_api] [info] smf_ref 1, method modify

[2025-08-28 11:04:16.897] [smf_api] [info] Handle Update SM Context Request from AMF

[2025-08-28 11:04:16.899] [smf_api] [info] Handle PDU Session Update SM Context Request.

[2025-08-28 11:04:16.899] [smf_api] [info] Received a PDUSession_UpdateSMContext Request from AMF.

[2025-08-28 11:04:16.899] [smf_app] [info] Handle a PDU Session Update SM Context Request from an AMF (HTTP version 2)

[2025-08-28 11:04:16.899] [smf_app] [info] Find PDU Session with ID 10

[2025-08-28 11:04:16.899] [smf_app] [info] Handle a PDU Session Update SM Context Request message from an AMF (HTTP version 2)

[2025-08-28 11:04:16.899] [smf_app] [info] Find PDU Session with ID 10

[2025-08-28 11:04:16.899] [smf_app] [info] PDU Session Resource Setup Response Transfer

[2025-08-28 11:04:16.899] [smf_n2 ] [info] Decode NGAP message (PDUSessionResourceSetupResponseTransfer) from N2 SM Information

[2025-08-28 11:04:16.899] [smf_app] [info] PDU Session Establishment Request, processing N2 SM Information

[2025-08-28 11:04:16.899] [smf_app] [info] Perform a procedure - Update SM Context Request

[2025-08-28 11:04:16.899] [smf_app] [info] Find PDU Session with ID 10

[2025-08-28 11:04:16.899] [smf_app] [info] Sending ITTI message 36itti_n4_session_modification_request to task TASK_SMF_N4

[2025-08-28 11:04:16.901] [smf_n4 ] [info] handle_receive(31 bytes)

[2025-08-28 11:04:16.901] [smf_app] [info] Handle N4 Session Modification Response (PDU Session Id 10)

[2025-08-28 11:04:16.901] [smf_app] [info] PDU Session Establishment Request (UE-Initiated)

[2025-08-28 11:04:16.901] [smf_app] [info] Set PDU Session Status to PDU_SESSION_ACTIVE

[2025-08-28 11:04:16.902] [smf_app] [info] Set upCnxState to UPCNX_STATE_ACTIVATED

[2025-08-28 11:04:16.902] [smf_app] [info] SMF context:

SMF CONTEXT:

SUPI: 999700000000001

PDU SESSION:

PDU Session ID: 10

DNN: internet

S-NSSAI: SST=1, SD=FFFFFF

PDN type: IPV4

PAA IPv4: 12.1.2.2

Default QFI: No QFI available

SEID: 1

N3:

- UPF Graph Edge

+ Interface Type.............................: N3

+ NWI........................................:

+ Uplink.....................................: No

+ PDR ID.....................................: 1

+ FAR ID.....................................: 2

[2025-08-28 11:04:16.902] [smf_app] [info] Find PDU Session with ID 10

[2025-08-28 11:04:16.902] [smf_app] [info] Find PDU Session with ID 10

[2025-08-28 11:04:16.902] [smf_app] [info] Find PDU Session with ID 10

[2025-08-28 11:04:16.902] [smf_app] [info] Find PDU Session with ID 10

[2025-08-28 11:04:16.902] [smf_app] [info] Find PDU Session with ID 10

[2025-08-28 11:04:16.902] [smf_app] [info] Find PDU Session with ID 10

[2025-08-28 11:04:16.902] [smf_app] [info] Find PDU Session with ID 10

[2025-08-28 11:04:16.902] [smf_app] [info] Find PDU Session with ID 10

[2025-08-28 11:04:16.902] [smf_app] [info] Sending ITTI message N11_SESSION_UPDATE_SM_CONTEXT_RESPONSE to task TASK_SMF_APP

[2025-08-28 11:04:16.902] [smf_app] [info] Handle N4 Session Modification Response

[2025-08-28 11:04:19.244] [smf_n4 ] [info] TIME-OUT event timer id 98

[2025-08-28 11:04:19.244] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2319886528 starting

[2025-08-28 11:04:19.244] [smf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:04:21.223] [smf_app] [info] TIME-OUT event timer id 99

[2025-08-28 11:04:21.852] [smf_n4 ] [info] TIME-OUT event timer id 101

[2025-08-28 11:04:21.900] [smf_n4 ] [info] TIME-OUT event timer id 103

[2025-08-28 11:04:24.244] [smf_n4 ] [info] TIME-OUT event timer id 106

[2025-08-28 11:04:29.244] [smf_n4 ] [info] TIME-OUT event timer id 107

[2025-08-28 11:04:29.245] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2319886528 starting

[2025-08-28 11:04:29.245] [smf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:04:31.223] [smf_app] [info] TIME-OUT event timer id 108

[2025-08-28 11:04:34.245] [smf_n4 ] [info] TIME-OUT event timer id 111

[2025-08-28 11:04:39.245] [smf_n4 ] [info] TIME-OUT event timer id 112

[2025-08-28 11:04:39.245] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2319886528 starting

[2025-08-28 11:04:39.246] [smf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:04:41.228] [smf_app] [info] TIME-OUT event timer id 113

[2025-08-28 11:04:44.246] [smf_n4 ] [info] TIME-OUT event timer id 116

[2025-08-28 11:04:49.246] [smf_n4 ] [info] TIME-OUT event timer id 117

[2025-08-28 11:04:49.246] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2319886528 starting

[2025-08-28 11:04:49.247] [smf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:04:51.228] [smf_app] [info] TIME-OUT event timer id 118

[2025-08-28 11:04:54.246] [smf_n4 ] [info] TIME-OUT event timer id 121

[2025-08-28 11:04:59.247] [smf_n4 ] [info] TIME-OUT event timer id 122

[2025-08-28 11:04:59.247] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2319886528 starting

[2025-08-28 11:04:59.248] [smf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:05:01.229] [smf_app] [info] TIME-OUT event timer id 123

[2025-08-28 11:05:04.247] [smf_n4 ] [info] TIME-OUT event timer id 126

[2025-08-28 11:05:09.248] [smf_n4 ] [info] TIME-OUT event timer id 127

[2025-08-28 11:05:09.248] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2319886528 starting

[2025-08-28 11:05:09.249] [smf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:05:11.230] [smf_app] [info] TIME-OUT event timer id 128

[2025-08-28 11:05:14.248] [smf_n4 ] [info] TIME-OUT event timer id 131

[2025-08-28 11:05:19.249] [smf_n4 ] [info] TIME-OUT event timer id 132

[2025-08-28 11:05:19.249] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2319886528 starting

[2025-08-28 11:05:19.250] [smf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:05:21.230] [smf_app] [info] TIME-OUT event timer id 133

[2025-08-28 11:05:24.249] [smf_n4 ] [info] TIME-OUT event timer id 136

[2025-08-28 11:05:29.250] [smf_n4 ] [info] TIME-OUT event timer id 137

[2025-08-28 11:05:29.250] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2319886528 starting

[2025-08-28 11:05:29.251] [smf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:05:31.230] [smf_app] [info] TIME-OUT event timer id 138

[2025-08-28 11:05:34.250] [smf_n4 ] [info] TIME-OUT event timer id 141

[2025-08-28 11:05:39.251] [smf_n4 ] [info] TIME-OUT event timer id 142

[2025-08-28 11:05:39.251] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2319886528 starting

[2025-08-28 11:05:39.252] [smf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:05:41.231] [smf_app] [info] TIME-OUT event timer id 143

[2025-08-28 11:05:44.252] [smf_n4 ] [info] TIME-OUT event timer id 146

[2025-08-28 11:05:49.253] [smf_n4 ] [info] TIME-OUT event timer id 147

[2025-08-28 11:05:49.253] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2319886528 starting

[2025-08-28 11:05:49.253] [smf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:05:51.231] [smf_app] [info] TIME-OUT event timer id 148

[2025-08-28 11:05:54.253] [smf_n4 ] [info] TIME-OUT event timer id 151

telcomaan@oai-5gc-vm:~/oai-cn5g-fed/docker-compose$ docker logs oai-upf

[2025-08-28 11:00:57.505] [upf_app] [start] Options parsed

[2025-08-28 11:00:57.512] [upf_app] [debug] Parsing the configuration file, file type YAML.

[2025-08-28 11:00:57.512] [config ] [info] Reading NF configuration from /openair-upf/etc/config.yaml

[2025-08-28 11:00:57.719] [config ] [debug] Unknown NF amf in configuration. Ignored

[2025-08-28 11:00:57.719] [config ] [debug] Unknown NF udm in configuration. Ignored

[2025-08-28 11:00:57.719] [config ] [debug] Unknown NF udr in configuration. Ignored

[2025-08-28 11:00:57.719] [config ] [debug] Unknown NF ausf in configuration. Ignored

[2025-08-28 11:00:57.725] [config ] [debug] Validating configuration of log_level

[2025-08-28 11:00:57.752] [config ] [info] ==== OPENAIRINTERFACE upf vBranch: HEAD Abrev. Hash: 89ee4c9 Date: Fri Aug 30 11:25:24 2024 +0000 ====

[2025-08-28 11:00:57.752] [config ] [info] Basic Configuration:

[2025-08-28 11:00:57.752] [config ] [info] - log_level..................................: info

[2025-08-28 11:00:57.752] [config ] [info] - register_nf................................: Yes

[2025-08-28 11:00:57.752] [config ] [info] - http_version...............................: 2

[2025-08-28 11:00:57.752] [config ] [info] - HTTP Request Timeout.......................: 3000 (ms)

[2025-08-28 11:00:57.752] [config ] [info] UPF Configuration:

[2025-08-28 11:00:57.752] [config ] [info] - host.......................................: oai-upf

[2025-08-28 11:00:57.752] [config ] [info] - SBI

[2025-08-28 11:00:57.752] [config ] [info] + URL......................................: http://oai-upf:8080

[2025-08-28 11:00:57.752] [config ] [info] + API Version..............................: v1

[2025-08-28 11:00:57.752] [config ] [info] + IPv4 Address ............................: 192.168.70.138

[2025-08-28 11:00:57.752] [config ] [info] - N3:

[2025-08-28 11:00:57.752] [config ] [info] + Port.....................................: 2152

[2025-08-28 11:00:57.752] [config ] [info] + IPv4 Address ............................: 192.168.70.138

[2025-08-28 11:00:57.752] [config ] [info] + MTU......................................: 1500

[2025-08-28 11:00:57.752] [config ] [info] + Interface name: .........................: eth0

[2025-08-28 11:00:57.752] [config ] [info] + Network Instance.........................: access.oai.org

[2025-08-28 11:00:57.752] [config ] [info] - N4:

[2025-08-28 11:00:57.752] [config ] [info] + Port.....................................: 8805

[2025-08-28 11:00:57.752] [config ] [info] + IPv4 Address ............................: 192.168.70.138

[2025-08-28 11:00:57.752] [config ] [info] + MTU......................................: 1500

[2025-08-28 11:00:57.752] [config ] [info] + Interface name: .........................: eth0

[2025-08-28 11:00:57.752] [config ] [info] - N6:

[2025-08-28 11:00:57.752] [config ] [info] + Port.....................................: 2152

[2025-08-28 11:00:57.752] [config ] [info] + IPv4 Address ............................: 192.168.70.138

[2025-08-28 11:00:57.752] [config ] [info] + MTU......................................: 1500

[2025-08-28 11:00:57.752] [config ] [info] + Interface name: .........................: eth0

[2025-08-28 11:00:57.752] [config ] [info] + Network Instance.........................: core.oai.org

[2025-08-28 11:00:57.752] [config ] [info] - Instance ID................................: 0

[2025-08-28 11:00:57.752] [config ] [info] - Remote N6 Gateway..........................: localhost

[2025-08-28 11:00:57.752] [config ] [info] - Support Features:

[2025-08-28 11:00:57.752] [config ] [info] + Enable BPF Datapath......................: No

[2025-08-28 11:00:57.752] [config ] [info] + Enable QoS...............................: No

[2025-08-28 11:00:57.752] [config ] [info] + Enable SNAT..............................: Yes

[2025-08-28 11:00:57.752] [config ] [info] + upf_info:

[2025-08-28 11:00:57.752] [config ] [info] - snssai_upf_info_item:

[2025-08-28 11:00:57.752] [config ] [info] + snssai:

[2025-08-28 11:00:57.752] [config ] [info] - sst..................................: 1

[2025-08-28 11:00:57.752] [config ] [info] - sd...................................: FFFFFF

[2025-08-28 11:00:57.752] [config ] [info] + dnns:

[2025-08-28 11:00:57.752] [config ] [info] - dnn..................................: internet

[2025-08-28 11:00:57.752] [config ] [info] Peer NF Configuration:

[2025-08-28 11:00:57.752] [config ] [info] NRF:

[2025-08-28 11:00:57.752] [config ] [info] - host.....................................: oai-nrf

[2025-08-28 11:00:57.752] [config ] [info] - SBI

[2025-08-28 11:00:57.752] [config ] [info] + URL....................................: http://oai-nrf:8080

[2025-08-28 11:00:57.752] [config ] [info] + API Version............................: v1

[2025-08-28 11:00:57.752] [config ] [info] SMF:

[2025-08-28 11:00:57.752] [config ] [info] - host.....................................: oai-smf

[2025-08-28 11:00:57.752] [config ] [info] - SBI

[2025-08-28 11:00:57.752] [config ] [info] + URL....................................: http://oai-smf:8080

[2025-08-28 11:00:57.752] [config ] [info] + API Version............................: v1

[2025-08-28 11:00:57.752] [config ] [info] DNNs:

[2025-08-28 11:00:57.752] [config ] [info] - DNN:

[2025-08-28 11:00:57.752] [config ] [info] + DNN......................................: internet

[2025-08-28 11:00:57.752] [config ] [info] + PDU session type.........................: IPV4

[2025-08-28 11:00:57.752] [config ] [info] + IPv4 subnet..............................: 12.1.2.0/24

[2025-08-28 11:00:57.752] [config ] [info] + DNS Settings:

[2025-08-28 11:00:57.752] [config ] [info] - primary_dns_ipv4.......................: 8.8.8.8

[2025-08-28 11:00:57.752] [config ] [info] - secondary_dns_ipv4.....................: 1.1.1.1

[2025-08-28 11:00:57.752] [upf_app] [info] HTTP Client successfully initiated on interface eth0 with timeout 3000 ms, HTTP version 2

[2025-08-28 11:00:57.752] [itti ] [start] Starting...

[2025-08-28 11:00:57.754] [itti ] [start] Started

[2025-08-28 11:00:57.757] [asc_cmd] [start] Starting...

[2025-08-28 11:00:57.756] [itti ] [info] Starting timer_manager_task

[2025-08-28 11:00:57.759] [asc_cmd] [start] Started

[2025-08-28 11:00:57.760] [upf_app] [start] Starting...

[2025-08-28 11:00:57.774] [pfcp ] [info] pfcp_l4_stack created listening to 192.168.70.138:8805

[2025-08-28 11:00:57.787] [upf_n4 ] [start] Starting...

[2025-08-28 11:00:57.795] [upf_n4 ] [start] Started

[2025-08-28 11:00:57.807] [gtpv1_u] [info] gtpu_l4_stack created listening to 192.168.70.138:2152

[2025-08-28 11:00:57.809] [upf_n3 ] [start] Starting...

[2025-08-28 11:00:57.816] [upf_n3 ] [start] Started

[2025-08-28 11:00:58.065] [upf_app] [start] Starting...

[2025-08-28 11:00:58.067] [upf_app] [info] Send NF Instance Registration to NRF

[2025-08-28 11:00:58.079] [upf_app] [info] Response from NRF, JSON data:

{"capacity":100,"heartBeatTimer":10,"ipv4Addresses":["192.168.70.138"],"json_data":null,"nfInstanceId":"5468f5a5-8632-44d4-aa4e-19d46a1e87a8","nfInstanceName":"OAI-UPF","nfServices":[],"nfStatus":"REGISTERED","nfType":"UPF","priority":1,"sNssais":[{"sd":"FFFFFF","sst":1}],"upfInfo":{"sNssaiUpfInfoList":[{"dnnUpfInfoList":[{"dnn":"internet"}],"sNssai":{"sd":"FFFFFF","sst":1}}]}}

[2025-08-28 11:00:58.080] [upf_app] [start] Started

[2025-08-28 11:00:58.080] [upf_app] [start] Started

[2025-08-28 11:00:59.216] [upf_n4 ] [info] handle_receive(34 bytes)

[2025-08-28 11:00:59.216] [upf_n4 ] [info] Handle SX ASSOCIATION SETUP REQUEST

[2025-08-28 11:01:08.079] [upf_app] [info] TIME-OUT event timer id 1

[2025-08-28 11:01:08.079] [upf_app] [info] Send NF Update to NRF

[2025-08-28 11:01:08.136] [upf_app] [info] Got successful response from NRF

[2025-08-28 11:01:09.221] [upf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:01:09.221] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST

[2025-08-28 11:01:18.136] [upf_app] [info] TIME-OUT event timer id 3

[2025-08-28 11:01:18.136] [upf_app] [info] Send NF Update to NRF

[2025-08-28 11:01:18.141] [upf_app] [info] Got successful response from NRF

[2025-08-28 11:01:19.223] [upf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:01:19.223] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST

[2025-08-28 11:01:28.142] [upf_app] [info] TIME-OUT event timer id 5

[2025-08-28 11:01:28.142] [upf_app] [info] Send NF Update to NRF

[2025-08-28 11:01:28.144] [upf_app] [info] Got successful response from NRF

[2025-08-28 11:01:29.224] [upf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:01:29.224] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST

[2025-08-28 11:01:38.145] [upf_app] [info] TIME-OUT event timer id 7

[2025-08-28 11:01:38.145] [upf_app] [info] Send NF Update to NRF

[2025-08-28 11:01:38.148] [upf_app] [info] Got successful response from NRF

[2025-08-28 11:01:39.225] [upf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:01:39.225] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST

[2025-08-28 11:01:48.148] [upf_app] [info] TIME-OUT event timer id 9

[2025-08-28 11:01:48.148] [upf_app] [info] Send NF Update to NRF

[2025-08-28 11:01:48.151] [upf_app] [info] Got successful response from NRF

[2025-08-28 11:01:49.226] [upf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:01:49.226] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST

[2025-08-28 11:01:58.154] [upf_app] [info] TIME-OUT event timer id 11

[2025-08-28 11:01:58.155] [upf_app] [info] Send NF Update to NRF

[2025-08-28 11:01:58.171] [upf_app] [info] Got successful response from NRF

[2025-08-28 11:01:59.227] [upf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:01:59.227] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST

[2025-08-28 11:02:08.172] [upf_app] [info] TIME-OUT event timer id 13

[2025-08-28 11:02:08.172] [upf_app] [info] Send NF Update to NRF

[2025-08-28 11:02:08.174] [upf_app] [info] Got successful response from NRF

[2025-08-28 11:02:09.228] [upf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:02:09.228] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST

[2025-08-28 11:02:18.175] [upf_app] [info] TIME-OUT event timer id 15

[2025-08-28 11:02:18.175] [upf_app] [info] Send NF Update to NRF

[2025-08-28 11:02:18.179] [upf_app] [info] Got successful response from NRF

[2025-08-28 11:02:19.229] [upf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:02:19.229] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST

[2025-08-28 11:02:28.180] [upf_app] [info] TIME-OUT event timer id 17

[2025-08-28 11:02:28.180] [upf_app] [info] Send NF Update to NRF

[2025-08-28 11:02:28.185] [upf_app] [info] Got successful response from NRF

[2025-08-28 11:02:29.231] [upf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:02:29.231] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST

[2025-08-28 11:02:38.186] [upf_app] [info] TIME-OUT event timer id 19

[2025-08-28 11:02:38.186] [upf_app] [info] Send NF Update to NRF

[2025-08-28 11:02:38.189] [upf_app] [info] Got successful response from NRF

[2025-08-28 11:02:39.234] [upf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:02:39.234] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST

[2025-08-28 11:02:48.189] [upf_app] [info] TIME-OUT event timer id 21

[2025-08-28 11:02:48.189] [upf_app] [info] Send NF Update to NRF

[2025-08-28 11:02:48.192] [upf_app] [info] Got successful response from NRF

[2025-08-28 11:02:49.235] [upf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:02:49.235] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST

[2025-08-28 11:02:58.192] [upf_app] [info] TIME-OUT event timer id 23

[2025-08-28 11:02:58.192] [upf_app] [info] Send NF Update to NRF

[2025-08-28 11:02:58.195] [upf_app] [info] Got successful response from NRF

[2025-08-28 11:02:59.236] [upf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:02:59.236] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST

[2025-08-28 11:03:08.195] [upf_app] [info] TIME-OUT event timer id 25

[2025-08-28 11:03:08.195] [upf_app] [info] Send NF Update to NRF

[2025-08-28 11:03:08.198] [upf_app] [info] Got successful response from NRF

[2025-08-28 11:03:09.237] [upf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:03:09.237] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST

[2025-08-28 11:03:18.198] [upf_app] [info] TIME-OUT event timer id 27

[2025-08-28 11:03:18.198] [upf_app] [info] Send NF Update to NRF

[2025-08-28 11:03:18.201] [upf_app] [info] Got successful response from NRF

[2025-08-28 11:03:19.237] [upf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:03:19.238] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST

[2025-08-28 11:03:28.201] [upf_app] [info] TIME-OUT event timer id 29

[2025-08-28 11:03:28.201] [upf_app] [info] Send NF Update to NRF

[2025-08-28 11:03:28.204] [upf_app] [info] Got successful response from NRF

[2025-08-28 11:03:29.238] [upf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:03:29.238] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST

[2025-08-28 11:03:38.204] [upf_app] [info] TIME-OUT event timer id 31

[2025-08-28 11:03:38.204] [upf_app] [info] Send NF Update to NRF

[2025-08-28 11:03:38.207] [upf_app] [info] Got successful response from NRF

[2025-08-28 11:03:39.239] [upf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:03:39.239] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST

[2025-08-28 11:03:48.207] [upf_app] [info] TIME-OUT event timer id 33

[2025-08-28 11:03:48.207] [upf_app] [info] Send NF Update to NRF

[2025-08-28 11:03:48.210] [upf_app] [info] Got successful response from NRF

[2025-08-28 11:03:49.241] [upf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:03:49.241] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST

[2025-08-28 11:03:58.211] [upf_app] [info] TIME-OUT event timer id 35

[2025-08-28 11:03:58.211] [upf_app] [info] Send NF Update to NRF

[2025-08-28 11:03:58.214] [upf_app] [info] Got successful response from NRF

[2025-08-28 11:03:59.242] [upf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:03:59.242] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST

[2025-08-28 11:04:08.214] [upf_app] [info] TIME-OUT event timer id 37

[2025-08-28 11:04:08.214] [upf_app] [info] Send NF Update to NRF

[2025-08-28 11:04:08.217] [upf_app] [info] Got successful response from NRF

[2025-08-28 11:04:09.243] [upf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:04:09.243] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST

[2025-08-28 11:04:16.852] [upf_n4 ] [info] handle_receive(174 bytes)

[2025-08-28 11:04:16.852] [upf_app] [info] Received N4_SESSION_ESTABLISHMENT_REQUEST seid 0x1

[2025-08-28 11:04:16.852] [upf_n4 ] [info] pfcp_session::add(far) seid 0x1

[2025-08-28 11:04:16.852] [upf_n4 ] [info] pfcp_session::add(pdr) seid 0x1

[2025-08-28 11:04:16.900] [upf_n4 ] [info] handle_receive(100 bytes)

[2025-08-28 11:04:16.900] [upf_app] [info] Received N4_SESSION_MODIFICATION_REQUEST seid 0x1

[2025-08-28 11:04:16.900] [upf_n4 ] [info] pfcp_session::add(far) seid 0x1

[2025-08-28 11:04:16.901] [upf_n4 ] [info] pfcp_session::add(pdr) seid 0x1

[2025-08-28 11:04:18.217] [upf_app] [info] TIME-OUT event timer id 39

[2025-08-28 11:04:18.217] [upf_app] [info] Send NF Update to NRF

[2025-08-28 11:04:18.220] [upf_app] [info] Got successful response from NRF

[2025-08-28 11:04:19.244] [upf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:04:19.244] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST

[2025-08-28 11:04:28.220] [upf_app] [info] TIME-OUT event timer id 43

[2025-08-28 11:04:28.220] [upf_app] [info] Send NF Update to NRF

[2025-08-28 11:04:28.223] [upf_app] [info] Got successful response from NRF

[2025-08-28 11:04:29.245] [upf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:04:29.245] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST

[2025-08-28 11:04:38.223] [upf_app] [info] TIME-OUT event timer id 45

[2025-08-28 11:04:38.223] [upf_app] [info] Send NF Update to NRF

[2025-08-28 11:04:38.226] [upf_app] [info] Got successful response from NRF

[2025-08-28 11:04:39.246] [upf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:04:39.246] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST

[2025-08-28 11:04:48.226] [upf_app] [info] TIME-OUT event timer id 47

[2025-08-28 11:04:48.226] [upf_app] [info] Send NF Update to NRF

[2025-08-28 11:04:48.229] [upf_app] [info] Got successful response from NRF

[2025-08-28 11:04:49.246] [upf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:04:49.247] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST

[2025-08-28 11:04:58.229] [upf_app] [info] TIME-OUT event timer id 49

[2025-08-28 11:04:58.229] [upf_app] [info] Send NF Update to NRF

[2025-08-28 11:04:58.232] [upf_app] [info] Got successful response from NRF

[2025-08-28 11:04:59.247] [upf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:04:59.247] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST

[2025-08-28 11:05:08.233] [upf_app] [info] TIME-OUT event timer id 51

[2025-08-28 11:05:08.233] [upf_app] [info] Send NF Update to NRF

[2025-08-28 11:05:08.236] [upf_app] [info] Got successful response from NRF

[2025-08-28 11:05:09.248] [upf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:05:09.248] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST

[2025-08-28 11:05:18.236] [upf_app] [info] TIME-OUT event timer id 53

[2025-08-28 11:05:18.236] [upf_app] [info] Send NF Update to NRF

[2025-08-28 11:05:18.239] [upf_app] [info] Got successful response from NRF

[2025-08-28 11:05:19.249] [upf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:05:19.249] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST

[2025-08-28 11:05:28.240] [upf_app] [info] TIME-OUT event timer id 55

[2025-08-28 11:05:28.240] [upf_app] [info] Send NF Update to NRF

[2025-08-28 11:05:28.243] [upf_app] [info] Got successful response from NRF

[2025-08-28 11:05:29.250] [upf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:05:29.250] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST

[2025-08-28 11:05:38.243] [upf_app] [info] TIME-OUT event timer id 57

[2025-08-28 11:05:38.243] [upf_app] [info] Send NF Update to NRF

[2025-08-28 11:05:38.246] [upf_app] [info] Got successful response from NRF

[2025-08-28 11:05:39.252] [upf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:05:39.252] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST

[2025-08-28 11:05:48.246] [upf_app] [info] TIME-OUT event timer id 59

[2025-08-28 11:05:48.246] [upf_app] [info] Send NF Update to NRF

[2025-08-28 11:05:48.249] [upf_app] [info] Got successful response from NRF

[2025-08-28 11:05:49.253] [upf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:05:49.253] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST

[2025-08-28 11:05:58.249] [upf_app] [info] TIME-OUT event timer id 61

[2025-08-28 11:05:58.249] [upf_app] [info] Send NF Update to NRF

[2025-08-28 11:05:58.252] [upf_app] [info] Got successful response from NRF

[2025-08-28 11:05:59.254] [upf_n4 ] [info] handle_receive(16 bytes)

[2025-08-28 11:05:59.254] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST

telcomaan@oai-5gc-vm:~/oai-cn5g-fed/docker-compose$

look very carefully and see whats happening and what failed