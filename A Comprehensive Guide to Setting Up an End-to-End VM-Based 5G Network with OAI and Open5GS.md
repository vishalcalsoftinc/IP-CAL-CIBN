
---
This guide provides a structured, in-depth walkthrough for deploying a virtualized 5G network. By leveraging OpenAirInterface (OAI) for the Radio Access Network (RAN) and Open5GS for the 5G Core, you will learn to install, configure, and connect all necessary components in a virtual machine (VM) environment.

### **1. Introduction to the Key Technologies**

#### **What is OpenAirInterface (OAI)?**

OpenAirInterface (OAI) is an open-source software project that offers a complete implementation of 4G and 5G wireless communication systems, covering both the Radio Access Network (RAN) and the Core Network. Developed and maintained by the OpenAirInterface Software Alliance (OSA) at EURECOM, France, OAI is highly versatile and can be integrated with other open-source tools like Open5GS, UERANSIM, and srsRAN.

#### **What is Open5GS?**

Open5GS is an open-source implementation of the 5G Core Network (5GC) and 4G Evolved Packet Core (EPC). It provides a standards-compliant mobile core network suitable for testing, prototyping, and research. Open5GS includes all essential 5G core network functions, such as:
*   **AMF (Access and Mobility Management Function):** For managing access and mobility.
*   **SMF (Session Management Function):** For controlling PDU sessions.
*   **UPF (User Plane Function):** For routing user data.
*   **NRF (Network Repository Function):** For service discovery.
*   **NSSF (Network Slice Selection Function):** For selecting network slices.
*   **AUSF/UDM (Authentication Server Function/Unified Data Management):** For authentication.
*   **PCF (Policy Control Function):** For managing policies.

### **2. System Architecture**

This setup implements a 5G Standalone (SA) architecture with a split RAN (CU/DU separation) and a further split of the Central Unit into a Control Plane (CU-CP) and a User Plane (CU-UP). All components are virtualized and operate on separate VMs.

#### **Network Diagram**

The following diagram illustrates the architecture of the deployment:

<img alt="5G Architecture Diagram" src="https://i.imgur.com/your-diagram-image.png" width="800"/>

*Image based on the provided document.*

#### **Component Breakdown and VM-to-IP Mapping**

The setup consists of five key components distributed across five virtual machines:

1.  **VM-1: OAI NR-UE (User Equipment)** - `172.17.42.91`
    *   **Function:** Simulates a 5G device like a smartphone. It initiates network registration, performs authentication, and manages data traffic.
    *   **Interface:** Connects to the OAI DU via a simulated radio interface (Uu interface) using an RF simulator.

2.  **VM-2: OAI DU (Distributed Unit)** - `172.17.42.92`
    *   **Function:** Manages the lower layers of the RAN stack (PHY, MAC, RLC). It handles real-time radio signal processing.
    *   **Interfaces:**
        *   **Uu:** Connects to the NR-UE.
        *   **F1-C (SCTP):** Sends control messages to the CU-CP.
        *   **F1-U (GTP-U):** Forwards user data to the CU-UP.

3.  **VM-3: OAI CU-CP (Central Unit - Control Plane)** - `172.17.42.93`
    *   **Function:** Manages Radio Resource Control (RRC), UE context, mobility, and routes NAS messages between the RAN and the Core.
    *   **Interfaces:**
        *   **N2:** Connects to the AMF for signaling (e.g., registration).
        *   **E1:** Coordinates with the CU-UP.
        *   **F1-C:** Connects to the DU.

4.  **VM-4: OAI CU-UP (Central Unit - User Plane)** - `172.17.42.94`
    *   **Function:** Forwards user data traffic between the DU and the UPF in the 5G Core.
    *   **Interfaces:**
        *   **N3:** Connects to the UPF via a GTP-U tunnel.
        *   **F1-U:** Receives data from the DU.
        *   **E1:** Connects to the CU-CP.

5.  **VM-5: Open5GS Core (AMF+UPF+SMF, etc.)** - `172.17.42.95`
    *   **Function:** The brain of the network, handling authentication, session management, and data routing to the internet.
    *   **Interfaces:**
        *   **N2:** Connects to the CU-CP.
        *   **N3:** Connects to the CU-UP.
        *   **N6:** Connects the UPF to the external data network (internet).

---

### **3. Installation and Configuration**

#### **Step 1: Install and Configure Open5GS (VM-5)**

1.  **Install Open5GS:**
    ```bash
    # Install Open5GS as a daemon service
    sudo apt update
    sudo apt install software-properties-common
    sudo add-apt-repository ppa:open5gs/latest
    sudo apt update
    sudo apt install open5gs -y
    ```

2.  **Enable IP Forwarding and NAT:**
    To allow traffic from the 5G Core to reach the internet, you need to enable IP forwarding and set up a NAT rule.
    ```bash
    # 1. Enable IP Forwarding
    sudo sysctl -w net.ipv4.ip_forward=1

    # 2. Add a NAT rule (replace ens34 if your interface is different)
    sudo iptables -t nat -A POSTROUTING -s 10.45.0.0/16 -o enp0s3 -j MASQUERADE

    # 3. Stop the firewall to avoid blocking traffic during setup
    sudo systemctl stop ufw

    # 4. Allow all forwarded packets
    sudo iptables -I FORWARD 1 -j ACCEPT
    ```
 
```bash
	user@IN-8KBKXD3:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet 10.255.255.254/32 brd 10.255.255.254 scope global lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:a0:90:f8 brd ff:ff:ff:ff:ff:ff
    inet 172.28.251.212/20 brd 172.28.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet 172.17.42.95/20 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::215:5dff:fea0:90f8/64 scope link
       valid_lft forever preferred_lft forever
3: ogstun: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UP group default qlen 500
    link/none
    inet 10.45.0.1/16 brd 10.45.255.255 scope global ogstun
       valid_lft forever preferred_lft forever
    inet6 2001:db8:cafe::1/48 scope global
       valid_lft forever preferred_lft forever
    inet6 fe80::7d5c:5849:7f67:234/64 scope link stable-privacy
       valid_lft forever preferred_lft forever
user@IN-8KBKXD3:~$
```

3.  **Configure Open5GS Components:**

    *   **AMF Configuration (`/etc/open5gs/amf.yaml`):**
        Set the NGAP server IP to the Open5GS VM's IP (`172.17.42.95`).
        ```yaml
        amf:
          ngap:
            server:
              address: 172.17.42.95 # open5gs vm ip
          # ... other configurations
        ```

    *   **UPF Configuration (`/etc/open5gs/upf.yaml`):**
        Set the GTP-U server IP and define the subnet for UEs.
        ```yaml
        upf:
          gtpu:
            server:
              address: 172.17.42.95 # open5gs vm ip
          session:
            - subnet: 10.45.0.0/16
              dnn: internet
        ```

    *   **SMF and NSSF Configuration:**
        The `smf.yaml` and `nssf.yaml` files should be reviewed to ensure they align with the network slices and DNNs you plan to use. For this guide, the default configurations are mostly sufficient.

4.  **Add a Subscriber via Open5GS WebUI:**

    *   **Install WebUI dependencies and clone the repository:**
        ```bash
        # Install Node.js
        sudo apt update
        sudo apt install curl
        curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
        sudo apt install nodejs

        # Clone the webui
        git clone https://github.com/open5gs/open5gs.git
        ```

    *   **Configure and run the WebUI:**
        Modify `open5gs/webui/server/index.js` to listen on all interfaces.
        ```javascript
        const hostname = process.env.HOSTNAME || '0.0.0.0';
        ```
        Then, run the WebUI:
        ```bash
        cd open5gs/webui
        npm run dev --host 0.0.0.0
        ```

    *   **Register a new subscriber:**
        Access the WebUI at `http://<vm_ip>:9999` (login: `admin`, password: `1423`). Add a new subscriber with the following details:
        *   **IMSI:** `999700000000001`
        *   **Subscriber Key (K):** `465B5CE8B199B49FAA5F0A2EE238A6BC`
        *   **Operator Key (OPc):** `E8ED289DEBA952E4283B54E88E6183CA`

#### **Step 2: Install and Configure OAI RAN Components**

1.  **Install OAI (on VMs 2, 3, 4):**
    Clone the OAI repository and build the required components.
    ```bash
    git clone https://gitlab.eurecom.fr/oai/openairinterface5g oai
    cd oai/
    cd cmake_targets/
    
    ./build_oai -I
    # For gNB components (CU/DU)
    ./build_oai -w SIMU --gNB --ninja
    # For the UE component (on VM-1)
    ./build_oai -w SIMU --nrUE --ninja
    ./build_oai --build-e2 --ninja
    ```

2.  **OAI CU-CP Configuration (VM-3):**
    In your OAI CU-CP configuration file, set the local and remote IPs for the F1, E1, and N2 interfaces.
    *   `local_s_address`: "172.17.42.93" (CU-CP VM IP)
    *   `remote_s_address`: "172.17.42.92" (DU VM IP)
    *   `amf_ip_address`: { ipv4 = "172.17.42.95"; } (Open5GS VM IP)
    *   `E1_INTERFACE`: Define IPs for CU-CP ("172.17.42.93") and CU-UP ("172.17.42.94").

3.  **OAI CU-UP Configuration (VM-4):**
    Configure the CU-UP to communicate with the DU, CU-CP, and UPF.
    *   `local_s_address`: "172.17.42.94" (CU-UP VM IP)
    *   `remote_s_address`: "172.17.42.92" (DU VM IP)
    *   `E1_INTERFACE`: Define IPs for CU-CP ("172.17.42.93") and CU-UP ("172.17.42.94").
    *   `GNB_IPV4_ADDRESS_FOR_NGU`: "172.17.42.94" (for the N3 interface towards UPF).

4.  **OAI DU Configuration (VM-2):**
    Set the local DU IP and the remote CU-UP IP for the F1 interface.
    *   `local_n_address`: "172.17.42.92" (DU VM IP)
    *   `remote_n_address`: "172.17.42.94" (CU-UP VM IP)

5.  **OAI UE Configuration (VM-1):**
    In the UE configuration file, set the IMSI, authentication keys, and selected network slice to match the subscriber you created in Open5GS.
    *   `imsi`: "999700000000012" (Note: Use a unique IMSI per UE).
    *   `key`: "465B5CE8B199B49FAA5F0A2EE238A6BC"
    *   `opc`: "E8ED289DEBA952E4283B54E88E6183CA"
    *   `dnn`: "internet"
    *   `nssai_sst`: 1
    *   `nssai_sd`: 0xFFFFFF

---

### **4. Starting the Network Components**

It is crucial to start the components in the correct order to ensure proper registration and communication.

1.  **Start Open5GS Core:**
    ```bash
    sudo systemctl start open5gs-*
    ```

2.  **Start OAI RAN Components (as services for robustness):**

    *   **CU-CP (on VM-3):** Create a systemd service file `/etc/systemd/system/oai-cucp.service` and then start it.
    *   **CU-UP (on VM-4):** Similarly, create a systemd service file `/etc/systemd/system/oai-cuup.service` and start it.
    *   **DU (on VM-2):** Create a systemd service file `/etc/systemd/system/oai-du.service` and start it.

    **To run directly (example for CU-CP):**
    ```bash
    sudo /usr/lib/nr-softmodem -O /etc/oai/oai-cucp.conf
    ```

3.  **Start OAI NR-UE (on VM-1):**
    Run the UE softmodem with the appropriate parameters.
    ```bash
    sudo /usr/lib/nr-uesoftmodem -r 106 --numerology 1 --band 48 -C 3619200000 --rfsim --rfsimulator.serveraddr 172.17.42.92 -O /home/vm-1/nr-ue.conf
    ```

### **5. Testing and Verification**

#### **End-to-End Connectivity**

Once all components are running, a tunnel interface named `oaitun_ue1` should appear on the UE VM. Verify this with `ip a`. You can then test connectivity to the internet:
```bash
ping -I oaitun_ue1 8.8.8.8
```
A successful ping with 0% packet loss confirms that the end-to-end connection is working.

#### **Use Cases for Data Transfer**

To further validate the user-plane functionality, perform the following tests:

*   **Use Case 1: File Download**
    1.  On the UPF (VM-5), monitor traffic on the `ogstun` interface: `sudo tcpdump -i ogstun`
    2.  On the UE (VM-1), download a file, binding to the tunnel interface's IP:
        ```bash
        sudo wget --no-check-certificate --bind-address=10.45.0.2 https://raw.githubusercontent.com/open5gs/open5gs/main/README.md
        ```

*   **Use Case 2 & 3: Image and Video Download**
    Follow the same procedure as the file download, but use URLs for an image and a large video file to test different traffic patterns and loads.

### **6. Customizing Network Slices**

To use custom network slices beyond the default:

1.  **Define Slices:** Decide on the `sst` and `sd` values for your slices (e.g., `sst=2, sd=0x111111`).
2.  **Update Configuration Files:** Add the new slice values to the configuration files of `amf`, `smf`, `nssf`, `cu-cp`, `cu-up`, and `du`.
3.  **Update UE Configuration:** In the `ue.conf` file, change `nssai_sst` and `nssai_sd` to match the desired slice.
4.  **Restart Components:** Restart all network components for the changes to take effect.

### **7. Conclusion**

This guide has detailed the step-by-step process for building a complete 5G standalone test environment using OAI and Open5GS. By following these instructions, developers and researchers can create a reproducible and accessible testbed for experimenting with 5G technologies, developing new applications, and testing various network configurations. This open-source setup provides a powerful platform for innovation in the 5G landscape.



### Issues & Resolution

While I was running `npm run dev --host 0.0.0.0` in Open 5gs webui

I got this error:

`MongooseServerSelectionError: connect ECONNREFUSED 127.0.0.1:27017`

### 🔍 **Root Cause:**

- The app uses **MongoDB** (via Mongoose) at `mongodb://localhost:27017`
    
- But **MongoDB was not running** in your WSL instance
    
- And `systemctl` doesn’t work in WSL (no systemd by default)
    

---

**Quick Resolution Summary**

1. Removed old MongoDB versions:
    

```bash
sudo apt purge -y mongodb* mongo-tools
```

2. Installed MongoDB 5.0 from the official repo:
    

```bash
sudo apt install -y mongodb-org
```

3. Started MongoDB manually (since WSL lacks systemd):
    

```bash
mongod --dbpath /data/db --bind_ip 127.0.0.1 --fork --logpath ~/mongod.log
```

4. Verified MongoDB is running:
    

```bash
ps aux | grep mongod
```

5. Application now connects without `ECONNREFUSED`.