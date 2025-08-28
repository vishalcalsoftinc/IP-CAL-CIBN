# Full, explicit implementation guide — verified, corrected, and expanded (every command & file change)

Nice — your checklist is mostly correct. I’ve expanded it into a **complete, runnable guide** with every command you’ll need, fixes for a few pitfalls, and explicit file paths/edits. I assume your working directory on the VM will be `~/oai-cn5g-fed` after cloning. If any path differs in your repo version, I give alternatives and how to detect them. Key references used: OAI `oai-cn5g-fed` docs (basic deployment & config) and several OAI deployment guides. ([GitLab](https://gitlab.eurecom.fr/oai/cn5g/oai-cn5g-fed?utm_source=chatgpt.com "oai-cn5g-fed"), [NVIDIA Docs](https://docs.nvidia.com/aerial/aerial-research-cloud/archive/23.1.a0.7/text/installation_guide/index.html?utm_source=chatgpt.com "Installation Guide - NVIDIA Docs"))

---

## Quick summary of what this guide does

1. Prepare `gcp-oai-5gc-vm` with Docker, Python3 and required tools.
    
2. Clone `oai-cn5g-fed`.
    
3. Edit the environment/config files (PLMN, TAC, FQDN, UE subscriber).
    
4. Import the subscriber row into the CN database.
    
5. Start the OAI CN (basic scenario) with `core-network.py`.
    
6. Enable NAT and forwarding so the UE gets Internet access.
    
7. Verify registration/attach and provide rollback/troubleshooting steps.
    

---

## Preflight & assumptions

- VM hostname: `gcp-oai-5gc-vm` (you used `oai-5gc-vm` — I will use `gcp-oai-5gc-vm` in examples).
    
- Network interface that goes to the Internet on that VM is `ens4` (you used `ens4` — if different, replace `ens4` below).
    
- You want `basic` OAI core scenario (NRF + AMF + SMF + UPF etc.) started by `core-network.py`. The `oai-cn5g-fed` repo provides helper scripts and config templates — we use them. ([GitLab](https://gitlab.eurecom.fr/oai/cn5g/oai-cn5g-fed/-/blob/master/docs/DEPLOY_SA5G_BASIC_DEPLOYMENT.md?utm_source=chatgpt.com "docs/DEPLOY_SA5G_BASIC_DEPLOYMENT.md · master"), [GitLab](https://gitlab.nrp-nautilus.io/oai/oai-cn5g-fed/-/blob/439a3e75be0ae76d045206ff9012bf9aaab52a99/docs/DEPLOY_SA5G_BASIC_DEPLOYMENT.md?utm_source=chatgpt.com "oai-cn5g-fed - docs - nautilus gitlab"))
    

---

# Step-by-step commands and file edits

> Run these as the main user you SSH with (we’ll add it to `docker` group). Use `sudo` where shown.

### 0) edit AMF IP in oai-cu.conf in oai-cu-vm
```

    ////////// AMF parameters:
    #amf_ip_address = ({ ipv4 = "172.17.0.95"; });   # connects to open5gs
    amf_ip_address = ({ ipv4 = "172.17.0.97"; });   # connects to oai-5gc

```

### 1) SSH into the VM

```bash
ssh <your_user>@<gcp-oai-5gc-vm-external-ip>
# or if using VS Code remote: open the VM workspace as usual
```

### 2) Install prerequisites (exact commands)

```bash
# update
sudo apt update && sudo apt upgrade -y

# Install core packages
sudo apt install -y git curl jq python3 python3-pip

# Install Docker 
sudo apt install -y docker.io 

# enable & start docker
sudo systemctl enable --now docker

# Optional: install docker-compose (standalone) if your scripts need `docker-compose` command
# NOTE: oai scripts usually work with 'docker compose' (plugin). If you prefer the legacy binary:
sudo apt install -y docker-compose

# Add your user to docker group so you can run docker without sudo
sudo usermod -aG docker $USER
sudo usermod -aG docker telcomaan
# Apply group immediately in this session
newgrp docker <<'EONG'
echo "docker group applied"
EONG

# Confirm versions (should show docker and python3)
docker version
python3 --version
```

Notes: `docker compose` (the plugin) is the supported method in many OAI docs; if you use the plugin, commands are `docker compose` (space). The OAI helper script uses Python to execute `docker compose` internally. ([NVIDIA Docs](https://docs.nvidia.com/aerial/aerial-research-cloud/archive/23.1.a0.7/text/installation_guide/index.html?utm_source=chatgpt.com "Installation Guide - NVIDIA Docs"))

---

### 3) Clone the OAI CN repo and sync components

```bash
# go to a workspace folder
cd ~
git clone https://gitlab.eurecom.fr/oai/cn5g/oai-cn5g-fed.git
cd oai-cn5g-fed

# (Optional recommended) checkout a stable tag or branch if you want reproducibility
# Example (only if you know a tag you want):
# git checkout tags/v1.3.0 -b pinned-v1.3.0

# Some deployments call a sync script to download subcomponents (if present)
# If repository has scripts/syncComponents.sh run it:
if [ -x ./scripts/syncComponents.sh ]; then
  ./scripts/syncComponents.sh
fi
```

If the repo cloning URL requires credentials in your environment, use your GitLab auth or a mirrored GitHub repo. See repo README for variants. ([GitLab](https://gitlab.eurecom.fr/oai/cn5g/oai-cn5g-fed?utm_source=chatgpt.com "oai-cn5g-fed"))

---

### 4) Locate the docker-compose folder and confirm available scenarios

```bash
cd docker-compose
ls -la
# look for files like:
# docker-compose-basic-nrf.yaml  core-network.py  database/  .env  core-network.env  ...
# confirm helper script exists
ls -l core-network.py
```

If `core-network.py` exists, it’s the script that brings up the core in the selected scenario. Many OAI docs show `python3 core-network.py --type start-basic --scenario 1` as the way to start the basic core. ([GitLab](https://gitlab.eurecom.fr/oai/cn5g/oai-cn5g-fed/-/blob/master/docs/DEPLOY_SA5G_BASIC_DEPLOYMENT.md?utm_source=chatgpt.com "docs/DEPLOY_SA5G_BASIC_DEPLOYMENT.md · master"), [WITest](https://witestlab.poly.edu/blog/exploring-the-5g-core-network/?utm_source=chatgpt.com "Exploring the 5G core network"))


### 5) Add correct PLMN , DNN , SLICE

#### basic_nrf_config.yaml

```basic-nrf-config.yaml
telcomaan@oai-5gc-vm:~/oai-cn5g-fed/docker-compose$ cat conf/basic_nrf_config.yaml
################################################################################
# Licensed to the OpenAirInterface (OAI) Software Alliance under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The OpenAirInterface Software Alliance licenses this file to You under
# the OAI Public License, Version 1.1  (the "License"); you may not use this file
# except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.openairinterface.org/?page_id=698
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#-------------------------------------------------------------------------------
# For more information about the OpenAirInterface (OAI) Software Alliance:
#      contact@openairinterface.org
################################################################################

# OAI CN Configuration File
### This file can be used by all OAI NFs
### Some fields are specific to an NF and will be ignored by other NFs

## NOTE ON YAML ANCHORS ##
# We use YAML anchors to ease the configuration and to avoid duplicating parts of the configuration.
# This is especially true for the SNSSAIs, as we have to define them for multiple NFs.
# Please note that the use of anchors is not mandatory, and you can define the SNSSAI in each NF yourself.
# You can read more about anchors here: https://yaml.org/spec/1.2.2/#anchors-and-aliases

############# Common configuration

# Log level for all the NFs
log_level:
  general: info # Possible values: trace, debug, info, warn, error, critical, off

# If you enable registration, the other NFs will use the NRF discovery mechanism
register_nf:
  general: yes

http_version: 2

############## SBI Interfaces
### Each NF takes its local SBI interfaces and remote interfaces from here, unless it gets them using NRF discovery mechanisms
nfs:
  amf:
    host: oai-amf
    sbi:
      port: 8080
      api_version: v1
      interface_name: eth0
    n2:
      interface_name: eth0
      port: 38412
  smf:
    host: oai-smf
    sbi:
      port: 8080
      api_version: v1
      interface_name: eth0
    n4:
      interface_name: eth0
      port: 8805
  upf:
    host: oai-upf
    sbi:
      port: 8080
      api_version: v1
      interface_name: eth0
    n3:
      interface_name: eth0
      port: 2152
    n4:
      interface_name: eth0
      port: 8805
    n6:
      interface_name: eth0
    n9:
      interface_name: eth0
      port: 2152
  udm:
    host: oai-udm
    sbi:
      port: 8080
      api_version: v1
      interface_name: eth0
  udr:
    host: oai-udr
    sbi:
      port: 8080
      api_version: v1
      interface_name: eth0
  ausf:
    host: oai-ausf
    sbi:
      port: 8080
      api_version: v1
      interface_name: eth0
  nrf:
    host: oai-nrf
    sbi:
      port: 8080
      api_version: v1
      interface_name: eth0

#### Common for UDR and AMF
database:
  host: mysql
  user: test
  type: mysql
  password: test
  database_name: oai_db
  generate_random: true
  connection_timeout: 300 # seconds

## general single_nssai configuration
## Defines YAML anchors, which are reused in the config file
snssais:
  - &embb_slice1
    sst: 1
    sd: FFFFFF # in hex

############## NF-specific configuration
amf:
  amf_name: "OAI-AMF"
  # This really depends on if we want to keep the "mini" version or not
  support_features_options:
    enable_simple_scenario: no # "no" by default with the normal deployment scenarios with AMF/SMF/UPF/AUSF/UDM/UDR/NRF.
                               # set it to "yes" to use with the minimalist deployment scenario (including only AMF/SMF/UPF) by using the internal AUSF/UDM implemented inside AMF.
                               # There's no NRF in this scenario, SMF info is taken from "nfs" section.
    enable_nssf: no
    enable_smf_selection: yes
  relative_capacity: 30
  statistics_timer_interval: 20  # in seconds
  emergency_support: false
  served_guami_list:
    - mcc: 999
      mnc: 70
      amf_region_id: 01
      amf_set_id: 001
      amf_pointer: 01
  plmn_support_list:
    - mcc: 999
      mnc: 70
      tac: 0x0001
      nssai:
        - *embb_slice1
  supported_integrity_algorithms:
    - "NIA0"
    - "NIA1"
    - "NIA2"
  supported_encryption_algorithms:
    - "NEA0"
    - "NEA1"
    - "NEA2"

smf:
  ue_mtu: 1500
  support_features:
    use_local_subscription_info: yes # Use infos from local_subscription_info or from UDM
    use_local_pcc_rules: yes # Use infos from local_pcc_rules or from PCF
  # we resolve from NRF, this is just to configure usage_reporting
  upfs:
    - host: oai-upf
      config:
        enable_usage_reporting: no
  ue_dns:
    primary_ipv4: "172.21.3.100"
    primary_ipv6: "2001:4860:4860::8888"
    secondary_ipv4: "8.8.8.8"
    secondary_ipv6: "2001:4860:4860::8888"
  ims:
    pcscf_ipv4: "127.0.0.1"
    pcscf_ipv6: "fe80::7915:f408:1787:db8b"
  # the DNN you configure here should be configured in "dnns"
  # follows the SmfInfo datatype from 3GPP TS 29.510
  smf_info:
    sNssaiSmfInfoList:
      - sNssai: *embb_slice1
        dnnSmfInfoList:
          - dnn: "internet"
  local_subscription_infos:
    - single_nssai: *embb_slice1
      dnn: "internet"
      qos_profile:
        5qi: 9
        session_ambr_ul: "200Mbps"
        session_ambr_dl: "400Mbps"

upf:
  support_features:
    enable_bpf_datapath: no    # If "on": BPF is used as datapath else simpleswitch is used, DEFAULT= off
    enable_snat: yes           # If "on": Source natting is done for UE, DEFAULT= off
  remote_n6_gw: localhost      # Dummy host since simple-switch does not use N6 GW
  smfs:
    - host: oai-smf            # To be used for PFCP association in case of no-NRF
  upf_info:
    sNssaiUpfInfoList:
      - sNssai: *embb_slice1
        dnnUpfInfoList:
          - dnn: "internet"

## DNN configuration
dnns:
  - dnn: "internet"
    pdu_session_type: "IPV4"
    ipv4_subnet: "12.1.2.0/24"

```


### 6. Expose port 38412 from amf , 2152 from upf for oai-ran

#### docker-compose-basic-nrf.yaml
```docker-compose-basic-nrf.yaml
telcomaan@oai-5gc-vm:~/oai-cn5g-fed/docker-compose$ cat docker-compose-basic-nrf.yaml
version: '3.8'
services:
    mysql:
        container_name: "mysql"
        image: mysql:8.0
        volumes:
            - ./database/oai_db2.sql:/docker-entrypoint-initdb.d/oai_db.sql
            - ./healthscripts/mysql-healthcheck2.sh:/tmp/mysql-healthcheck.sh
        environment:
            - TZ=Europe/Paris
            - MYSQL_DATABASE=oai_db
            - MYSQL_USER=test
            - MYSQL_PASSWORD=test
            - MYSQL_ROOT_PASSWORD=linux
        healthcheck:
            test: /bin/bash -c "/tmp/mysql-healthcheck.sh"
            interval: 10s
            timeout: 5s
            retries: 30
        networks:
            public_net:
# For CI purposes, we are keeping the line commented
#                ipv4_address: 192.168.70.131
    oai-udr:
        container_name: "oai-udr"
        image: oaisoftwarealliance/oai-udr:v2.1.0
        expose:
            - 80/tcp
            - 8080/tcp
        volumes:
            - ./conf/basic_nrf_config.yaml:/openair-udr/etc/config.yaml
        environment:
            - TZ=Europe/Paris
        depends_on:
            - mysql
            - oai-nrf
        networks:
            public_net:
# For CI purposes, we are keeping the line commented
#                ipv4_address: 192.168.70.136
    oai-udm:
        container_name: "oai-udm"
        image: oaisoftwarealliance/oai-udm:v2.1.0
        expose:
            - 80/tcp
            - 8080/tcp
        volumes:
            - ./conf/basic_nrf_config.yaml:/openair-udm/etc/config.yaml
        environment:
            - TZ=Europe/Paris
        depends_on:
            - oai-nrf
        networks:
            public_net:
# For CI purposes, we are keeping the line commented
#                ipv4_address: 192.168.70.137
    oai-ausf:
        container_name: "oai-ausf"
        image: oaisoftwarealliance/oai-ausf:v2.1.0
        expose:
            - 80/tcp
            - 8080/tcp
        volumes:
            - ./conf/basic_nrf_config.yaml:/openair-ausf/etc/config.yaml
        environment:
            - TZ=Europe/Paris
        depends_on:
            - oai-nrf
        networks:
            public_net:
# For CI purposes, we are keeping the line commented
#                ipv4_address: 192.168.70.138
    oai-nrf:
        container_name: "oai-nrf"
        image: oaisoftwarealliance/oai-nrf:v2.1.0
        expose:
            - 80/tcp
            - 8080/tcp
        volumes:
            - ./conf/basic_nrf_config.yaml:/openair-nrf/etc/config.yaml
        environment:
            - TZ=Europe/Paris
        networks:
            public_net:
# For CI purposes, we are keeping the line commented
#                ipv4_address: 192.168.70.130
    oai-amf:
        container_name: "oai-amf"
        image: oaisoftwarealliance/oai-amf:v2.1.0
        expose:
            - 80/tcp
            - 8080/tcp
            - 38412/sctp
        ports:
            - "38412:38412/sctp" # exposing N2 interface for gNB
        volumes:
            - ./conf/basic_nrf_config.yaml:/openair-amf/etc/config.yaml
        environment:
            - TZ=Europe/Paris
        depends_on:
            - oai-nrf
        networks:
            public_net:
# For CI purposes, we are keeping the line commented
#                ipv4_address: 192.168.70.132
    oai-smf:
        container_name: "oai-smf"
        image: oaisoftwarealliance/oai-smf:v2.1.0
        expose:
            - 80/tcp
            - 8080/tcp
            - 8805/udp
        volumes:
            - ./conf/basic_nrf_config.yaml:/openair-smf/etc/config.yaml
        environment:
            - TZ=Europe/Paris
        depends_on:
            - oai-nrf
        networks:
            public_net:
# For CI purposes, we are keeping the line commented
#                ipv4_address: 192.168.70.133
    oai-upf:
        container_name: "oai-upf"
        image: oaisoftwarealliance/oai-upf:v2.1.0
        expose:
            - 2152/udp
            - 8805/udp
        ports:
            - "2152:2152/udp" # exposing N3 interface for UE/DN
        volumes:
            - ./conf/basic_nrf_config.yaml:/openair-upf/etc/config.yaml
        environment:
            - TZ=Europe/Paris
        depends_on:
            - oai-nrf
        cap_add:
            - NET_ADMIN
            - SYS_ADMIN
        cap_drop:
            - ALL
        privileged: true
        networks:
            public_net:
# For CI purposes, we are keeping the line commented
#                ipv4_address: 192.168.70.134
    oai-ext-dn:
        privileged: true
        init: true
        container_name: oai-ext-dn
        image: oaisoftwarealliance/trf-gen-cn5g:latest
        environment:
            - UPF_FQDN=oai-upf
            - UE_NETWORK=12.1.1.0/24
            - USE_FQDN=yes
        healthcheck:
            test: /bin/bash -c "ip r | grep 12.1.1"
            interval: 10s
            timeout: 5s
            retries: 5
        networks:
            public_net:
# For CI purposes, we are keeping the line commented
#                ipv4_address: 192.168.70.135

networks:
    public_net:
        driver: bridge
        name: demo-oai-public-net
        ipam:
            config:
                - subnet: 192.168.70.128/26
        driver_opts:
            com.docker.network.bridge.name: "demo-oai"
```

---

### 7) Configure subscriber (UE) data — SQL file edit (exact commands)

#### oai_db2.sql

```bash
# find SQL file
ls docker-compose/database
# open the SQL you find (example)
nano docker-compose/database/oai_db2.sql
```

Append or edit the subscriber row so it matches your UE IMSI/key. Use the SQL you proposed — this is correct in format used by OAI MySQL-based DBs (UNHEX for keys):

```sql
-- Add UE 999700000000001 (replace servingPlmnid '99970' if needed)

INSERT INTO `AuthenticationSubscription`
(`ueid`, `authenticationMethod`, `encPermanentKey`, `protectionParameterId`, `sequenceNumber`,
 `authenticationManagementField`, `algorithmId`, `encOpcKey`, `encTopcKey`, `vectorGenerationInHss`,
 `n5gcAuthMethod`, `rgAuthenticationInd`, `supi`)
VALUES
('999700000000001',
 '5G_AKA',
 '465B5CE8B199B49FAA5F0A2EE238A6BC',
 '465B5CE8B199B49FAA5F0A2EE238A6BC',
 '{"sqn":"000000000020","sqnScheme":"NON_TIME_BASED","lastIndexes":{"ausf":0}}',
 '8000',
 'milenage',
 'E8ED289DEBA952E4283B54E88E6183CA',
 NULL,
 NULL,
 NULL,
 NULL,
 '999700000000001');

INSERT INTO `AccessAndMobilitySubscriptionData`
(`ueid`, `servingPlmnid`, `nssai`)
VALUES
('999700000000001',
 '99970',
 '{"defaultSingleNssais":[{"sst":1,"sd":"FFFFFF"}]}');

INSERT INTO `SessionManagementSubscriptionData`
(`ueid`, `servingPlmnid`, `singleNssai`, `dnnConfigurations`)
VALUES
('999700000000001',
 '99970',
 '{"sst":1,"sd":"FFFFFF"}',
 '{"internet": {"pduSessionTypes": {"defaultSessionType": "IPV4"}, "sscModes": {"defaultSscMode": "SSC_MODE_1"}, "5gQosProfile": {"5qi": 6, "arp": {"priorityLevel": 1, "preemptCap": "NOT_PREEMPT", "preemptVuln": "NOT_PREEMPTABLE"}, "priorityLevel": 1}, "sessionAmbr": {"uplink": "100Mbps", "downlink": "100Mbps"}}}');

INSERT INTO `SmfSelectionSubscriptionData`
(`ueid`, `servingPlmnid`, `subscribedSnssaiInfos`)
VALUES
('999700000000001',
 '99970',
 '{"singleNssai": {"sst":1, "sd":"FFFFFF"}, "dnn": "internet"}');
```


---

### 8. Added NAT forwading in Host vm and inside UPF container 

```
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -s 12.1.2.0/24 -o ens4 -j MASQUERADE
sudo iptables -A FORWARD -s 12.1.2.0/24 -o ens4 -j ACCEPT
sudo iptables -A FORWARD -d 12.1.2.0/24 -i ens4 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```
---

### 9) Start the OAI core (the exact start command)

From `~/oai-cn5g-fed/docker-compose` run:

```bash
cd ~/oai-cn5g-fed/docker-compose

# Recommended: run as your normal user (docker group), but use sudo if permissions require it
python3 core-network.py --type start-basic --scenario 1
python3 core-network.py --type stop-basic --scenario 1
```


**Verify containers are running:**

```bash
# List running containers
docker ps

# Useful: follow AMF logs
docker logs -f oai-amf   # or docker compose logs -f oai-amf
docker logs -f oai-smf 
docker logs -f oai-upf 
```


```
telcomaan@oai-5gc-vm:~/oai-cn5g-fed/docker-compose$ python3 core-network.py --type start-basic --scenario 1
[2025-08-28 09:43:27,109] root:DEBUG:  Starting 5gcn components... Please wait....
[2025-08-28 09:43:27,770] root:DEBUG: docker-compose -f docker-compose-basic-nrf.yaml up -d
Creating network "demo-oai-public-net" with driver "bridge"
Creating mysql      ... done
Creating oai-ext-dn ... done
Creating oai-nrf    ... done
Creating oai-amf    ... done
Creating oai-ausf   ... done
Creating oai-udr    ... done
Creating oai-smf    ... done
Creating oai-udm    ... done
Creating oai-upf    ... done

[2025-08-28 09:43:33,048] root:DEBUG:  OAI 5G Core network started, checking the health status of the containers... takes few secs....
[2025-08-28 09:43:33,048] root:DEBUG: docker-compose -f docker-compose-basic-nrf.yaml ps -a
[2025-08-28 09:44:06,603] root:DEBUG:  All components are healthy, please see below for more details....
Name                 Command                  State                                         Ports                                   
---------------------------------------------------------------------------------------------------------------------------------------
mysql        docker-entrypoint.sh mysqld      Up (healthy)   3306/tcp, 33060/tcp                                                       
oai-amf      /openair-amf/bin/oai_amf - ...   Up (healthy)   0.0.0.0:38412->38412/sctp,:::38412->38412/sctp, 80/tcp, 8080/tcp, 9090/tcp
oai-ausf     /openair-ausf/bin/oai_ausf ...   Up (healthy)   80/tcp, 8080/tcp                                                          
oai-ext-dn   /bin/bash /tmp/trfgen_entr ...   Up (healthy)                                                                             
oai-nrf      /openair-nrf/bin/oai_nrf - ...   Up (healthy)   80/tcp, 8080/tcp, 9090/tcp                                                
oai-smf      /openair-smf/bin/oai_smf - ...   Up (healthy)   80/tcp, 8080/tcp, 8805/udp                                                
oai-udm      /openair-udm/bin/oai_udm - ...   Up (healthy)   80/tcp, 8080/tcp                                                          
oai-udr      /openair-udr/bin/oai_udr - ...   Up (healthy)   80/tcp, 8080/tcp                                                          
oai-upf      /openair-upf/bin/oai_upf - ...   Up (healthy)   0.0.0.0:2152->2152/udp,:::2152->2152/udp, 8805/udp
[2025-08-28 09:44:16,604] root:DEBUG:  Checking if the containers are configured....
[2025-08-28 09:44:16,862] root:DEBUG:  Checking if AMF, SMF and UPF registered with nrf core network....
[2025-08-28 09:44:16,862] root:DEBUG: curl -s -X GET --http2-prior-knowledge http://192.168.70.132:8080/nnrf-nfm/v1/nf-instances?nf-type="AMF" | grep -o "192.168.70.135"
192.168.70.135
[2025-08-28 09:44:16,879] root:DEBUG: curl -s -X GET --http2-prior-knowledge http://192.168.70.132:8080/nnrf-nfm/v1/nf-instances?nf-type="SMF" | grep -o "192.168.70.133"
192.168.70.133
[2025-08-28 09:44:16,895] root:DEBUG: curl -s -X GET --http2-prior-knowledge http://192.168.70.132:8080/nnrf-nfm/v1/nf-instances?nf-type="UPF" | grep -o "192.168.70.136"
192.168.70.136
[2025-08-28 09:44:16,911] root:DEBUG:  Checking if AUSF, UDM and UDR registered with nrf core network....
[2025-08-28 09:44:16,911] root:DEBUG: curl -s -X GET --http2-prior-knowledge http://192.168.70.132:8080/nnrf-nfm/v1/nf-instances?nf-type="AUSF" | grep -o "192.168.70.138"
192.168.70.138
[2025-08-28 09:44:16,926] root:DEBUG: curl -s -X GET --http2-prior-knowledge http://192.168.70.132:8080/nnrf-nfm/v1/nf-instances?nf-type="UDM" | grep -o "192.168.70.137"
192.168.70.137
[2025-08-28 09:44:16,942] root:DEBUG: curl -s -X GET --http2-prior-knowledge http://192.168.70.132:8080/nnrf-nfm/v1/nf-instances?nf-type="UDR" | grep -o "192.168.70.134"
192.168.70.134
[2025-08-28 09:44:16,958] root:DEBUG:  AUSF, UDM, UDR, AMF, SMF and UPF are registered to NRF....
[2025-08-28 09:44:16,959] root:DEBUG:  Checking if SMF is able to connect with UPF....
[2025-08-28 09:44:17,033] root:ERROR:  UPF did not answer to N4 Association request from SMF....
[2025-08-28 09:44:17,070] root:DEBUG:  SMF is receiving heartbeats from UPF....
[2025-08-28 09:44:17,070] root:ERROR:  OAI 5G Core network may not be properly deployed....
telcomaan@oai-5gc-vm:~/oai-cn5g-fed/docker-compose$ docker logs oai-smf | grep -i "association setup"
[2025-08-28 11:43:33.855] [smf_n4 ] [info] Received N4 ASSOCIATION SETUP RESPONSE from an UPF
[2025-08-28 11:43:33.855] [smf_n4 ] [info] Received N4 ASSOCIATION SETUP RESPONSE
telcomaan@oai-5gc-vm:~/oai-cn5g-fed/docker-compose$ 
```



### 10) Summary of Actions: Replacing Open5GS with OAI-Core

You successfully executed a complete 5G core network swap, replacing a VM-based Open5GS with a containerized OAI 5G Core. The key steps you performed were:

*   **Deployment:** Deployed the OAI 5G Core (AMF, SMF, UPF, etc.) as Docker containers on a dedicated GCP virtual machine.
*   **Configuration & Integration:**
    *   Reconfigured the OAI-CU to point to the new OAI-AMF's IP address.
    *   Aligned OAI-Core's network parameters (PLMN, S-NSSAI, DNN) to match the RAN.
    *   Provisioned the UE's credentials in the OAI-Core's database.
    *   Exposed the necessary N2 (AMF) and N3 (UPF) ports and configured host-level `iptables` rules for NAT.

---

#### Status Report

##### What's Working: The Control Plane is Fully Functional

All signaling and session setup procedures are succeeding. The OAI-RAN (CU/DU) successfully connects to the OAI-AMF, and the UE completes its registration and authentication. The network correctly establishes a PDU session and assigns an IP address to the UE, creating the `oaitun_ue1` network interface.

##### What's Not Working: The User Plane is Unreachable

Despite a successful connection, there is zero data connectivity. A `ping` from the UE's `oaitun_ue1` interface fails with 100% packet loss.

*   **Root Cause:** The OAI-UPF container, by default, registers its **internal Docker IP address** (e.g., `192.168.70.x`) with the NRF. The SMF discovers this IP and instructs the OAI-CU to send all user data (GTP-U packets) to this address. However, this IP is private to the Docker network on the core VM and is **completely unreachable** from the RAN VMs, causing the data packets to be dropped before they ever reach the core network host.

---

#### Troubleshooting Steps Attempted (Unsuccessful)

To resolve the unreachable UPF IP issue, we systematically tried four different approaches, all of which failed to force the correct IP to be advertised to the RAN:

1.  **Host `iptables` and NAT Rules:** Initially, we suspected a faulty NAT configuration on the host. We corrected and applied the proper `iptables` rules, but `tcpdump` revealed that no user plane packets were even arriving at the host, proving this was not the root cause.
2.  **`upf.yaml` Configuration:** We attempted to force the UPF's N3 IP by adding an `ipv4_address` field directly into the `basic_nrf_config.yaml` file. This configuration was ignored, and the UPF continued to register its internal Docker IP.
3.  **`.env` File Override:** We created a `.env` file to define the `UPF_IPV4_ADDRESS_FOR_N3` variable, which is a standard method for injecting environment-specific settings. This change was also not reflected in the final CU configuration.
4.  **Direct `docker-compose.yaml` Environment Variable:** We injected the `UPF_IPV4_ADDRESS_FOR_N3` variable directly into the `environment` section for the `oai-upf` service in the `docker-compose-basic-nrf.yaml` file. This is the highest level of override, yet the CU logs confirmed it was still receiving the incorrect internal Docker IP for the user plane.

---
### logs

#### cu logs
```
telcomaan@oai-cu-vm:~/openairinterface5g/cmake_targets/ran_build/build$ sudo -E ./nr-softmodem -O /etc/oai/oai-cu.conf --sa --telnetsrv --telnetsrv.shrmod ci
CMDLINE: "./nr-softmodem" "-O" "/etc/oai/oai-cu.conf" "--sa" "--telnetsrv" "--telnetsrv.shrmod" "ci"
[CONFIG] function config_libconfig_init returned 0
[LOADER] library libtelnetsrv_gnb.so is not loaded: libtelnetsrv_gnb.so: cannot open shared object file: No such file or directory
[TELNETSRV] Telnet server: module 0 = telnet added to shell
policy set to other, priority 0
Error 3: No such process trying to get nice value of thread 998241984
[TELNETSRV] Telnet server: module 1 = softmodem added to shell
[TELNETSRV] couldn't find add_phy_cmds for module phy
[TELNETSRV] Telnet server: module 2 = loader added to shell
[TELNETSRV]
Initializing telnet server...
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
[GTPU]   Created gtpu instance id: 96
[UTIL]   threadCreate() for TASK_GNB_APP: creating thread with affinity ffffffff, priority 50
[NR_RRC]   Accepting new CU-UP ID 3584 name gNB-Eurecom-CU (assoc_id -1)
[NGAP]   Send NGSetupRequest to AMF
[NGAP]   3584 -> 0000e000
[UTIL]   threadCreate() for TASK_GTPV1_U: creating thread with affinity ffffffff, priority 50
[UTIL]   threadCreate() for TASK_CU_F1: creating thread with affinity ffffffff, priority 50
[NGAP]   Served GUAMIs for AMF (no name) (assoc_id=24):
[NGAP]    GUAMI:
[NGAP]      PLMN: MCC=999, MNC=70
[NGAP]      AMF Region ID: 1
[NGAP]      AMF Set ID: 1
[NGAP]      AMF Pointer: 1
[NGAP]   Supported PLMN 0: MCC=999 MNC=70
[NGAP]   Supported slice (PLMN 0): SST=0x01 SD=000
[NGAP]   Received NGSetupResponse from AMF
[UTIL]   threadCreate() for time source realtime: creating thread with affinity ffffffff, priority 2
[F1AP]   Starting F1AP at CU
[GNB_APP]   [gNB 0] Received NGAP_REGISTER_GNB_CNF: associated AMF 1
[F1AP]   F1AP_CU_SCTP_REQ(create socket) for 172.17.0.93 len 12
[GTPU]   Initializing UDP for local address 172.17.0.93 with port 2153
[GTPU]   Created gtpu instance id: 97
[UTIL]   time manager configuration: [time source: reatime] [mode: standalone] [server IP: 127.0.0.1} [server port: 7374] (server IP/port not used)
TYPE <CTRL-C> TO TERMINATE
[F1AP]   CU Task Received SCTP_NEW_ASSOCIATION_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   CU Task Received SCTP_NEW_ASSOCIATION_RESP for instance 0: sending SCTP message via assoc_id 0
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[F1AP]   CU_handle_F1_SETUP_REQUEST
[NR_RRC]   Received F1 Setup Request from gNB_DU 3585 (oai-cu-cp) on assoc_id 25
[NR_RRC]   Accepting DU 3585 (oai-cu-cp), sending F1 Setup Response
[NR_RRC]   DU uses RRC version 17.3.0
[F1AP]   CU Task Received F1AP_SETUP_RESP for instance 0: sending SCTP message via assoc_id 25
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[F1AP]   [SCTP 25] CU_handle_gNB_DU_CONFIGURATION_UPDATE
[F1AP]   Sending F1AP_GNB_DU_CONFIGURATION_UPDATE ITTI message
[NR_RRC]   cell PLMN 999.70 Cell ID 12345678 is in service
[F1AP]   CU Task Received F1AP_GNB_DU_CONFIGURATION_UPDATE_ACKNOWLEDGE for instance 0: sending SCTP message via assoc_id 25
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[NR_RRC]   Decoding CCCH: RNTI 15cc, payload_size 6
[NR_RRC]   [--] (cellID 0, UE ID 1 RNTI 15cc) Create UE context: CU UE ID 1 DU UE ID 5580 (rnti: 15cc, random ue id 344fc41eec000000)
[RRC]   activate SRB 1 of UE 1
[NR_RRC]   [DL] (cellID bc614e, UE ID 1 RNTI 15cc) Send RRC Setup
[F1AP]   CU Task Received F1AP_DL_RRC_MESSAGE for instance 0: sending SCTP message via assoc_id 25
[F1AP]   CU send DL_RRC_MESSAGE_TRANSFER
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[F1AP]   CU_handle_UL_RRC_MESSAGE_TRANSFER
[F1AP]   UL RRC MESSAGE for SRB 1 in DCCH
[F1AP]   [UE 1] calling pdcp_data_ind for SRB 1 with size 39 (DCCH)
[NR_RRC]   [UL] (cellID bc614e, UE ID 1 RNTI 15cc) Received RRCSetupComplete (RRC_CONNECTED reached)
[NGAP]   Selected PLMN in the NG Initial UE Message: MCC 999, MNC 70
[NGAP]   UE 1: Chose AMF 'OAI-AMF' (assoc_id 24) through selected PLMN MCC=999 MNC=70
[NGAP]   Create UE context (ID 1) for AMF 'OAI-AMF' (assoc_id 24)
[NR_RRC]   [DL] (cellID bc614e, UE ID 1 RNTI 15cc) Send DL Information Transfer [42 bytes]
[F1AP]   CU Task Received F1AP_DL_RRC_MESSAGE for instance 0: sending SCTP message via assoc_id 25
[F1AP]   CU send DL_RRC_MESSAGE_TRANSFER
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[F1AP]   CU_handle_UL_RRC_MESSAGE_TRANSFER
[F1AP]   UL RRC MESSAGE for SRB 1 in DCCH
[F1AP]   [UE 1] calling pdcp_data_ind for SRB 1 with size 30 (DCCH)
[NR_RRC]   [UL] (cellID bc614e, UE ID 1 RNTI 15cc) Received RRC UL Information Transfer [24 bytes]
[NR_RRC]   [DL] (cellID bc614e, UE ID 1 RNTI 15cc) Send DL Information Transfer [21 bytes]
[F1AP]   CU Task Received F1AP_DL_RRC_MESSAGE for instance 0: sending SCTP message via assoc_id 25
[F1AP]   CU send DL_RRC_MESSAGE_TRANSFER
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[F1AP]   CU_handle_UL_RRC_MESSAGE_TRANSFER
[F1AP]   UL RRC MESSAGE for SRB 1 in DCCH
[F1AP]   [UE 1] calling pdcp_data_ind for SRB 1 with size 66 (DCCH)
[NR_RRC]   [UL] (cellID bc614e, UE ID 1 RNTI 15cc) Received RRC UL Information Transfer [60 bytes]
[NGAP]   NGAP_FIND_PROTOCOLIE_BY_ID ie is NULL (searching for ie: 110)
[NGAP]   NGAP_FIND_PROTOCOLIE_BY_ID ie is NULL (searching for ie: 71)
[NGAP]   AllowedNSSAI.list.count 1
[NR_RRC]   [--] (cellID bc614e, UE ID 1 RNTI 15cc) Selected security algorithms: ciphering 0, integrity 2
[NR_RRC]   [UE 15cc] Saved security key F9
[NR_RRC]   UE 1 Logical Channel DL-DCCH, Generate SecurityModeCommand (bytes 3)
[F1AP]   CU Task Received F1AP_DL_RRC_MESSAGE for instance 0: sending SCTP message via assoc_id 25
[F1AP]   CU send DL_RRC_MESSAGE_TRANSFER
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[F1AP]   CU_handle_UL_RRC_MESSAGE_TRANSFER
[F1AP]   UL RRC MESSAGE for SRB 1 in DCCH
[F1AP]   [UE 1] calling pdcp_data_ind for SRB 1 with size 8 (DCCH)
[NR_RRC]   [UL] (cellID bc614e, UE ID 1 RNTI 15cc) Received Security Mode Complete
[NR_RRC]   UE 1: Logical Channel DL-DCCH, Generate NR UECapabilityEnquiry (bytes 8, xid 1)
[F1AP]   CU Task Received F1AP_DL_RRC_MESSAGE for instance 0: sending SCTP message via assoc_id 25
[F1AP]   CU send DL_RRC_MESSAGE_TRANSFER
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[F1AP]   CU_handle_UL_RRC_MESSAGE_TRANSFER
[F1AP]   UL RRC MESSAGE for SRB 1 in DCCH
[F1AP]   [UE 1] calling pdcp_data_ind for SRB 1 with size 20 (DCCH)
[NR_RRC]   [UL] (cellID bc614e, UE ID 1 RNTI 15cc) Received UE capabilities
[NR_RRC]   Send message to ngap: NGAP_UE_CAPABILITIES_IND
[NR_RRC]   [DL] (cellID bc614e, UE ID 1 RNTI 15cc) Send DL Information Transfer [53 bytes]
[NR_RRC]   Send message to sctp: NGAP_InitialContextSetupResponse
[F1AP]   CU Task Received F1AP_DL_RRC_MESSAGE for instance 0: sending SCTP message via assoc_id 25
[F1AP]   CU send DL_RRC_MESSAGE_TRANSFER
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[F1AP]   CU_handle_UL_RRC_MESSAGE_TRANSFER
[F1AP]   UL RRC MESSAGE for SRB 1 in DCCH
[F1AP]   [UE 1] calling pdcp_data_ind for SRB 1 with size 19 (DCCH)
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[F1AP]   CU_handle_UL_RRC_MESSAGE_TRANSFER
[F1AP]   UL RRC MESSAGE for SRB 1 in DCCH
[F1AP]   [UE 1] calling pdcp_data_ind for SRB 1 with size 46 (DCCH)
[NR_RRC]   [UL] (cellID bc614e, UE ID 1 RNTI 15cc) Received RRC UL Information Transfer [13 bytes]
[NR_RRC]   [UL] (cellID bc614e, UE ID 1 RNTI 15cc) Received RRC UL Information Transfer [40 bytes]
[NGAP]   PDUSESSIONSetup initiating message
[NR_RRC]   UE 1: received PDU Session Resource Setup Request
[NR_RRC]   Bearer Context Setup: PDU Session ID=10, incoming TEID=0x00000001, Addr=192.168.70.136
[NR_RRC]   UE 1: configure DRB ID 1 for PDU session ID 10
[NR_RRC]   [--] (cellID bc614e, UE ID 1 RNTI 15cc) selecting CU-UP ID 3584 based on exact NSSAI match (1:0xffffff)
[RRC]   UE 1 associating to CU-UP assoc_id -1 out of 1 CU-UPs
[E1AP]   UE 1: add PDU session ID 10 (1 bearers)
[GTPU]   [96] Created tunnel for UE ID 1, teid for incoming: bf3a879b, teid for outgoing 1 to remote IPv4: 192.168.70.136, IPv6 ::
[PDCP]   added drb 1 to UE ID 1
[SDAP]   Default DRB for the created SDAP entity: 1
[GTPU]   [97] Created tunnel for UE ID 1, teid for incoming: 8266378e, teid for outgoing ffff to remote IPv4: 0.0.0.0, IPv6 ::
[RRC]   activate SRB 2 of UE 1
[RRC]   UE 1 trigger UE context setup request with 1 DRBs
[F1AP]   CU Task Received F1AP_UE_CONTEXT_SETUP_REQ for instance 0: sending SCTP message via assoc_id 25
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[RRC]   UE 15cc replacing existing CellGroupConfig with new one received from DU
[E1AP]   UE 1: updating PDU session ID 10 (1 bearers)
[PDCP]   DRB 1 re-established
[GTPU]   [97] Tunnel Outgoing TEID updated to 5bb3310e and address to 5c0011ac
[NR_RRC]   [DL] (cellID bc614e, UE ID 1 RNTI 15cc) Generate RRCReconfiguration (bytes 318, xid 3)
[F1AP]   CU Task Received F1AP_DL_RRC_MESSAGE for instance 0: sending SCTP message via assoc_id 25
[RRC]   UE 1: PDU session ID 10 modified 1 bearers
[F1AP]   CU send DL_RRC_MESSAGE_TRANSFER
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0
[F1AP]   CU_handle_UL_RRC_MESSAGE_TRANSFER
[F1AP]   UL RRC MESSAGE for SRB 1 in DCCH
[F1AP]   [UE 1] calling pdcp_data_ind for SRB 1 with size 8 (DCCH)
[NR_RRC]   [UL] (cellID bc614e, UE ID 1 RNTI 15cc) Received RRCReconfigurationComplete
[NR_RRC]   PDU Session Setup Response: ID=10, outgoing TEID=0xbf3a879b, Addr=172.17.0.93
[NR_RRC]   NGAP_PDUSESSION_SETUP_RESP: sending the message
[F1AP]   CU Task Received F1AP_UE_CONTEXT_MODIFICATION_REQ for instance 0: sending SCTP message via assoc_id 25
[NGAP]   Encoded PDU Session Transfer (10): TEID=0xbf3a879b, Addr=172.17.0.93
[F1AP]   CU Task Received SCTP_DATA_IND for instance 0: sending SCTP message via assoc_id 0
[F1AP]   Calling handler with instance 0


```

#### du logs
```
telcomaan@oai-du0-vm:~/openairinterface5g/cmake_targets/ran_build/build$ sudo ./nr-softmodem -O /etc/oai/oai-du0.conf --rfsim --sa
CMDLINE: "./nr-softmodem" "-O" "/etc/oai/oai-du0.conf" "--rfsim" "--sa"
[CONFIG] function config_libconfig_init returned 0
[UTIL]   running in SA mode (no --phy-test, --do-ra, --nsa option present)
[OPT]   OPT disabled
[HW]   Version: Branch: develop Abrev. Hash: b763692792 Date: Fri Aug 1 04:40:24 2025 +0000
[GNB_APP]   Initialized RAN Context: RC.nb_nr_inst = 1, RC.nb_nr_macrlc_inst = 1, RC.nb_nr_L1_inst = 1, RC.nb_RU = 1, RC.nb_nr_CC[0] = 1
[NR_PHY]   Initializing gNB RAN context: RC.nb_nr_L1_inst = 1
[NR_PHY]   Registered with MAC interface module (0x65063a47d670)
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
[UTIL]   threadCreate() for TASK_GTPV1_U: creating thread with affinity ffffffff, priority 50
[UTIL]   threadCreate() for TASK_DU_F1: creating thread with affinity ffffffff, priority 50
[UTIL]   threadCreate() for time source iq samples: creating thread with affinity ffffffff, priority 2
[F1AP]   Starting F1AP at DU
[F1AP]   F1-C DU IPaddr 172.17.0.92, connect to F1-C CU 172.17.0.93, binding GTP to 172.17.0.92
[GTPU]   Initializing UDP for local address 172.17.0.92 with port 2153
[GTPU]   Created gtpu instance id: 94
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
Initializing random number generator, seed 13527362091981151034
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
[PHY]   got sync (ru_thread)
[PHY]   got sync (L1_stats_thread)
[HW]   Trying to connect to 172.17.0.91:4043
TYPE <CTRL-C> TO TERMINATE
[HW]   Connection to 172.17.0.91:4043 established
[PHY]   RU 0 rf device ready
[PHY]   RU 0 RF started cpu_meas_enabled 0
[NR_MAC]   Frame.Slot 256.0

[PHY]   Command line parameters for OAI UE: -C 3619200000 -r 106 --numerology 1 --ssb 516
[NR_PHY]   [RAPROC] 343.19 Initiating RA procedure with preamble 55, energy 43.2 dB (I0 0, thres 200), delay 0 start symbol 4 freq index 0
[NR_MAC]   343.19 UE RA-RNTI 010f TC-RNTI 15cc: initiating RA procedure
[NR_MAC]   UE 15cc: Msg3 scheduled at 344.17 (344.7 TDA 3) start 0 RBs 8
[NR_MAC]   UE 15cc: 344.7 Generating RA-Msg2 DCI, RA RNTI 0x10f, state 1, preamble_index(RAPID) 55, timing_offset = 0 (estimated distance 0.0 [m])
[NR_MAC]   344.7 Send RAR to RA-RNTI 010f
[NR_MAC]    344.17 PUSCH with TC_RNTI 0x15cc received correctly
[MAC]   [RAPROC] Received SDU for CCCH length 6 for UE 15cc
[RLC]   Activated srb0 for UE 5580
[RLC]   Added srb 1 to UE 5580
[NR_MAC]   Activating scheduling Msg4 for TC_RNTI 0x15cc (state WAIT_Msg3)
[NR_MAC]   Cannot find free vrb_map for RNTI 15cc!
[NR_MAC]   UE 15cc Generate Msg4: feedback at  346. 7, payload 161 bytes, next state nrRA_WAIT_Msg4_MsgB_ACK
[NR_MAC]    346. 7 UE 15cc: Received Ack of Msg4. CBRA procedure succeeded (UE Connected)
[NR_MAC]   Adding new UE context with RNTI 0x15cc
[NR_MAC]   Frame.Slot 384.0
UE RNTI 15cc CU-UE-ID 1 in-sync PH 0 dB PCMAX 0 dBm, average RSRP -44 (4 meas)
UE 15cc: dlsch_rounds 1/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE 15cc: ulsch_rounds 37/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.07290 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 50.5 dB
UE 15cc: MAC:    TX              0 RX            111 bytes
UE 15cc: LCID 1: TX              0 RX              0 bytes

[RLC]   Added srb 2 to UE 5580
[RLC]   Added drb 1 to UE 5580
[RLC]   Added DRB to UE 5580
[GTPU]   [94] Created tunnel for UE ID 5580, teid for incoming: 5bb3310e, teid for outgoing 8266378e to remote IPv4: 172.17.0.93, IPv6 ::
[NR_MAC]   DU received confirmation of successful RRC Reconfiguration
[NR_MAC]   Frame.Slot 512.0
UE RNTI 15cc CU-UE-ID 1 in-sync PH 52 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 15cc: dlsch_rounds 17/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.05314 MCS (0) 0
UE 15cc: ulsch_rounds 169/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.02288 MCS (0) 8 (Qm 2 deltaMCS 0 dB) NPRB 12  SNR 51.0 dB
UE 15cc: MAC:    TX            739 RX           1375 bytes
UE 15cc: LCID 1: TX            527 RX            350 bytes
UE 15cc: LCID 2: TX              0 RX              0 bytes
UE 15cc: LCID 4: TX              3 RX             64 bytes

[NR_MAC]   Frame.Slot 640.0
UE RNTI 15cc CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 15cc: dlsch_rounds 18/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.04783 MCS (0) 0
UE 15cc: ulsch_rounds 182/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00581 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 15cc: MAC:    TX            760 RX           2050 bytes
UE 15cc: LCID 1: TX            527 RX            350 bytes
UE 15cc: LCID 2: TX              0 RX              0 bytes
UE 15cc: LCID 4: TX              3 RX             64 bytes

```

#### ue logs
```
telcomaan@oai-nr-ue-vm:~/openairinterface5g/cmake_targets/ran_build/build$ sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --rfsim -O /etc/oai/nr-ue.conf
CMDLINE: "./nr-uesoftmodem" "-r" "106" "--numerology" "1" "--band" "78" "-C" "3619200000" "--rfsim" "-O" "/etc/oai/nr-ue.conf"
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
UE threads created by 87626
TYPE <CTRL-C> TO TERMINATE
[HW]   Running as server waiting opposite rfsimulators to connect
Initializing random number generator, seed 14879428019673314987
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
[PHY]   [UE 0] Measured Carrier Frequency offset 6 Hz
[PHY]   Initial sync successful, PCI: 0
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_freq 3619200006 Hz, rx_freq 3619200006 Hz, tune_offset 0
[PHY]   Got synch: hw_slot_offset 2, carrier off 6 Hz, rxgain 0.000000 (DL 3619200006.000000 Hz, UL 3619200006.000000 Hz)
[PHY]   UE synchronized! decoded_frame_rx=292 UE->init_sync_frame=1 trashed_frames=48
[PHY]   Resynchronizing RX by 30720 samples
[HW]   received write reorder clear context
[NR_RRC]   SIB1 decoded
[NR_MAC]   TDD period index = 6, based on the sum of dl_UL_TransmissionPeriodicity from Pattern1 (5.000000 ms) and Pattern2 (0.000000 ms): Total = 5.000000 ms
[NR_MAC]   Set TDD configuration period to: 8 DL slots, 3 UL slots, 10 slots per period (NR_TDD_UL_DL_Pattern is 7 DL slots, 2 UL slots, 6 DL symbols, 4 UL symbols)
[NR_MAC]   Configured 1 TDD patterns (total slots: pattern1 = 10, pattern2 = 0)
[PHY]   N_TA_offset changed from 0 to 800
[MAC]   Initialization of 4-Step CBRA procedure
[NR_MAC]   PRACH scheduler: Selected RO Frame 343, Slot 19, Symbol 4, Fdm 0
[PHY]   PRACH [UE 0] in frame.slot 343.19, placing PRACH in position 2828, Msg1/MsgA-Preamble frequency start 0 (k1 0), preamble_offset 13, first_nonzero_root_idx 0, preambleIndex = 55
[PHY]   RAR-Msg2 decoded
[NR_MAC]   [UE 0][RAPROC][RA-RNTI 010f] Got BI RAR subPDU 5 ms
[NR_MAC]   [UE 0][RAPROC][RA-RNTI 010f] Got RAPID RAR subPDU
[NR_MAC]   [UE 0][RAPROC][344.7] Found RAR with the intended RAPID 55
[MAC]   received TA command 31
[NR_MAC]   [RAPROC][344.17] RA-Msg3 transmitted
[MAC]   [UE 0][346.1][RAPROC] 4-Step RA procedure succeeded. CBRA: Contention Resolution is successful.
[NR_RRC]   [UE0][RAPROC] Logical Channel DL-CCCH (SRB0), Received NR_RRCSetup
[RLC]   Added srb 1 to UE 0
[NR_RRC]   State = NR_RRC_CONNECTED
[NAS]   [UE 0] Received NR_NAS_CONN_ESTABLISH_IND: asCause 0
[NAS]   Generate Initial NAS Message: Registration Request
[NR_RRC]   [UE 0][RAPROC] Logical Channel UL-DCCH (SRB1), Generating RRCSetupComplete (bytes33)
[MAC]   [UE 0] Applying CellGroupConfig from gNodeB
[NR_MAC]   UE 0 RNTI 15cc stats sfn: 384.8, cumulated bad DCI 0
    DL harq: 1/0
    Ul harq: 38/0 avg code rate 0.1, avg bit/symbol 3.0, avg per TB: (nb RBs 5.1, nb symbols 3.0)
[NAS]   [UE 0] Received NAS_DOWNLINK_DATA_IND type FGS_AUTHENTICATION_REQUEST with length 42
kausf:d9 71 c8 f9 df 95 36 90 8e c2 35 13 18 e0 3d 8d 20 cf 78 17 91 1c 15 f0 33 85 15 c7 a5 49 da e7
kseaf:73 2c b0 35 bd c9 21 e4 c2 fc 55 3e e 2e b6 a2 b4 8a 68 2e 63 f8 66 a3 f1 ac 7b e2 34 31 f5 32
kamf:62 41 1f e5 8f a4 d dd 7c da 91 e3 ff b 1e 7b 6f d2 77 81 15 60 a2 ba 77 4e db de 87 4c b5 c7
[NAS]   [UE 0] Received NAS_DOWNLINK_DATA_IND type FGS_SECURITY_MODE_COMMAND with length 21
knas_int: 35 1 fe 9b e7 75 74 8 68 23 b5 b1 67 5c 3d 75
knas_enc: 3a 36 70 ae 59 3b 7a cf 64 95 5b 8a 57 d7 96 2c
[NAS]   Generate Initial NAS Message: Registration Request
mac d1 74 90 1d
[NR_RRC]   Received securityModeCommand (gNB 0)
[NR_RRC]   Receiving from SRB1 (DL-DCCH), Processing securityModeCommand
[NR_RRC]   Security algorithm is set to nea0
[NR_RRC]   Integrity protection algorithm is set to nia2
[NR_RRC]   deriving kRRCenc, kRRCint from KgNB=f9 3f 16 1f 12 7a a0 57 cd 37 9c d1 15 d4 b4 18 e1 db e7 ac 68 c2 db 8d c1 9c 8f 7f b5 bd 9b 8e
[NR_RRC]   Receiving from SRB1 (DL-DCCH), encoding securityModeComplete, rrc_TransactionIdentifier: 0
[NR_RRC]   securityModeComplete payload: 28 00 00 00 00 00 00 00 c0 56 00 38 66 70 00 00
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
[NAS]   [UE 0] Received NAS_DOWNLINK_DATA_IND type FGS_REGISTRATION_ACCEPT with length 53
[NAS]   Received Registration Accept with result 3GPP
[NAS]   SMS not allowed in 5GS Registration Result
[NR_RRC]   5G-GUTI: AMF pointer 1, AMF Set ID 1, 5G-TMSI 1
mac c9 12 5b 79
[NAS]   Send NAS_UPLINK_DATA_REQ message(RegistrationComplete)
mac c7 c6 6d e1
[NAS]   Send NAS_UPLINK_DATA_REQ message(PduSessionEstablishRequest)
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
[NAS]   [UE 0] Received NAS_CONN_ESTABLI_CNF: errCode 1, length 103
[MAC]   [UE 0] Applying CellGroupConfig from gNodeB
[NAS]   Received PDU Session Establishment Accept, UE IPv4: 12.1.2.2
Unknown IEI 129
[OIP]   Interface oaitun_ue1 successfully configured, IPv4 12.1.2.2, IPv6 (null)
[UTIL]   threadCreate() for ue_tun_read_0_p10: creating thread with affinity ffffffff, priority 1
[NR_MAC]   UE 0 RNTI 15cc stats sfn: 512.8, cumulated bad DCI 0
    DL harq: 17/0
    Ul harq: 170/0 avg code rate 0.3, avg bit/symbol 2.2, avg per TB: (nb RBs 6.0, nb symbols 3.0)
[NR_MAC]   UE 0 RNTI 15cc stats sfn: 640.8, cumulated bad DCI 0
    DL harq: 18/0
    Ul harq: 183/0 avg code rate 0.4, avg bit/symbol 2.2, avg per TB: (nb RBs 5.9, nb symbols 3.7)

```

#### ue-log for tun and ping
```
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
       valid_lft 3408sec preferred_lft 3408sec
    inet6 fe80::4001:acff:fe11:5b/64 scope link
       valid_lft forever preferred_lft forever
8: oaitun_ue1: <POINTOPOINT,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 500
    link/none
    inet 12.1.2.3/24 scope global oaitun_ue1
       valid_lft forever preferred_lft forever
    inet6 fe80::f49:c048:78e8:f167/64 scope link stable-privacy
       valid_lft forever preferred_lft forever
telcomaan@oai-nr-ue-vm:~$ ping -I oaitun_ue1 8.8.8.8
PING 8.8.8.8 (8.8.8.8) from 12.1.2.3 oaitun_ue1: 56(84) bytes of data.
^C
--- 8.8.8.8 ping statistics ---
5 packets transmitted, 0 received, 100% packet loss, time 4100ms

telcomaan@oai-nr-ue-vm:~$

```

#### amf-logs
```
telcomaan@oai-5gc-vm:~/oai-cn5g-fed/docker-compose$ docker logs oai-amf
Trying to read .yaml configuration file
LTTNG Tracing disabled at build-time!
[2025-08-28 11:43:33.216] [amf_app] [start] Options parsed!
[2025-08-28 11:43:33.218] [system] [debug] Parsing the configuration file, file type YAML.
[2025-08-28 11:43:33.218] [config ] [info] Reading NF configuration from /openair-amf/etc/config.yaml
[2025-08-28 11:43:33.258] [config ] [debug] Unknown NF upf in configuration. Ignored
[2025-08-28 11:43:33.258] [config ] [debug] Unknown NF udr in configuration. Ignored
[2025-08-28 11:43:33.259] [config ] [debug] Validating configuration of log_level
[2025-08-28 11:43:33.275] [config ] [info] ==== OPENAIRINTERFACE amf vBranch: HEAD Abrev. Hash: 58f77cfb Date: Fri Aug 30 10:39:14 2024 +0000 ====
[2025-08-28 11:43:33.275] [config ] [info] Basic Configuration:
[2025-08-28 11:43:33.275] [config ] [info]   - log_level..................................: info
[2025-08-28 11:43:33.275] [config ] [info]   - register_nf................................: Yes
[2025-08-28 11:43:33.275] [config ] [info]   - http_version...............................: 2
[2025-08-28 11:43:33.275] [config ] [info]   - HTTP Request Timeout.......................: 3000 (ms)
[2025-08-28 11:43:33.275] [config ] [info]   AMF:
[2025-08-28 11:43:33.275] [config ] [info]     - host.....................................: oai-amf
[2025-08-28 11:43:33.275] [config ] [info]     - SBI
[2025-08-28 11:43:33.275] [config ] [info]       + URL....................................: http://oai-amf:8080
[2025-08-28 11:43:33.275] [config ] [info]       + API Version............................: v1
[2025-08-28 11:43:33.275] [config ] [info]       + IPv4 Address ..........................: 192.168.70.135
[2025-08-28 11:43:33.275] [config ] [info]     - N2
[2025-08-28 11:43:33.275] [config ] [info]       + Port...................................: 38412
[2025-08-28 11:43:33.275] [config ] [info]       + IPv4 Address ..........................: 192.168.70.135
[2025-08-28 11:43:33.275] [config ] [info]       + MTU....................................: 1500
[2025-08-28 11:43:33.275] [config ] [info]       + Interface name: .......................: eth0
[2025-08-28 11:43:33.275] [config ] [info]     - Instance ID..............................: 1
[2025-08-28 11:43:33.275] [config ] [info]     - PID Directory............................: /var/run
[2025-08-28 11:43:33.275] [config ] [info]     - AMF Name.................................: OAI-AMF
[2025-08-28 11:43:33.275] [config ] [info]     - Support Features Options
[2025-08-28 11:43:33.275] [config ] [info]       + Enable Simple Scenario.................: No
[2025-08-28 11:43:33.275] [config ] [info]       + Enable NSSF............................: No
[2025-08-28 11:43:33.275] [config ] [info]       + Enable SMF Selection...................: Yes
[2025-08-28 11:43:33.275] [config ] [info]     - Relative Capacity........................: 30
[2025-08-28 11:43:33.275] [config ] [info]     - Emergency Support........................: No
[2025-08-28 11:43:33.275] [config ] [info]     - Served GUAMI List
[2025-08-28 11:43:33.275] [config ] [info]       + MCC....................................: 999
[2025-08-28 11:43:33.275] [config ] [info]         MNC....................................: 70
[2025-08-28 11:43:33.275] [config ] [info]         AMF Region ID..........................: 01
[2025-08-28 11:43:33.275] [config ] [info]         AMF Set ID.............................: 001
[2025-08-28 11:43:33.275] [config ] [info]         AMF Pointer............................: 01
[2025-08-28 11:43:33.275] [config ] [info]     - PLMN Support List
[2025-08-28 11:43:33.275] [config ] [info]       + MCC....................................: 999
[2025-08-28 11:43:33.275] [config ] [info]         MNC....................................: 70
[2025-08-28 11:43:33.275] [config ] [info]         TAC....................................: 1
[2025-08-28 11:43:33.275] [config ] [info]             - NSSAI
[2025-08-28 11:43:33.275] [config ] [info]                   + SST........................: 1
[2025-08-28 11:43:33.275] [config ] [info]                     SD.........................: FFFFFF
[2025-08-28 11:43:33.275] [config ] [info]     - Supported Integrity Algorithms
[2025-08-28 11:43:33.275] [config ] [info]       + .......................................: NIA0
[2025-08-28 11:43:33.276] [config ] [info]       + .......................................: NIA1
[2025-08-28 11:43:33.276] [config ] [info]       + .......................................: NIA2
[2025-08-28 11:43:33.276] [config ] [info]     - Supported Encryption Algorithms
[2025-08-28 11:43:33.276] [config ] [info]       + .......................................: NEA0
[2025-08-28 11:43:33.276] [config ] [info]       + .......................................: NEA1
[2025-08-28 11:43:33.276] [config ] [info]       + .......................................: NEA2
[2025-08-28 11:43:33.276] [config ] [info] Peer NF Configuration:
[2025-08-28 11:43:33.276] [config ] [info]   NRF:
[2025-08-28 11:43:33.276] [config ] [info]     - host.....................................: oai-nrf
[2025-08-28 11:43:33.276] [config ] [info]     - SBI
[2025-08-28 11:43:33.276] [config ] [info]       + URL....................................: http://oai-nrf:8080
[2025-08-28 11:43:33.276] [config ] [info]       + API Version............................: v1
[2025-08-28 11:43:33.276] [config ] [info]   UDM:
[2025-08-28 11:43:33.276] [config ] [info]     - host.....................................: oai-udm
[2025-08-28 11:43:33.276] [config ] [info]     - SBI
[2025-08-28 11:43:33.276] [config ] [info]       + URL....................................: http://oai-udm:8080
[2025-08-28 11:43:33.276] [config ] [info]       + API Version............................: v1
[2025-08-28 11:43:33.276] [config ] [info]   AUSF:
[2025-08-28 11:43:33.276] [config ] [info]     - host.....................................: oai-ausf
[2025-08-28 11:43:33.276] [config ] [info]     - SBI
[2025-08-28 11:43:33.276] [config ] [info]       + URL....................................: http://oai-ausf:8080
[2025-08-28 11:43:33.276] [config ] [info]       + API Version............................: v1
[2025-08-28 11:43:33.276] [itti] [start] Starting...
[2025-08-28 11:43:33.276] [itti] [start] Started
[2025-08-28 11:43:33.276] [amf_sbi] [info] HTTP Client successfully initiated on interface eth0 with timeout 3000 ms, HTTP version 2
[2025-08-28 11:43:33.276] [amf_app] [start] Creating AMF application functionality layer
[2025-08-28 11:43:33.276] [itti] [info] Starting timer_manager_task
[2025-08-28 11:43:33.277] [itti] [warning] Could not set schedparam to ITTI task 0, err=1
[2025-08-28 11:43:33.278] [amf_n1] [start] amf_n1 started
[2025-08-28 11:43:33.279] [sctp] [info] Created socket (3)
[2025-08-28 11:43:33.280] [ngap] [info] Set N2 AMF IPv4 Addr 192.168.70.135, port 38412
[2025-08-28 11:43:33.280] [sctp] [info] Create pthread to receive SCTP message
[2025-08-28 11:43:33.282] [amf_n2] [start] amf_n2 started
[2025-08-28 11:43:33.283] [amf_sbi] [start] amf_sbi started
[2025-08-28 11:43:33.285] [amf_app] [start] Started timer (1)
[2025-08-28 11:43:33.285] [amf_server] [info] HTTP2 server being started
[2025-08-28 11:43:33.286] [amf_sbi] [info] Receive Register NF Instance Request, handling ...
[2025-08-28 11:43:33.286] [amf_sbi] [info] Send HTTP message to http://oai-nrf:8080/nnrf-nfm/v1/nf-instances/b9f08296-e89f-4a03-b494-162dc784895d
[2025-08-28 11:43:33.286] [amf_sbi] [info] HTTP message Body: {"amfInfo":{"amfRegionId":"01","amfSetId":"001","guamiList":[{"amfId":"010041","plmnId":{"mcc":"999","mnc":"70"}}]},"capacity":100,"custom_info":null,"heartBeatTimer":50,"ipv4Addresses":["192.168.70.135"],"nfInstanceId":"b9f08296-e89f-4a03-b494-162dc784895d","nfInstanceName":"OAI-AMF","nfServices":[{"ipEndPoints":[{"ipv4Address":"192.168.70.135","port":8080,"transport":"TCP"}],"nfServiceStatus":"REGISTERED","scheme":"http","serviceInstanceId":"namf_communication","serviceName":"namf_communication","versions":[{"apiFullVersion":"1.0.0","apiVersionInUri":"v1"}]}],"nfStatus":"REGISTERED","nfType":"AMF","priority":1,"sNssais":[{"sd":"FFFFFF","sst":1}]}
[2025-08-28 11:43:33.297] [amf_sbi] [info] Get response with HTTP code (201)
[2025-08-28 11:43:43.300] [amf_sbi] [info] Receive Update NF Instance Request, handling ...
[2025-08-28 11:43:43.301] [amf_sbi] [info] Send HTTP message to http://oai-nrf:8080/nnrf-nfm/v1/nf-instances/b9f08296-e89f-4a03-b494-162dc784895d
[2025-08-28 11:43:43.301] [amf_sbi] [info] HTTP message Body: [{"op":"replace","path":"/nfStatus","value":"REGISTERED"}]
[2025-08-28 11:43:43.402] [amf_sbi] [info] Get response with HTTP code (204)
[2025-08-28 11:43:43.407] [amf_sbi] [info] Could not get JSON content from the response
[2025-08-28 11:43:53.288] [amf_app] [info] 
   |------------------------------------------------------------------------------------------------------------------------------------------------------------|
   |----------------------------------------------------------------------gNBs' Information---------------------------------------------------------------------|
   |  Index |               Status               |              Global Id             |              gNB Name              |                PLMN                |
   |    -   |                  -                 |                  -                 |                  -                 |                  -                 |
   |------------------------------------------------------------------------------------------------------------------------------------------------------------|

   |-----------------------------------------------------------------------------------------------------------------------------------------------------------|
   |---------------------------------------------------------------------UEs' Information----------------------------------------------------------------------|
   |  Index |     5GMM State     |        IMSI        |        GUTI        |   RAN UE NGAP ID   |   AMF UE NGAP ID   |        PLMN        |       Cell Id      |
   |    -   |          -         |          -         |          -         |          -         |          -         |          -         |          -         |
   |-----------------------------------------------------------------------------------------------------------------------------------------------------------|

[2025-08-28 11:43:53.415] [amf_sbi] [info] Receive Update NF Instance Request, handling ...
[2025-08-28 11:43:53.415] [amf_sbi] [info] Send HTTP message to http://oai-nrf:8080/nnrf-nfm/v1/nf-instances/b9f08296-e89f-4a03-b494-162dc784895d
[2025-08-28 11:43:53.415] [amf_sbi] [info] HTTP message Body: [{"op":"replace","path":"/nfStatus","value":"REGISTERED"}]
[2025-08-28 11:43:53.429] [amf_sbi] [info] Get response with HTTP code (204)
[2025-08-28 11:43:53.429] [amf_sbi] [info] Could not get JSON content from the response
[2025-08-28 11:44:03.431] [amf_sbi] [info] Receive Update NF Instance Request, handling ...
[2025-08-28 11:44:03.431] [amf_sbi] [info] Send HTTP message to http://oai-nrf:8080/nnrf-nfm/v1/nf-instances/b9f08296-e89f-4a03-b494-162dc784895d
[2025-08-28 11:44:03.431] [amf_sbi] [info] HTTP message Body: [{"op":"replace","path":"/nfStatus","value":"REGISTERED"}]
[2025-08-28 11:44:03.434] [amf_sbi] [info] Get response with HTTP code (204)
[2025-08-28 11:44:03.435] [amf_sbi] [info] Could not get JSON content from the response
[2025-08-28 11:44:13.288] [amf_app] [info] 
   |------------------------------------------------------------------------------------------------------------------------------------------------------------|
   |----------------------------------------------------------------------gNBs' Information---------------------------------------------------------------------|
   |  Index |               Status               |              Global Id             |              gNB Name              |                PLMN                |
   |    -   |                  -                 |                  -                 |                  -                 |                  -                 |
   |------------------------------------------------------------------------------------------------------------------------------------------------------------|

   |-----------------------------------------------------------------------------------------------------------------------------------------------------------|
   |---------------------------------------------------------------------UEs' Information----------------------------------------------------------------------|
   |  Index |     5GMM State     |        IMSI        |        GUTI        |   RAN UE NGAP ID   |   AMF UE NGAP ID   |        PLMN        |       Cell Id      |
   |    -   |          -         |          -         |          -         |          -         |          -         |          -         |          -         |
   |-----------------------------------------------------------------------------------------------------------------------------------------------------------|

[2025-08-28 11:44:13.436] [amf_sbi] [info] Receive Update NF Instance Request, handling ...
[2025-08-28 11:44:13.436] [amf_sbi] [info] Send HTTP message to http://oai-nrf:8080/nnrf-nfm/v1/nf-instances/b9f08296-e89f-4a03-b494-162dc784895d
[2025-08-28 11:44:13.436] [amf_sbi] [info] HTTP message Body: [{"op":"replace","path":"/nfStatus","value":"REGISTERED"}]
[2025-08-28 11:44:13.439] [amf_sbi] [info] Get response with HTTP code (204)
[2025-08-28 11:44:13.439] [amf_sbi] [info] Could not get JSON content from the response
[2025-08-28 11:44:23.439] [amf_sbi] [info] Receive Update NF Instance Request, handling ...
[2025-08-28 11:44:23.439] [amf_sbi] [info] Send HTTP message to http://oai-nrf:8080/nnrf-nfm/v1/nf-instances/b9f08296-e89f-4a03-b494-162dc784895d
[2025-08-28 11:44:23.439] [amf_sbi] [info] HTTP message Body: [{"op":"replace","path":"/nfStatus","value":"REGISTERED"}]
[2025-08-28 11:44:23.442] [amf_sbi] [info] Get response with HTTP code (204)
[2025-08-28 11:44:23.442] [amf_sbi] [info] Could not get JSON content from the response
[2025-08-28 11:44:33.288] [amf_app] [info] 
   |------------------------------------------------------------------------------------------------------------------------------------------------------------|
   |----------------------------------------------------------------------gNBs' Information---------------------------------------------------------------------|
   |  Index |               Status               |              Global Id             |              gNB Name              |                PLMN                |
   |    -   |                  -                 |                  -                 |                  -                 |                  -                 |
   |------------------------------------------------------------------------------------------------------------------------------------------------------------|

   |-----------------------------------------------------------------------------------------------------------------------------------------------------------|
   |---------------------------------------------------------------------UEs' Information----------------------------------------------------------------------|
   |  Index |     5GMM State     |        IMSI        |        GUTI        |   RAN UE NGAP ID   |   AMF UE NGAP ID   |        PLMN        |       Cell Id      |
   |    -   |          -         |          -         |          -         |          -         |          -         |          -         |          -         |
   |-----------------------------------------------------------------------------------------------------------------------------------------------------------|

[2025-08-28 11:44:33.442] [amf_sbi] [info] Receive Update NF Instance Request, handling ...
[2025-08-28 11:44:33.442] [amf_sbi] [info] Send HTTP message to http://oai-nrf:8080/nnrf-nfm/v1/nf-instances/b9f08296-e89f-4a03-b494-162dc784895d
[2025-08-28 11:44:33.442] [amf_sbi] [info] HTTP message Body: [{"op":"replace","path":"/nfStatus","value":"REGISTERED"}]
[2025-08-28 11:44:33.445] [amf_sbi] [info] Get response with HTTP code (204)
[2025-08-28 11:44:33.445] [amf_sbi] [info] Could not get JSON content from the response
[2025-08-28 11:44:43.445] [amf_sbi] [info] Receive Update NF Instance Request, handling ...
[2025-08-28 11:44:43.445] [amf_sbi] [info] Send HTTP message to http://oai-nrf:8080/nnrf-nfm/v1/nf-instances/b9f08296-e89f-4a03-b494-162dc784895d
[2025-08-28 11:44:43.445] [amf_sbi] [info] HTTP message Body: [{"op":"replace","path":"/nfStatus","value":"REGISTERED"}]
[2025-08-28 11:44:43.449] [amf_sbi] [info] Get response with HTTP code (204)
[2025-08-28 11:44:43.449] [amf_sbi] [info] Could not get JSON content from the response
[2025-08-28 11:44:53.288] [amf_app] [info] 
   |------------------------------------------------------------------------------------------------------------------------------------------------------------|
   |----------------------------------------------------------------------gNBs' Information---------------------------------------------------------------------|
   |  Index |               Status               |              Global Id             |              gNB Name              |                PLMN                |
   |    -   |                  -                 |                  -                 |                  -                 |                  -                 |
   |------------------------------------------------------------------------------------------------------------------------------------------------------------|

   |-----------------------------------------------------------------------------------------------------------------------------------------------------------|
   |---------------------------------------------------------------------UEs' Information----------------------------------------------------------------------|
   |  Index |     5GMM State     |        IMSI        |        GUTI        |   RAN UE NGAP ID   |   AMF UE NGAP ID   |        PLMN        |       Cell Id      |
   |    -   |          -         |          -         |          -         |          -         |          -         |          -         |          -         |
   |-----------------------------------------------------------------------------------------------------------------------------------------------------------|

[2025-08-28 11:44:53.450] [amf_sbi] [info] Receive Update NF Instance Request, handling ...
[2025-08-28 11:44:53.450] [amf_sbi] [info] Send HTTP message to http://oai-nrf:8080/nnrf-nfm/v1/nf-instances/b9f08296-e89f-4a03-b494-162dc784895d
[2025-08-28 11:44:53.450] [amf_sbi] [info] HTTP message Body: [{"op":"replace","path":"/nfStatus","value":"REGISTERED"}]
[2025-08-28 11:44:53.452] [amf_sbi] [info] Get response with HTTP code (204)
[2025-08-28 11:44:53.452] [amf_sbi] [info] Could not get JSON content from the response
[2025-08-28 11:44:54.108] [sctp] [info] ----------------------
[2025-08-28 11:44:54.108] [sctp] [info] Local addresses: 
[2025-08-28 11:44:54.110] [sctp] [info]     - IPv4 Addr: 192.168.70.135
[2025-08-28 11:44:54.110] [sctp] [info] ----------------------
[2025-08-28 11:44:54.110] [sctp] [info] Peer addresses: 
[2025-08-28 11:44:54.110] [sctp] [info]     - IPv4 Addr: 172.17.0.93
[2025-08-28 11:44:54.110] [sctp] [info] ----------------------
[2025-08-28 11:44:54.110] [sctp] [info] [Assoc_id 12, Socket 8] Received a message (length 67) from port 38147, on stream 0, PPID 60
[2025-08-28 11:44:54.113] [amf_n2] [info] Received NGSetupRequest message, handling
[2025-08-28 11:45:03.453] [amf_sbi] [info] Receive Update NF Instance Request, handling ...
[2025-08-28 11:45:03.454] [amf_sbi] [info] Send HTTP message to http://oai-nrf:8080/nnrf-nfm/v1/nf-instances/b9f08296-e89f-4a03-b494-162dc784895d
[2025-08-28 11:45:03.454] [amf_sbi] [info] HTTP message Body: [{"op":"replace","path":"/nfStatus","value":"REGISTERED"}]
[2025-08-28 11:45:03.458] [amf_sbi] [info] Get response with HTTP code (204)
[2025-08-28 11:45:03.458] [amf_sbi] [info] Could not get JSON content from the response
[2025-08-28 11:45:03.587] [sctp] [info] [Assoc_id 12, Socket 8] Received a message (length 76) from port 38147, on stream 1, PPID 60
[2025-08-28 11:45:03.588] [amf_n2] [info] Received Initial UE Message, handling
[2025-08-28 11:45:03.588] [amf_n2] [warning] No UE NGAP context with ran_ue_ngap_id 1, gnb_id 57344
[2025-08-28 11:45:03.588] [amf_app] [warning] No existing UE context associated with key app_ue_ranid_1:amfid_1
[2025-08-28 11:45:03.588] [amf_n1] [info] Received UL_NAS_DATA_IND
[2025-08-28 11:45:03.588] [amf_n1] [warning] No NAS context with amf_ue_ngap_id 1
[2025-08-28 11:45:03.588] [amf_n1] [warning] No existing nas_context with amf_ue_ngap_id 1
[2025-08-28 11:45:03.589] [amf_n1] [info] Associating SUPI (imsi-999700000000001) with NAS context
[2025-08-28 11:45:03.589] [amf_n1] [warning] No Optional IE 5GMMCapability available
[2025-08-28 11:45:03.589] [amf_sbi] [info] Receive UE Authentication Request message, handling ...
[2025-08-28 11:45:03.589] [amf_sbi] [info] Send HTTP message to http://oai-ausf:8080/nausf-auth/v1/ue-authentications
[2025-08-28 11:45:03.589] [amf_sbi] [info] HTTP message Body: {"servingNetworkName":"5G:mnc070.mcc999.3gppnetwork.org","supiOrSuci":"999700000000001"}
[2025-08-28 11:45:03.622] [amf_sbi] [info] Get response with HTTP code (201)
[2025-08-28 11:45:03.622] [amf_n1] [info] Links is: http://192.168.70.138:8080/nausf-auth/v1/ue-authentications/f9a4483463ad8000636ac89e492ee286/5g-aka-confirmation
[2025-08-28 11:45:03.623] [amf_n2] [info] Received Downlink NAS Transport message, handling
[2025-08-28 11:45:03.681] [sctp] [info] [Assoc_id 12, Socket 8] Received a message (length 64) from port 38147, on stream 1, PPID 60
[2025-08-28 11:45:03.681] [amf_n2] [info] Received Uplink NAS Transport message, handling
[2025-08-28 11:45:03.681] [amf_n1] [info] Received UL_NAS_DATA_IND
[2025-08-28 11:45:03.681] [amf_n1] [info] Found nas_context (0x73f1b8000e50) with amf_ue_ngap_id (1)
[2025-08-28 11:45:03.681] [amf_n1] [info] resStar_s (A39705A4841983F27F73683E7CF5C876)
[2025-08-28 11:45:03.682] [amf_sbi] [info] Receive UE Authentication Confirmation message, handling ...
[2025-08-28 11:45:03.682] [amf_sbi] [info] Send HTTP message to http://192.168.70.138:8080/nausf-auth/v1/ue-authentications/f9a4483463ad8000636ac89e492ee286/5g-aka-confirmation
[2025-08-28 11:45:03.682] [amf_sbi] [info] HTTP message Body: {"resStar":"A39705A4841983F27F73683E7CF5C876"}
[2025-08-28 11:45:03.703] [amf_sbi] [info] Get response with HTTP code (200)
[2025-08-28 11:45:03.704] [amf_n2] [info] Received Downlink NAS Transport message, handling
[2025-08-28 11:45:03.732] [sctp] [info] [Assoc_id 12, Socket 8] Received a message (length 100) from port 38147, on stream 1, PPID 60
[2025-08-28 11:45:03.732] [amf_n2] [info] Received Uplink NAS Transport message, handling
[2025-08-28 11:45:03.732] [amf_n1] [info] Received UL_NAS_DATA_IND
[2025-08-28 11:45:03.733] [amf_sbi] [info] Receive Slice Selection Subscription Data Retrieval Request, handling ...
[2025-08-28 11:45:03.733] [amf_sbi] [info] Send HTTP message to http://oai-udm:8080/nudm-sdm/v1/999700000000001/nssai?plmn-id={"mcc":"999","mnc":"70"}
[2025-08-28 11:45:03.733] [amf_sbi] [info] HTTP message Body: 
[2025-08-28 11:45:03.743] [amf_sbi] [info] Get response with HTTP code (500)
[2025-08-28 11:45:03.750] [amf_n1] [info] UE (IMSI 999700000000001, GUTI 9997001004100000001, current RAN ID 1, current AMF ID 1) has been registered to the network
[2025-08-28 11:45:03.750] [amf_n2] [info] Received Initial Context Setup Request message, handling
[2025-08-28 11:45:03.750] [amf_app] [info] 
   |------------------------------------------------------------------------------------------------------------------------------------------------------------|
   |----------------------------------------------------------------------gNBs' Information---------------------------------------------------------------------|
   |  Index |               Status               |              Global Id             |              gNB Name              |                PLMN                |
   |    1   |              Connected             |               0xE000               |           gNB-Eurecom-CU           |               999,70               |
   |------------------------------------------------------------------------------------------------------------------------------------------------------------|

   |-----------------------------------------------------------------------------------------------------------------------------------------------------------|
   |---------------------------------------------------------------------UEs' Information----------------------------------------------------------------------|
   |  Index |     5GMM State     |        IMSI        |        GUTI        |   RAN UE NGAP ID   |   AMF UE NGAP ID   |        PLMN        |       Cell Id      |
   |    1   |   5GMM-REGISTERED  |   999700000000001  | 9997001004100000001|        0x01        |        0x01        |       999,70       |      0xE00000      |
   |-----------------------------------------------------------------------------------------------------------------------------------------------------------|

[2025-08-28 11:45:03.821] [sctp] [info] [Assoc_id 12, Socket 8] Received a message (length 38) from port 38147, on stream 1, PPID 60
[2025-08-28 11:45:03.821] [amf_n2] [info] Received UE Radio Capability Indication message, handling
[2025-08-28 11:45:03.821] [sctp] [info] [Assoc_id 12, Socket 8] Received a message (length 19) from port 38147, on stream 1, PPID 60
[2025-08-28 11:45:03.845] [sctp] [info] [Assoc_id 12, Socket 8] Received a message (length 53) from port 38147, on stream 1, PPID 60
[2025-08-28 11:45:03.845] [sctp] [info] [Assoc_id 12, Socket 8] Received a message (length 80) from port 38147, on stream 1, PPID 60
[2025-08-28 11:45:03.845] [amf_n2] [info] Received Uplink NAS Transport message, handling
[2025-08-28 11:45:03.845] [amf_n1] [info] Received UL_NAS_DATA_IND
[2025-08-28 11:45:03.845] [amf_n2] [info] Received Uplink NAS Transport message, handling
[2025-08-28 11:45:03.845] [amf_n1] [info] Received UL_NAS_DATA_IND
[2025-08-28 11:45:03.846] [amf_sbi] [info] Running ITTI_SMF_PDU_SESSION_CREATE_SM_CTX
[2025-08-28 11:45:03.846] [amf_sbi] [info] Find ue_context in amf_app using UE Context Key: app_ue_ranid_1:amfid_1
[2025-08-28 11:45:03.846] [amf_app] [warning] No PDU Session Context with PDU Session ID 10
[2025-08-28 11:45:03.846] [amf_sbi] [info] Send HTTP message to http://oai-nrf:8080/nnrf-disc/v1/nf-instances?target-nf-type=SMF&requester-nf-type=AMF
[2025-08-28 11:45:03.846] [amf_sbi] [info] HTTP message Body: 
[2025-08-28 11:45:03.849] [amf_sbi] [info] Get response with HTTP code (200)
[2025-08-28 11:45:03.859] [amf_sbi] [info] JSON part {"smfServiceInstanceId":"801c8596-643c-4e9f-8547-1c27a5c1e3ec"}
[2025-08-28 11:45:03.859] [amf_sbi] [info] Location of the created SMF context: /nsmf-pdusession/v1/sm-contexts/1
[2025-08-28 11:45:03.867] [amf_server] [info] ue_context_id imsi-999700000000001
[2025-08-28 11:45:03.867] [amf_server] [info] Procedure n1-n2-messages
[2025-08-28 11:45:03.867] [amf_app] [info] Handle ITTI N1N2 Message Transfer Request
[2025-08-28 11:45:03.867] [amf_n1] [info] Received DOWNLINK_NAS_TRANSFER
[2025-08-28 11:45:03.868] [amf_n2] [info] Received PDU Session Resource Setup Request message, handling
[2025-08-28 11:45:03.901] [sctp] [info] [Assoc_id 12, Socket 8] Received a message (length 40) from port 38147, on stream 1, PPID 60
[2025-08-28 11:45:03.901] [amf_sbi] [info] Receive Nsmf_PDUSessionUpdateSMContext, handling ...
[2025-08-28 11:45:03.905] [amf_sbi] [info] JSON part {"cause":255}
telcomaan@oai-5gc-vm:~/oai-cn5g-fed/docker-compose$ 
```

#### smf logs
```
telcomaan@oai-5gc-vm:~/oai-cn5g-fed/docker-compose$ docker logs oai-smf
[2025-08-28 11:43:32.550] [smf_app] [start] Options parsed
[2025-08-28 11:43:32.551] [config ] [info] Reading NF configuration from /openair-smf/etc/config.yaml
[2025-08-28 11:43:32.694] [config ] [debug] Unknown NF upf in configuration. Ignored
[2025-08-28 11:43:32.694] [config ] [debug] Unknown NF udr in configuration. Ignored
[2025-08-28 11:43:32.694] [config ] [debug] Unknown NF ausf in configuration. Ignored
[2025-08-28 11:43:32.718] [config ] [debug] Validating configuration of log_level
[2025-08-28 11:43:32.781] [config ] [info] ==== OPENAIRINTERFACE smf vBranch: HEAD Abrev. Hash: 531b6d87 Date: Tue Jul 30 07:16:24 2024 +0000 ====
[2025-08-28 11:43:32.781] [config ] [info] Basic Configuration:
[2025-08-28 11:43:32.782] [config ] [info]   - log_level..................................: info
[2025-08-28 11:43:32.782] [config ] [info]   - register_nf................................: Yes
[2025-08-28 11:43:32.782] [config ] [info]   - http_version...............................: 2
[2025-08-28 11:43:32.782] [config ] [info]   - HTTP Request Timeout.......................: 3000 (ms)
[2025-08-28 11:43:32.782] [config ] [info] SMF Config:
[2025-08-28 11:43:32.782] [config ] [info]   - host.......................................: oai-smf
[2025-08-28 11:43:32.782] [config ] [info]   - sbi
[2025-08-28 11:43:32.782] [config ] [info]     + URL......................................: http://oai-smf:8080
[2025-08-28 11:43:32.782] [config ] [info]     + API Version..............................: v1
[2025-08-28 11:43:32.782] [config ] [info]     + IPv4 Address ............................: 192.168.70.133
[2025-08-28 11:43:32.782] [config ] [info]   - n4
[2025-08-28 11:43:32.782] [config ] [info]     + Port.....................................: 8805
[2025-08-28 11:43:32.782] [config ] [info]     + IPv4 Address ............................: 192.168.70.133
[2025-08-28 11:43:32.782] [config ] [info]     + MTU......................................: 1500
[2025-08-28 11:43:32.782] [config ] [info]     + Interface name: .........................: eth0
[2025-08-28 11:43:32.782] [config ] [info]   supported_features:
[2025-08-28 11:43:32.782] [config ] [info]     + use_local_subscription_info..............: Yes
[2025-08-28 11:43:32.782] [config ] [info]     + use_local_pcc_rules......................: Yes
[2025-08-28 11:43:32.782] [config ] [info]     + use_external_ausf........................: No
[2025-08-28 11:43:32.782] [config ] [info]     + use_external_udm.........................: No
[2025-08-28 11:43:32.782] [config ] [info]     + use_external_nssf........................: No
[2025-08-28 11:43:32.783] [config ] [info]   - ue_mtu.....................................: 1500
[2025-08-28 11:43:32.785] [config ] [info]   - p-cscf_ipv4................................: 127.0.0.1
[2025-08-28 11:43:32.785] [config ] [info]   - p-cscf_ipv6................................: fe80::7915:f408:1787:db8b
[2025-08-28 11:43:32.785] [config ] [info]   UPF List:
[2025-08-28 11:43:32.785] [config ] [info]     + oai-upf
[2025-08-28 11:43:32.785] [config ] [info]       + host...................................: oai-upf
[2025-08-28 11:43:32.785] [config ] [info]       + port...................................: 8805
[2025-08-28 11:43:32.785] [config ] [info]       + enable_usage_reporting.................: No
[2025-08-28 11:43:32.785] [config ] [info]       + enable_dl_pdr_in_session_establishment.: No
[2025-08-28 11:43:32.785] [config ] [info]   Local Subscription Infos:
[2025-08-28 11:43:32.786] [config ] [info]     - local_subscription_info
[2025-08-28 11:43:32.786] [config ] [info]       + dnn....................................: internet
[2025-08-28 11:43:32.786] [config ] [info]       + ssc_mode...............................: 1
[2025-08-28 11:43:32.786] [config ] [info]       + snssai:
[2025-08-28 11:43:32.786] [config ] [info]         - sst..................................: 1
[2025-08-28 11:43:32.786] [config ] [info]         - sd...................................: FFFFFF
[2025-08-28 11:43:32.786] [config ] [info]       + qos_profile:
[2025-08-28 11:43:32.786] [config ] [info]         - 5qi..................................: 9
[2025-08-28 11:43:32.786] [config ] [info]         - priority.............................: 1
[2025-08-28 11:43:32.788] [config ] [info]         - arp_priority.........................: 1
[2025-08-28 11:43:32.788] [config ] [info]         - arp_preempt_vulnerability............: NOT_PREEMPTABLE
[2025-08-28 11:43:32.788] [config ] [info]         - arp_preempt_capability...............: NOT_PREEMPT
[2025-08-28 11:43:32.788] [config ] [info]         - session_ambr_dl......................: 400Mbps
[2025-08-28 11:43:32.788] [config ] [info]         - session_ambr_ul......................: 200Mbps
[2025-08-28 11:43:32.788] [config ] [info]   + smf_info:
[2025-08-28 11:43:32.788] [config ] [info]     - snssai_smf_info_item:
[2025-08-28 11:43:32.788] [config ] [info]       + snssai:
[2025-08-28 11:43:32.788] [config ] [info]         - sst..................................: 1
[2025-08-28 11:43:32.788] [config ] [info]         - sd...................................: FFFFFF
[2025-08-28 11:43:32.788] [config ] [info]       + dnns:
[2025-08-28 11:43:32.788] [config ] [info]         - dnn..................................: internet
[2025-08-28 11:43:32.788] [config ] [info] Peer NF Configuration:
[2025-08-28 11:43:32.788] [config ] [info]   nrf:
[2025-08-28 11:43:32.788] [config ] [info]     - host.....................................: oai-nrf
[2025-08-28 11:43:32.788] [config ] [info]     - sbi
[2025-08-28 11:43:32.788] [config ] [info]       + URL....................................: http://oai-nrf:8080
[2025-08-28 11:43:32.788] [config ] [info]       + API Version............................: v1
[2025-08-28 11:43:32.788] [config ] [info] DNNs:
[2025-08-28 11:43:32.788] [config ] [info] - DNN:
[2025-08-28 11:43:32.788] [config ] [info]     + DNN......................................: internet
[2025-08-28 11:43:32.788] [config ] [info]     + PDU session type.........................: IPV4
[2025-08-28 11:43:32.788] [config ] [info]     + IPv4 subnet..............................: 12.1.2.0/24
[2025-08-28 11:43:32.788] [config ] [info]     + DNS Settings:
[2025-08-28 11:43:32.788] [config ] [info]       - primary_dns_ipv4.......................: 172.21.3.100
[2025-08-28 11:43:32.788] [config ] [info]       - primary_dns_ipv6.......................: 2001:4860:4860::8888
[2025-08-28 11:43:32.788] [config ] [info]       - secondary_dns_ipv4.....................: 8.8.8.8
[2025-08-28 11:43:32.788] [config ] [info]       - secondary_dns_ipv6.....................: 2001:4860:4860::8888
[2025-08-28 11:43:32.788] [itti   ] [start] Starting...
[2025-08-28 11:43:32.788] [itti   ] [start] Started
[2025-08-28 11:43:32.788] [smf_sbi] [info] HTTP Client successfully initiated on interface eth0 with timeout 3000 ms, HTTP version 2
[2025-08-28 11:43:32.788] [async  ] [start] Starting...
[2025-08-28 11:43:32.788] [itti   ] [info] Starting timer_manager_task
[2025-08-28 11:43:32.788] [itti   ] [warning] Could not set schedparam to ITTI task 0, err=1
[2025-08-28 11:43:32.792] [async  ] [warning] Could not set schedparam to ITTI task 1, err=1
[2025-08-28 11:43:32.799] [async  ] [start] Started
[2025-08-28 11:43:32.799] [smf_app] [start] Starting...
[2025-08-28 11:43:32.799] [smf_app] [info] Apply config...
[2025-08-28 11:43:32.801] [smf_app] [info] Applied config internet
[2025-08-28 11:43:32.801] [smf_app] [info] PAA Ipv4: 12.1.2.2
[2025-08-28 11:43:32.801] [smf_app] [info] Applied config
[2025-08-28 11:43:32.802] [pfcp   ] [info] pfcp_l4_stack created listening to 192.168.70.133:8805
[2025-08-28 11:43:32.803] [udp    ] [warning] Could not set schedparam to ITTI task 6, err=1
[2025-08-28 11:43:32.803] [smf_n4 ] [start] Starting...
[2025-08-28 11:43:32.803] [udp    ] [warning] Could not set schedparam to ITTI task 6, err=1
[2025-08-28 11:43:32.804] [smf_n4 ] [start] Started
[2025-08-28 11:43:32.804] [smf_sbi] [start] Starting...
[2025-08-28 11:43:32.805] [smf_sbi] [start] Started
[2025-08-28 11:43:32.805] [smf_app] [start] Started
[2025-08-28 11:43:32.818] [smf_api] [info] HTTP2 server being started
[2025-08-28 11:43:33.833] [smf_api] [info] NFStatusNotifyApiImpl, received a NF status notification...
[2025-08-28 11:43:33.833] [smf_app] [info] Handle a NF status notification from NRF (HTTP version 2)
[2025-08-28 11:43:33.855] [smf_n4 ] [info] handle_receive(53 bytes)
[2025-08-28 11:43:33.855] [smf_n4 ] [info] Received N4 ASSOCIATION SETUP RESPONSE from an UPF
[2025-08-28 11:43:33.855] [smf_n4 ] [info] Received N4 ASSOCIATION SETUP RESPONSE
[2025-08-28 11:43:33.856] [smf_app] [info] Successfully added UPF node: 192.168.70.136
[2025-08-28 11:43:38.854] [smf_n4 ] [info] TIME-OUT event timer id 2
[2025-08-28 11:43:43.856] [smf_n4 ] [info] TIME-OUT event timer id 3
[2025-08-28 11:43:43.856] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2286332096 starting
[2025-08-28 11:43:43.859] [smf_n4 ] [info] handle_receive(16 bytes)
[2025-08-28 11:43:45.854] [smf_app] [info] TIME-OUT event timer id 4
[2025-08-28 11:43:48.857] [smf_n4 ] [info] TIME-OUT event timer id 7
[2025-08-28 11:43:53.859] [smf_n4 ] [info] TIME-OUT event timer id 8
[2025-08-28 11:43:53.859] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2286332096 starting
[2025-08-28 11:43:53.860] [smf_n4 ] [info] handle_receive(16 bytes)
[2025-08-28 11:43:55.854] [smf_app] [info] TIME-OUT event timer id 9
[2025-08-28 11:43:58.860] [smf_n4 ] [info] TIME-OUT event timer id 12
[2025-08-28 11:44:03.861] [smf_n4 ] [info] TIME-OUT event timer id 13
[2025-08-28 11:44:03.861] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2286332096 starting
[2025-08-28 11:44:03.861] [smf_n4 ] [info] handle_receive(16 bytes)
[2025-08-28 11:44:05.854] [smf_app] [info] TIME-OUT event timer id 14
[2025-08-28 11:44:08.861] [smf_n4 ] [info] TIME-OUT event timer id 17
[2025-08-28 11:44:13.862] [smf_n4 ] [info] TIME-OUT event timer id 18
[2025-08-28 11:44:13.862] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2286332096 starting
[2025-08-28 11:44:13.863] [smf_n4 ] [info] handle_receive(16 bytes)
[2025-08-28 11:44:15.855] [smf_app] [info] TIME-OUT event timer id 19
[2025-08-28 11:44:18.862] [smf_n4 ] [info] TIME-OUT event timer id 22
[2025-08-28 11:44:23.863] [smf_n4 ] [info] TIME-OUT event timer id 23
[2025-08-28 11:44:23.863] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2286332096 starting
[2025-08-28 11:44:23.864] [smf_n4 ] [info] handle_receive(16 bytes)
[2025-08-28 11:44:25.855] [smf_app] [info] TIME-OUT event timer id 24
[2025-08-28 11:44:28.864] [smf_n4 ] [info] TIME-OUT event timer id 27
[2025-08-28 11:44:33.865] [smf_n4 ] [info] TIME-OUT event timer id 28
[2025-08-28 11:44:33.865] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2286332096 starting
[2025-08-28 11:44:33.865] [smf_n4 ] [info] handle_receive(16 bytes)
[2025-08-28 11:44:35.855] [smf_app] [info] TIME-OUT event timer id 29
[2025-08-28 11:44:38.865] [smf_n4 ] [info] TIME-OUT event timer id 32
[2025-08-28 11:44:43.866] [smf_n4 ] [info] TIME-OUT event timer id 33
[2025-08-28 11:44:43.866] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2286332096 starting
[2025-08-28 11:44:43.867] [smf_n4 ] [info] handle_receive(16 bytes)
[2025-08-28 11:44:45.856] [smf_app] [info] TIME-OUT event timer id 34
[2025-08-28 11:44:48.867] [smf_n4 ] [info] TIME-OUT event timer id 37
[2025-08-28 11:44:53.867] [smf_n4 ] [info] TIME-OUT event timer id 38
[2025-08-28 11:44:53.867] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2286332096 starting
[2025-08-28 11:44:53.868] [smf_n4 ] [info] handle_receive(16 bytes)
[2025-08-28 11:44:55.856] [smf_app] [info] TIME-OUT event timer id 39
[2025-08-28 11:44:58.868] [smf_n4 ] [info] TIME-OUT event timer id 42
[2025-08-28 11:45:03.850] [smf_api] [info] Received a SM context create request from AMF.
[2025-08-28 11:45:03.850] [smf_api] [info] Handle PDU Session Create SM Context Request.
[2025-08-28 11:45:03.850] [smf_app] [warning] No SelMode available
[2025-08-28 11:45:03.851] [smf_app] [info] Handle a PDU Session Create SM Context Request from an AMF (HTTP version 2)
[2025-08-28 11:45:03.851] [smf_n1 ] [info] Decode NAS message from N1 SM Container.
[2025-08-28 11:45:03.854] [smf_app] [info] Handle a PDU Session Create SM Context Request message from AMF, SUPI 999700000000001, - snssai:
  + sst........................................: 1
  + sd.........................................: FFFFFF

[2025-08-28 11:45:03.855] [smf_app] [info] Inserted DNN Subscription, key: 4294967041 dnn internet 
 - snssai:
  + sst........................................: 1
  + sd.........................................: FFFFFF

[2025-08-28 11:45:03.855] [smf_app] [info] Handle a PDU Session Create SM Context Request message from AMF (HTTP version 2)
[2025-08-28 11:45:03.855] [smf_app] [info] Find PDU Session with ID 10
[2025-08-28 11:45:03.856] [smf_app] [info] Add PDU Session with Id 10
[2025-08-28 11:45:03.856] [smf_n7 ] [warning] Local PCC rules are not supported yet
[2025-08-28 11:45:03.856] [smf_n7 ] [info] PCF SM Policy Association Creation was not successful. Continue using default rules
[2025-08-28 11:45:03.856] [smf_app] [info] Find a DNN Subscription with key: 4294967041, map size 1 and 
 - snssai:
  + sst........................................: 1
  + sd.........................................: FFFFFF

[2025-08-28 11:45:03.856] [smf_app] [info] Find DNN configuration with DNN internet
[2025-08-28 11:45:03.856] [smf_app] [info] PAA, Ipv4 Address: 12.1.2.2
[2025-08-28 11:45:03.857] [smf_app] [info] Create a procedure to process this message.
[2025-08-28 11:45:03.857] [smf_app] [info] Perform a procedure - Create SM Context Request
[2025-08-28 11:45:03.857] [smf_app] [info] Get default QoS for a PDU Session, key 1
[2025-08-28 11:45:03.857] [smf_app] [info] Find a DNN Subscription with key: 4294967041, map size 1 and 
 - snssai:
  + sst........................................: 1
  + sd.........................................: FFFFFF

[2025-08-28 11:45:03.857] [smf_app] [info] Find DNN configuration with DNN internet
[2025-08-28 11:45:03.857] [smf_app] [info] Find PDU Session with ID 10
[2025-08-28 11:45:03.857] [smf_app] [info] Verifying if UPF edge serves network
[2025-08-28 11:45:03.857] [smf_app] [info] Successfully added UPF node: 192.168.70.136
[2025-08-28 11:45:03.857] [smf_app] [info] Verifying if UPF edge serves network
[2025-08-28 11:45:03.857] [smf_app] [info] UPF selection was successful.
[2025-08-28 11:45:03.858] [smf_app] [info] Sending ITTI message 37itti_n4_session_establishment_request to task TASK_SMF_N4
[2025-08-28 11:45:03.860] [smf_n4 ] [info] handle_receive(70 bytes)
[2025-08-28 11:45:03.861] [smf_n1 ] [info] Create N1 SM Container, PDU Session Establishment Accept
[2025-08-28 11:45:03.861] [smf_n1 ] [info] PDU_SESSION_ESTABLISHMENT_ACCEPT, encode starting...
[2025-08-28 11:45:03.861] [smf_app] [info] Find a DNN Subscription with key: 4294967041, map size 1 and 
 - snssai:
  + sst........................................: 1
  + sd.........................................: FFFFFF

[2025-08-28 11:45:03.861] [smf_app] [info] Find DNN configuration with DNN internet
[2025-08-28 11:45:03.861] [smf_app] [info] Get default QoS Flow Description (PDU session type 1)
[2025-08-28 11:45:03.861] [smf_n1 ] [info] Encode PDU Session Establishment Accept
[2025-08-28 11:45:03.861] [smf_n2 ] [info] Create N2 SM Information, PDU Session Resource Setup Request Transfer
[2025-08-28 11:45:03.861] [smf_n2 ] [info] QoS parameters: QFI 1, Priority level 1, ARP priority level 1
[2025-08-28 11:45:03.862] [smf_app] [info] Find a DNN Subscription with key: 4294967041, map size 1 and 
 - snssai:
  + sst........................................: 1
  + sd.........................................: FFFFFF

[2025-08-28 11:45:03.862] [smf_app] [info] Find DNN configuration with DNN internet
[2025-08-28 11:45:03.862] [smf_n2 ] [info] QoS parameters: QFI 1, ARP priority level 1, qos_flow.qos_profile.arp.preempt_cap NOT_PREEMPT, qos_flow.qos_profile.arp.preempt_vuln NOT_PREEMPTABLE
[2025-08-28 11:45:03.865] [smf_app] [info] Sending ITTI message N11_SESSION_CREATE_SM_CONTEXT_RESPONSE to task TASK_SMF_APP
[2025-08-28 11:45:03.868] [smf_app] [info] Process N1N2MessageTransfer Response
[2025-08-28 11:45:03.868] [smf_app] [info] PDU_SESSION_ESTABLISHMENT_UE_REQUESTED
[2025-08-28 11:45:03.868] [smf_app] [info] Update PDU Session Status
[2025-08-28 11:45:03.868] [smf_app] [info] Find PDU Session with ID 10
[2025-08-28 11:45:03.868] [smf_app] [info] Set PDU Session Status to PDU_SESSION_ESTABLISHMENT_PENDING
[2025-08-28 11:45:03.868] [smf_app] [info] Set PDU Session Status to PDU_SESSION_ESTABLISHMENT_PENDING
[2025-08-28 11:45:03.868] [smf_app] [info] Update UpCnx_State
[2025-08-28 11:45:03.868] [smf_app] [info] Find PDU Session with ID 10
[2025-08-28 11:45:03.868] [smf_app] [info] Set upCnxState to UPCNX_STATE_DEACTIVATED
[2025-08-28 11:45:03.868] [smf_app] [info] Set PDU Session UpCnxState to UPCNX_STATE_DEACTIVATED
[2025-08-28 11:45:03.868] [smf_n4 ] [info] TIME-OUT event timer id 43
[2025-08-28 11:45:03.868] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2286332096 starting
[2025-08-28 11:45:03.869] [smf_n4 ] [info] handle_receive(16 bytes)
[2025-08-28 11:45:03.902] [smf_api] [info] Received a SM context update request from AMF.
[2025-08-28 11:45:03.902] [smf_api] [info] smf_ref 1, method modify
[2025-08-28 11:45:03.902] [smf_api] [info] Handle Update SM Context Request from AMF
[2025-08-28 11:45:03.903] [smf_api] [info] Handle PDU Session Update SM Context Request.
[2025-08-28 11:45:03.903] [smf_api] [info] Received a PDUSession_UpdateSMContext Request from AMF.
[2025-08-28 11:45:03.903] [smf_app] [info] Handle a PDU Session Update SM Context Request from an AMF (HTTP version 2)
[2025-08-28 11:45:03.903] [smf_app] [info] Find PDU Session with ID 10
[2025-08-28 11:45:03.903] [smf_app] [info] Handle a PDU Session Update SM Context Request message from an AMF (HTTP version 2)
[2025-08-28 11:45:03.903] [smf_app] [info] Find PDU Session with ID 10
[2025-08-28 11:45:03.903] [smf_app] [info] PDU Session Resource Setup Response Transfer
[2025-08-28 11:45:03.903] [smf_n2 ] [info] Decode NGAP message (PDUSessionResourceSetupResponseTransfer) from N2 SM Information
[2025-08-28 11:45:03.903] [smf_app] [info] PDU Session Establishment Request, processing N2 SM Information
[2025-08-28 11:45:03.903] [smf_app] [info] Perform a procedure - Update SM Context Request
[2025-08-28 11:45:03.903] [smf_app] [info] Find PDU Session with ID 10
[2025-08-28 11:45:03.903] [smf_app] [info] Sending ITTI message 36itti_n4_session_modification_request to task TASK_SMF_N4
[2025-08-28 11:45:03.904] [smf_n4 ] [info] handle_receive(31 bytes)
[2025-08-28 11:45:03.904] [smf_app] [info] Handle N4 Session Modification Response (PDU Session Id 10)
[2025-08-28 11:45:03.904] [smf_app] [info] PDU Session Establishment Request (UE-Initiated)
[2025-08-28 11:45:03.904] [smf_app] [info] Set PDU Session Status to PDU_SESSION_ACTIVE
[2025-08-28 11:45:03.904] [smf_app] [info] Set upCnxState to UPCNX_STATE_ACTIVATED
[2025-08-28 11:45:03.904] [smf_app] [info] SMF context: 
 
SMF CONTEXT:
SUPI:                           999700000000001
PDU SESSION:
        PDU Session ID:                 10
        DNN:                    internet
        S-NSSAI:                        SST=1, SD=FFFFFF
        PDN type:               IPV4
        PAA IPv4:               12.1.2.2
        Default QFI:            No QFI available
        SEID:                   1
        N3:
- UPF Graph Edge
  + Interface Type.............................: N3
  + NWI........................................: 
  + Uplink.....................................: No
  + PDR ID.....................................: 1
  + FAR ID.....................................: 2


[2025-08-28 11:45:03.904] [smf_app] [info] Find PDU Session with ID 10
[2025-08-28 11:45:03.904] [smf_app] [info] Find PDU Session with ID 10
[2025-08-28 11:45:03.904] [smf_app] [info] Find PDU Session with ID 10
[2025-08-28 11:45:03.904] [smf_app] [info] Find PDU Session with ID 10
[2025-08-28 11:45:03.904] [smf_app] [info] Find PDU Session with ID 10
[2025-08-28 11:45:03.904] [smf_app] [info] Find PDU Session with ID 10
[2025-08-28 11:45:03.904] [smf_app] [info] Find PDU Session with ID 10
[2025-08-28 11:45:03.904] [smf_app] [info] Find PDU Session with ID 10
[2025-08-28 11:45:03.904] [smf_app] [info] Sending ITTI message N11_SESSION_UPDATE_SM_CONTEXT_RESPONSE to task TASK_SMF_APP
[2025-08-28 11:45:03.904] [smf_app] [info] Handle N4 Session Modification Response
[2025-08-28 11:45:05.856] [smf_app] [info] TIME-OUT event timer id 44
[2025-08-28 11:45:08.858] [smf_n4 ] [info] TIME-OUT event timer id 46
[2025-08-28 11:45:08.869] [smf_n4 ] [info] TIME-OUT event timer id 49
[2025-08-28 11:45:08.903] [smf_n4 ] [info] TIME-OUT event timer id 52
[2025-08-28 11:45:13.869] [smf_n4 ] [info] TIME-OUT event timer id 50
[2025-08-28 11:45:13.869] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2286332096 starting
[2025-08-28 11:45:13.870] [smf_n4 ] [info] handle_receive(16 bytes)
[2025-08-28 11:45:15.856] [smf_app] [info] TIME-OUT event timer id 53
[2025-08-28 11:45:18.870] [smf_n4 ] [info] TIME-OUT event timer id 56
[2025-08-28 11:45:23.870] [smf_n4 ] [info] TIME-OUT event timer id 57
[2025-08-28 11:45:23.871] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2286332096 starting
[2025-08-28 11:45:23.871] [smf_n4 ] [info] handle_receive(16 bytes)
[2025-08-28 11:45:25.857] [smf_app] [info] TIME-OUT event timer id 58
[2025-08-28 11:45:28.871] [smf_n4 ] [info] TIME-OUT event timer id 61
[2025-08-28 11:45:33.871] [smf_n4 ] [info] TIME-OUT event timer id 62
[2025-08-28 11:45:33.871] [smf_n4 ] [info] PFCP HEARTBEAT PROCEDURE hash 2286332096 starting
[2025-08-28 11:45:33.872] [smf_n4 ] [info] handle_receive(16 bytes)
[2025-08-28 11:45:35.857] [smf_app] [info] TIME-OUT event timer id 63
[2025-08-28 11:45:38.872] [smf_n4 ] [info] TIME-OUT event timer id 66
telcomaan@oai-5gc-vm:~/oai-cn5g-fed/docker-compose$ 
```

#### upf-logs
```
telcomaan@oai-5gc-vm:~/oai-cn5g-fed/docker-compose$ docker logs oai-upf
[2025-08-28 11:43:32.890] [upf_app] [start] Options parsed
[2025-08-28 11:43:32.892] [upf_app] [debug] Parsing the configuration file, file type YAML.
[2025-08-28 11:43:32.892] [config ] [info] Reading NF configuration from /openair-upf/etc/config.yaml
[2025-08-28 11:43:32.995] [config ] [debug] Unknown NF amf in configuration. Ignored
[2025-08-28 11:43:32.995] [config ] [debug] Unknown NF udm in configuration. Ignored
[2025-08-28 11:43:32.995] [config ] [debug] Unknown NF udr in configuration. Ignored
[2025-08-28 11:43:32.995] [config ] [debug] Unknown NF ausf in configuration. Ignored
[2025-08-28 11:43:32.999] [config ] [debug] Validating configuration of log_level
[2025-08-28 11:43:33.032] [config ] [info] ==== OPENAIRINTERFACE upf vBranch: HEAD Abrev. Hash: 89ee4c9 Date: Fri Aug 30 11:25:24 2024 +0000 ====
[2025-08-28 11:43:33.032] [config ] [info] Basic Configuration:
[2025-08-28 11:43:33.032] [config ] [info]   - log_level..................................: info
[2025-08-28 11:43:33.032] [config ] [info]   - register_nf................................: Yes
[2025-08-28 11:43:33.032] [config ] [info]   - http_version...............................: 2
[2025-08-28 11:43:33.032] [config ] [info]   - HTTP Request Timeout.......................: 3000 (ms)
[2025-08-28 11:43:33.032] [config ] [info] UPF Configuration:
[2025-08-28 11:43:33.032] [config ] [info]   - host.......................................: oai-upf
[2025-08-28 11:43:33.032] [config ] [info]   - SBI
[2025-08-28 11:43:33.032] [config ] [info]     + URL......................................: http://oai-upf:8080
[2025-08-28 11:43:33.032] [config ] [info]     + API Version..............................: v1
[2025-08-28 11:43:33.032] [config ] [info]     + IPv4 Address ............................: 192.168.70.136
[2025-08-28 11:43:33.032] [config ] [info]   - N3:
[2025-08-28 11:43:33.032] [config ] [info]     + Port.....................................: 2152
[2025-08-28 11:43:33.032] [config ] [info]     + IPv4 Address ............................: 192.168.70.136
[2025-08-28 11:43:33.032] [config ] [info]     + MTU......................................: 1500
[2025-08-28 11:43:33.032] [config ] [info]     + Interface name: .........................: eth0
[2025-08-28 11:43:33.032] [config ] [info]     + Network Instance.........................: access.oai.org
[2025-08-28 11:43:33.032] [config ] [info]   - N4:
[2025-08-28 11:43:33.032] [config ] [info]     + Port.....................................: 8805
[2025-08-28 11:43:33.032] [config ] [info]     + IPv4 Address ............................: 192.168.70.136
[2025-08-28 11:43:33.032] [config ] [info]     + MTU......................................: 1500
[2025-08-28 11:43:33.032] [config ] [info]     + Interface name: .........................: eth0
[2025-08-28 11:43:33.032] [config ] [info]   - N6:
[2025-08-28 11:43:33.032] [config ] [info]     + Port.....................................: 2152
[2025-08-28 11:43:33.032] [config ] [info]     + IPv4 Address ............................: 192.168.70.136
[2025-08-28 11:43:33.032] [config ] [info]     + MTU......................................: 1500
[2025-08-28 11:43:33.032] [config ] [info]     + Interface name: .........................: eth0
[2025-08-28 11:43:33.032] [config ] [info]     + Network Instance.........................: core.oai.org
[2025-08-28 11:43:33.032] [config ] [info]   - Instance ID................................: 0
[2025-08-28 11:43:33.032] [config ] [info]   - Remote N6 Gateway..........................: localhost
[2025-08-28 11:43:33.032] [config ] [info]   - Support Features:
[2025-08-28 11:43:33.032] [config ] [info]     + Enable BPF Datapath......................: No
[2025-08-28 11:43:33.032] [config ] [info]     + Enable QoS...............................: No
[2025-08-28 11:43:33.032] [config ] [info]     + Enable SNAT..............................: Yes
[2025-08-28 11:43:33.032] [config ] [info]   + upf_info:
[2025-08-28 11:43:33.032] [config ] [info]     - snssai_upf_info_item:
[2025-08-28 11:43:33.032] [config ] [info]       + snssai:
[2025-08-28 11:43:33.032] [config ] [info]         - sst..................................: 1
[2025-08-28 11:43:33.032] [config ] [info]         - sd...................................: FFFFFF
[2025-08-28 11:43:33.032] [config ] [info]       + dnns:
[2025-08-28 11:43:33.032] [config ] [info]         - dnn..................................: internet
[2025-08-28 11:43:33.032] [config ] [info] Peer NF Configuration:
[2025-08-28 11:43:33.032] [config ] [info]   NRF:
[2025-08-28 11:43:33.032] [config ] [info]     - host.....................................: oai-nrf
[2025-08-28 11:43:33.032] [config ] [info]     - SBI
[2025-08-28 11:43:33.032] [config ] [info]       + URL....................................: http://oai-nrf:8080
[2025-08-28 11:43:33.032] [config ] [info]       + API Version............................: v1
[2025-08-28 11:43:33.032] [config ] [info]   SMF:
[2025-08-28 11:43:33.032] [config ] [info]     - host.....................................: oai-smf
[2025-08-28 11:43:33.032] [config ] [info]     - SBI
[2025-08-28 11:43:33.032] [config ] [info]       + URL....................................: http://oai-smf:8080
[2025-08-28 11:43:33.032] [config ] [info]       + API Version............................: v1
[2025-08-28 11:43:33.032] [config ] [info] DNNs:
[2025-08-28 11:43:33.032] [config ] [info] - DNN:
[2025-08-28 11:43:33.032] [config ] [info]     + DNN......................................: internet
[2025-08-28 11:43:33.032] [config ] [info]     + PDU session type.........................: IPV4
[2025-08-28 11:43:33.032] [config ] [info]     + IPv4 subnet..............................: 12.1.2.0/24
[2025-08-28 11:43:33.032] [config ] [info]     + DNS Settings:
[2025-08-28 11:43:33.032] [config ] [info]       - primary_dns_ipv4.......................: 8.8.8.8
[2025-08-28 11:43:33.032] [config ] [info]       - secondary_dns_ipv4.....................: 1.1.1.1
[2025-08-28 11:43:33.032] [upf_app] [info] HTTP Client successfully initiated on interface eth0 with timeout 3000 ms, HTTP version 2
[2025-08-28 11:43:33.032] [itti   ] [start] Starting...
[2025-08-28 11:43:33.032] [itti   ] [start] Started
[2025-08-28 11:43:33.032] [asc_cmd] [start] Starting...
[2025-08-28 11:43:33.033] [itti   ] [info] Starting timer_manager_task
[2025-08-28 11:43:33.034] [asc_cmd] [start] Started
[2025-08-28 11:43:33.034] [upf_app] [start] Starting...
[2025-08-28 11:43:33.040] [pfcp   ] [info] pfcp_l4_stack created listening to 192.168.70.136:8805
[2025-08-28 11:43:33.042] [upf_n4 ] [start] Starting...
[2025-08-28 11:43:33.043] [upf_n4 ] [start] Started
[2025-08-28 11:43:33.044] [gtpv1_u] [info] gtpu_l4_stack created listening to 192.168.70.136:2152
[2025-08-28 11:43:33.045] [upf_n3 ] [start] Starting...
[2025-08-28 11:43:33.047] [upf_n3 ] [start] Started
[2025-08-28 11:43:33.137] [upf_app] [start] Starting...
[2025-08-28 11:43:33.138] [upf_app] [info] Send NF Instance Registration to NRF
[2025-08-28 11:43:33.154] [upf_app] [info] Response from NRF, JSON data: 
 {"capacity":100,"heartBeatTimer":10,"ipv4Addresses":["192.168.70.136"],"json_data":null,"nfInstanceId":"1de790eb-0b06-4098-bda9-6460a793fa72","nfInstanceName":"OAI-UPF","nfServices":[],"nfStatus":"REGISTERED","nfType":"UPF","priority":1,"sNssais":[{"sd":"FFFFFF","sst":1}],"upfInfo":{"sNssaiUpfInfoList":[{"dnnUpfInfoList":[{"dnn":"internet"}],"sNssai":{"sd":"FFFFFF","sst":1}}]}}
[2025-08-28 11:43:33.154] [upf_app] [start] Started
[2025-08-28 11:43:33.154] [upf_app] [start] Started
[2025-08-28 11:43:33.854] [upf_n4 ] [info] handle_receive(34 bytes)
[2025-08-28 11:43:33.854] [upf_n4 ] [info] Handle SX ASSOCIATION SETUP REQUEST
[2025-08-28 11:43:43.155] [upf_app] [info] TIME-OUT event timer id 1
[2025-08-28 11:43:43.155] [upf_app] [info] Send NF Update to NRF
[2025-08-28 11:43:43.211] [upf_app] [info] Got successful response from NRF
[2025-08-28 11:43:43.857] [upf_n4 ] [info] handle_receive(16 bytes)
[2025-08-28 11:43:43.858] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST
[2025-08-28 11:43:53.232] [upf_app] [info] TIME-OUT event timer id 3
[2025-08-28 11:43:53.232] [upf_app] [info] Send NF Update to NRF
[2025-08-28 11:43:53.239] [upf_app] [info] Got successful response from NRF
[2025-08-28 11:43:53.860] [upf_n4 ] [info] handle_receive(16 bytes)
[2025-08-28 11:43:53.860] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST
[2025-08-28 11:44:03.240] [upf_app] [info] TIME-OUT event timer id 5
[2025-08-28 11:44:03.240] [upf_app] [info] Send NF Update to NRF
[2025-08-28 11:44:03.243] [upf_app] [info] Got successful response from NRF
[2025-08-28 11:44:03.861] [upf_n4 ] [info] handle_receive(16 bytes)
[2025-08-28 11:44:03.861] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST
[2025-08-28 11:44:13.243] [upf_app] [info] TIME-OUT event timer id 7
[2025-08-28 11:44:13.243] [upf_app] [info] Send NF Update to NRF
[2025-08-28 11:44:13.246] [upf_app] [info] Got successful response from NRF
[2025-08-28 11:44:13.862] [upf_n4 ] [info] handle_receive(16 bytes)
[2025-08-28 11:44:13.863] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST
[2025-08-28 11:44:23.246] [upf_app] [info] TIME-OUT event timer id 9
[2025-08-28 11:44:23.246] [upf_app] [info] Send NF Update to NRF
[2025-08-28 11:44:23.249] [upf_app] [info] Got successful response from NRF
[2025-08-28 11:44:23.864] [upf_n4 ] [info] handle_receive(16 bytes)
[2025-08-28 11:44:23.864] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST
[2025-08-28 11:44:33.249] [upf_app] [info] TIME-OUT event timer id 11
[2025-08-28 11:44:33.249] [upf_app] [info] Send NF Update to NRF
[2025-08-28 11:44:33.252] [upf_app] [info] Got successful response from NRF
[2025-08-28 11:44:33.865] [upf_n4 ] [info] handle_receive(16 bytes)
[2025-08-28 11:44:33.865] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST
[2025-08-28 11:44:43.252] [upf_app] [info] TIME-OUT event timer id 13
[2025-08-28 11:44:43.252] [upf_app] [info] Send NF Update to NRF
[2025-08-28 11:44:43.255] [upf_app] [info] Got successful response from NRF
[2025-08-28 11:44:43.867] [upf_n4 ] [info] handle_receive(16 bytes)
[2025-08-28 11:44:43.867] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST
[2025-08-28 11:44:53.255] [upf_app] [info] TIME-OUT event timer id 15
[2025-08-28 11:44:53.255] [upf_app] [info] Send NF Update to NRF
[2025-08-28 11:44:53.260] [upf_app] [info] Got successful response from NRF
[2025-08-28 11:44:53.868] [upf_n4 ] [info] handle_receive(16 bytes)
[2025-08-28 11:44:53.868] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST
[2025-08-28 11:45:03.260] [upf_app] [info] TIME-OUT event timer id 17
[2025-08-28 11:45:03.260] [upf_app] [info] Send NF Update to NRF
[2025-08-28 11:45:03.263] [upf_app] [info] Got successful response from NRF
[2025-08-28 11:45:03.858] [upf_n4 ] [info] handle_receive(174 bytes)
[2025-08-28 11:45:03.860] [upf_app] [info] Received N4_SESSION_ESTABLISHMENT_REQUEST seid 0x1 
[2025-08-28 11:45:03.860] [upf_n4 ] [info] pfcp_session::add(far) seid 0x1 
[2025-08-28 11:45:03.860] [upf_n4 ] [info] pfcp_session::add(pdr) seid 0x1 
[2025-08-28 11:45:03.869] [upf_n4 ] [info] handle_receive(16 bytes)
[2025-08-28 11:45:03.869] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST
[2025-08-28 11:45:03.903] [upf_n4 ] [info] handle_receive(100 bytes)
[2025-08-28 11:45:03.904] [upf_app] [info] Received N4_SESSION_MODIFICATION_REQUEST seid 0x1 
[2025-08-28 11:45:03.904] [upf_n4 ] [info] pfcp_session::add(far) seid 0x1 
[2025-08-28 11:45:03.904] [upf_n4 ] [info] pfcp_session::add(pdr) seid 0x1 
[2025-08-28 11:45:13.263] [upf_app] [info] TIME-OUT event timer id 19
[2025-08-28 11:45:13.263] [upf_app] [info] Send NF Update to NRF
[2025-08-28 11:45:13.266] [upf_app] [info] Got successful response from NRF
[2025-08-28 11:45:13.870] [upf_n4 ] [info] handle_receive(16 bytes)
[2025-08-28 11:45:13.870] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST
[2025-08-28 11:45:23.266] [upf_app] [info] TIME-OUT event timer id 23
[2025-08-28 11:45:23.266] [upf_app] [info] Send NF Update to NRF
[2025-08-28 11:45:23.269] [upf_app] [info] Got successful response from NRF
[2025-08-28 11:45:23.871] [upf_n4 ] [info] handle_receive(16 bytes)
[2025-08-28 11:45:23.871] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST
[2025-08-28 11:45:33.270] [upf_app] [info] TIME-OUT event timer id 25
[2025-08-28 11:45:33.270] [upf_app] [info] Send NF Update to NRF
[2025-08-28 11:45:33.273] [upf_app] [info] Got successful response from NRF
[2025-08-28 11:45:33.872] [upf_n4 ] [info] handle_receive(16 bytes)
[2025-08-28 11:45:33.872] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST
[2025-08-28 11:45:43.273] [upf_app] [info] TIME-OUT event timer id 27
[2025-08-28 11:45:43.273] [upf_app] [info] Send NF Update to NRF
[2025-08-28 11:45:43.276] [upf_app] [info] Got successful response from NRF
[2025-08-28 11:45:43.873] [upf_n4 ] [info] handle_receive(16 bytes)
[2025-08-28 11:45:43.873] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST
[2025-08-28 11:45:53.276] [upf_app] [info] TIME-OUT event timer id 29
[2025-08-28 11:45:53.276] [upf_app] [info] Send NF Update to NRF
[2025-08-28 11:45:53.279] [upf_app] [info] Got successful response from NRF
[2025-08-28 11:45:53.874] [upf_n4 ] [info] handle_receive(16 bytes)
[2025-08-28 11:45:53.874] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST
[2025-08-28 11:46:03.279] [upf_app] [info] TIME-OUT event timer id 31
[2025-08-28 11:46:03.279] [upf_app] [info] Send NF Update to NRF
[2025-08-28 11:46:03.282] [upf_app] [info] Got successful response from NRF
[2025-08-28 11:46:03.875] [upf_n4 ] [info] handle_receive(16 bytes)
[2025-08-28 11:46:03.875] [upf_n4 ] [info] Received SX HEARTBEAT REQUEST
telcomaan@oai-5gc-vm:~/oai-cn5g-fed/docker-compose$ 
```