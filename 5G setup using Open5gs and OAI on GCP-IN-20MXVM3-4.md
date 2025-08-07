
---
## **Deploying a 5G Testbed on GCP: OAI and Open5GS End-to-End Guide**

This guide provides a complete walkthrough for deploying a virtualized 5G Standalone (SA) network on Google Cloud Platform (GCP) using OpenAirInterface (OAI) for the Radio Access Network (RAN) and Open5GS for the 5G Core (5GC).

### **Part 1: GCP Environment and Cost Management**

#### **1.1. Leveraging the GCP Free Trial**
Google Cloud offers a generous 90-day, $300 free trial for new users. This credit is more than sufficient to cover all costs for this deployment.

*   **Machine Types:** We will use `e2-small` (2 vCPU, 2 GB memory) VMs. While the `e2-micro` is part of the "Always Free" tier, its limited resources may cause performance issues with the RAN components. The `e2-small` instances offer a better balance and will be fully covered by your free trial credits.
*   **Cost Estimation:** With the $300 credit, the cost for this entire setup during the trial period will be **$0**.
*   **⚠️ Important: Cleanup!** To avoid any charges after the trial ends, you **must** shut down or delete your created resources (VMs, Cloud NAT) once you are finished experimenting.

#### **1.2. Initial Project Setup**
1.  Navigate to the [Google Cloud Console](https://console.cloud.google.com/).
2.  Create a new GCP project. Give it a memorable name like `5g-oai-open5gs-proj`.
3.  Ensure that billing is enabled for the project by linking your free trial billing account.

#### **1.3. VPC Network and Firewall Configuration**
We will create an isolated Virtual Private Cloud (VPC) for our 5G components.

1.  **Create a VPC:**
    *   Navigate to **VPC network > VPC networks**.
    *   Click **CREATE VPC NETWORK**.
    *   **Name:** `oai-5g-vpc`
    *   **Subnet creation mode:** Custom
    *   **New subnet:**
        *   Name: `oai-subnet`
        *   Region: `us-central1` (or another cost-effective US region)
        *   IP address range: `172.17.0.0/24`
    *   Click **Create**.

2.  **Configure Firewall Rules:**
    *   Navigate to **VPC network > Firewall**.
    *   **Rule 1: Allow SSH (for management)**
        *   Click **CREATE FIREWALL RULE**.
        *   **Name:** `allow-ssh`
        *   **Network:** `oai-5g-vpc`
        *   **Targets:** All instances in the network
        *   **Source filter:** IPv4 ranges
        *   **Source IPv4 ranges:** `0.0.0.0/0` (allows SSH from any IP)
        *   **Protocols and ports:** Specified protocols and ports > `tcp:22`
        *   Click **Create**.
    *   **Rule 2: Allow Internal Communication**
        *   Click **CREATE FIREWALL RULE**.
        *   **Name:** `allow-internal-all`
        *   **Network:** `oai-5g-vpc`
        *   **Targets:** All instances in the network
        *   **Source filter:** IPv4 ranges
        *   **Source IPv4 ranges:** `172.17.0.0/24` (the range of our subnet)
        *   **Protocols and ports:** Allow all
        *   Click **Create**.

3.  **Configure Cloud NAT for Internet Access:**
    To allow the UE to access the internet through the 5G core, we'll use a Cloud NAT instead of `iptables`.
    *   Navigate to **Network services > Cloud NAT**.
    *   Click **GET STARTED** or **CREATE NAT GATEWAY**.
    *   **Gateway name:** `oai-5g-nat`
    *   **VPC network:** `oai-5g-vpc`
    *   **Region:** `us-central1`
    *   **Cloud Router:** Click **Create new router**, name it `oai-5g-router`, and click **Create**.
    *   Leave other settings as default and click **Create**.

### **Part 2: Provisioning Virtual Machines**

We need to create five VMs and assign them static internal IP addresses that correspond to the roles in the original documentation.

1.  **Reserve Static Internal IPs:**
    *   Navigate to **VPC network > IP addresses**.
    *   Click **RESERVE INTERNAL STATIC IP ADDRESS** five times to reserve the following IPs:
        *   `172.17.0.91` (for `oai-nr-ue-vm`)
        *   `172.17.0.92` (for `oai-du-vm`)
        *   `172.17.0.93` (for `oai-cucp-vm`)
        *   `172.17.0.94` (for `oai-cuup-vm`)
        *   `172.17.0.95` (for `open5gs-vm`)  
	
2. **Create the Virtual Machines:**    
    - Navigate to **Compute Engine > VM instances** and click **CREATE INSTANCE** for each VM as specified below.
        
    - **VM Creation: Open5GS Core (open5gs-vm)**        
        - **Name:** open5gs-vm            
        - **Region:** us-central1            
        - **Series:** E2            
        - **Machine type:** e2-small (2 vCPU, 2 GB memory)            
        - **Boot disk:** Ubuntu, Version: **Ubuntu 24.04 LTS**, Size: 30 GB            
        - Expand **Advanced options**. Under **Networking**:            
            - Select the oai-5g-vpc network and oai-subnet.                
            - **Primary internal IP:** Select the reserved 172.17.0.95.                
            - **External IPv4 address:** Select Ephemeral to allow SSH access.                
            - Check **Enable IP forwarding**.
                
    - **VM Creation: All OAI RAN VMs**        
        - Create the four OAI VMs (oai-cucp-vm, oai-cuup-vm, oai-du-vm, oai-nr-ue-vm) with the following settings:            
        - **Name:** The respective VM name from the table below.            
        - **Region:** us-central1            
        - **Series:** **N2** (This is critical for AVX-512 support)            
        - **Machine type:** **n2-standard-4** (4 vCPU, 16 GB memory)            
        - **Boot disk:** Ubuntu, Version: **Ubuntu 24.04 LTS**, Size: 30 GB            
        - Expand **Advanced options**. Under **Networking**:            
            - Select the oai-5g-vpc network and oai-subnet.                
            - **Primary internal IP:** Select the corresponding reserved IP from the table.             
            - ==**External IPv4 address:** Select Ephemeral for UE and DU , Select none for CUCP and CUUP as we will access them using their internal-IP.==

![[Pasted image 20250731112932.png]]![[Pasted image 20250731113002.png]]![[Pasted image 20250731135750.png]]

 **VM-Specific Names and IPs:**
 
| Original Role   | GCP VM Name    | Reserved Static IP |
| :-------------- | :------------- | :----------------- |
| Open5GS (vm5)   | `open5gs-vm`   | `172.17.0.95`      |
| OAI CU-CP (vm3) | `oai-cucp-vm`  | `172.17.0.93`      |
| OAI CU-UP (vm4) | `oai-cuup-vm`  | `172.17.0.94`      |
| OAI DU (vm2)    | `oai-du-vm`    | `172.17.0.92`      |
| OAI NR-UE (vm1) | `oai-nr-ue-vm` | `172.17.0.91`      |

After creation, you will manage these VMs by using the **SSH** button in the GCP console, which opens a terminal in your browser.

---
### **Part 3: Setting up SSH for VS Code and Google Cloud**

This guide details the steps required to securely connect to your Google Cloud (GCP) Virtual Machines (VMs) using Visual Studio Code (VS Code) and an SSH key pair. This method is more secure than using passwords and provides a powerful, integrated development environment.

#### **3.1. Create SSH Key Pairs**

The first step is to generate a secure key pair on your local computer. This consists of a **public key** (which you share with GCP) and a **private key** (which you keep secret on your machine).

1.  **Open PowerShell or Terminal** on your local Windows, Mac, or Linux machine.

2.  Run the `ssh-keygen` command. We will specify the RSA algorithm and a key size of 4096 bits for strong security and username. This username will be created on the GCP VMs you connect to.
    ```bash
    ssh-keygen -t rsa -b 4096 -C "telcomaan"
    ```

3.  The command will prompt you for the following:
    *   **`Enter file in which to save the key...`**: It is highly recommended to **press Enter** to accept the default file location and name (`id_rsa`). This is the standard location that SSH tools look for keys.
    *   **`Enter passphrase (empty for no passphrase):`**: Type a strong, memorable password and press Enter. This is a crucial security step that encrypts your private key. Even if someone steals your private key file, they cannot use it without this passphrase.
    *   **`Enter same passphrase again:`**: Re-type the exact same password and press Enter.

4.  **Confirm Key Creation:** Once successful, the command will confirm that it has saved your "identification" (private key) and "public key" in your `.ssh` directory.

    *   **Private Key:** `~/.ssh/id_rsa`
    *   **Public Key:** `~/.ssh/id_rsa.pub`

#### **3.2. Add the Public Key on GCP**

Now, you must provide your public key to Google Cloud. By adding it as project-wide metadata, you grant access to all VMs within that project.

1.  **Display and Copy Your Public Key:** In your local terminal, run the following command to display the contents of your public key file.
    ```bash
    cat ~/.ssh/id_rsa.pub
    ```
2.  The output will be a long, single line of text starting with `ssh-rsa`. **Select this entire line of text and copy it** to your clipboard.

3.  **Navigate to GCP Metadata:**
    *   In the Google Cloud Console, use the navigation menu (☰) to go to **Compute Engine > Metadata**.

4.  **Add the SSH Key:**
    *   Select the **SSH Keys** tab.
    *   Click the **ADD SSH KEY** button.
    *   Paste your copied public key into the large text box that appears.
    *   GCP will automatically parse the username from the end of the key.
    *   Click **Save**.

Within a few minutes, Google Cloud will distribute this key to all VMs in the project, authorizing access for your private key.

#### **3.3. Install and Set Up Remote SSH in VS Code**

The final step is to configure VS Code to use your private key to establish a connection.

1.  **Copy the External IP:** Ensure the VM you want to connect to has a public IP address.
    *   In the GCP Console, go to **Compute Engine > VM instances**.
    *   Copy the External IP

2.  **Install the "Remote - SSH" Extension:**
    *   In VS Code, open the Extensions view (Ctrl+Shift+X).
    *   Search for `Remote - SSH`.
    *   Install the official extension provided by **Microsoft**.

3.  **Configure the SSH Connection File:**
    *   Open the Command Palette (`Ctrl+Shift+P`).
    *   Type `Remote-SSH: Open SSH Configuration File...` and select it.
    *   Choose the standard `config` file located in your user's `.ssh` directory.

4.  **Add Host Entry:** Add a new block of text to this file to define each VM connection. 
```
# Read more about SSH config files: https://linux.die.net/man/5/ssh_config
Host alias
    HostName hostname
    User user
  
# The Bastion Host - This one has a public IP
# Google Cloud VM for Open5GS Core
Host gcp-open5gs-vm    
    HostName 34.44.235.156
    User telcomaan
    IdentityFile C:\Users\vishalkumar.shaw\.ssh\id_rsa
  
# Google Cloud VM for OAI-NR-UE
Host gcp-oai-nr-ue-vm    
    HostName 34.27.116.250
    # HostName 172.17.0.91
    User telcomaan  
    # ProxyJump gcp-open5gs-vm
    IdentityFile C:\Users\vishalkumar.shaw\.ssh\id_rsa
  
# Google Cloud VM for OAI-DU
Host gcp-oai-du-vm    
    HostName 172.17.0.92
    User telcomaan  
    ProxyJump gcp-open5gs-vm  
    IdentityFile C:\Users\vishalkumar.shaw\.ssh\id_rsa
  
# Google Cloud VM for OAI-CUCP
Host gcp-oai-cucp-vm    
    HostName 172.17.0.93
    User telcomaan  
    ProxyJump gcp-open5gs-vm
    IdentityFile C:\Users\vishalkumar.shaw\.ssh\id_rsa
  
# Google Cloud VM for OAI-CUUP
Host gcp-oai-cuup-vm    
    HostName 172.17.0.94
    User telcomaan  
    ProxyJump gcp-open5gs-vm
    IdentityFile C:\Users\vishalkumar.shaw\.ssh\id_rsa
  
  

# the HostName is the IP address of the VM (its epemeral and changes everytime you start the VM)
# the User is the username you use to log in to the VM
# the IdentityFile is the path to your private SSH key file

```

5.  **Save the `config` file.** Repeat Step 4 for any other VMs you wish to connect to, giving each a unique `Host` name and its correct `HostName` (External IP).

6.  **Connect to the VM:**
    *   Open the Command Palette (`Ctrl+Shift+P`).
    *   Type `Remote-SSH: Connect to Host...` and select it.
    *   Choose the host you configured (e.g., `gcp-open5gs-vm`) from the list.
    *   A new VS Code window will open. You will be prompted for your **SSH key passphrase**. Enter it to complete the connection.
    * If the SSH connection fails , its likely due to conflict of hostname IP with the store list of previously used IPs. Check the `C:\Users\vishalkumar.shaw\.ssh\known_hosts` and remove the conflicting entries.

You are now successfully connected. The VS Code terminal and file explorer are directly linked to your remote GCP machine. 

### **Part 4: Component Installation & Configuration**

Now, SSH into each VM and perform the setup.

#### **4.1. Open5GS Core Network Setup (on `open5gs-vm`)**

##### 1. **Install Prerequisites and Repositories**

First, prepare the system by installing necessary software and adding the required repositories for both MongoDB and Open5GS.

*   **Update System and Install `software-properties-common`:**
    ```bash
    sudo apt update
    sudo apt install nano iptables git
    sudo apt install software-properties-common curl gnupg -y
    ```

*   **Install MongoDB:**
    Open5GS requires MongoDB, which must be installed from its official repository.

    a. **Import the MongoDB GPG Key:**
    ```bash
    curl -fsSL https://pgp.mongodb.com/server-8.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg --dearmor
    ```

    b. **Add the MongoDB Repository for Ubuntu 24.04 (Noble):**
    ```bash
    echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg] https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
    ```

    c. **Install and Enable MongoDB:**
    ```bash
    sudo apt update
    sudo apt install -y mongodb-org
    sudo systemctl start mongod
    sudo systemctl enable mongod
    ```

*   **Add the Open5GS PPA Repository:**
    ```bash
    sudo add-apt-repository ppa:open5gs/latest
    sudo apt update
    ```

##### 2. **Install Open5GS**

With all dependencies and repositories in place, you can now install Open5GS.

```bash
sudo apt install open5gs -y
```

##### 3. **Configure Open5GS Components**

Update the configuration files with the new IP address of your `open5gs-vm`.

*   **In `/etc/open5gs/amf.yaml`:**
    ```yaml
    #...
    amf:
      sbi: #...
      ngap:
        server:
          - address: 172.17.0.95 # open5gs vm ip
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
          - address: 172.17.0.95 # open5gs vm ip
    #...
    	
	
	# restart upf services
	sudo systemctl restart open5gs-upfd
	
	# upf logs can be found in /var/log/open5gs/upf.log
	sudo tail -f /var/log/open5gs/upf.log
    ```
*   The `smf.yaml` and `nssf.yaml` files do not need IP changes for this basic setup. The default slice configurations are sufficient to start.

##### 4. **NAT Port Forwarding**

```bash
# nat port forwarding
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -o ens4 -j MASQUERADE
sudo iptables -I FORWARD 1 -j ACCEPT
```

##### 5. **Setup Open5GS WebUI**

*   **Install Node.js and dependencies:**
    ```bash
    sudo apt update
    sudo apt install curl
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
*   **Start the WebUI:**
    This command will run the user interface in the foreground.
    ```bash
    npm run dev
    ```
*   **Accessing the WebUI:**
    * VS-Code by default does port-forwarding and opens in on your localhost. Or you can do it other way given below.
    * The WebUI is now running on `localhost:3000` inside the VM. To access it from your computer, open a **new local terminal** (not the browser SSH) and run:
      ```bash
      gcloud compute ssh open5gs-vm --project=YOUR_PROJECT_ID --zone=us-central1-a -- -L 3000:localhost:3000
      ```
    *   Now, open a browser on your local machine and go to `http://localhost:3000`.
    *   Login with default credentials: `admin` / `1423`.
    *   Add a new subscriber as per the document's instructions (IMSI, Key, OPC).
	    * IMSI: 999700000000001
	    * Subscriber Key: 465B5CE8B199B49FAA5F0A2EE238A6BC
	    * USIM Type: OPc
	    * Operator Key: E8ED289DEBA952E4283B54E88E6183CA
#### **4.2. OAI RAN Setup (on RAN VMs)**

On **each** of the four OAI VMs (`oai-cucp-vm`, `oai-cuup-vm`, `oai-du-vm`, `oai-nr-ue-vm`), perform the following installation steps.

1.  **Install Dependencies and Clone OAI:**
    ```bash
    # This may take a while
    sudo apt update
    sudo apt install git -y
    cd ~
    git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git
    cd openairinterface5g
    source oaienv
    ```
2.  **Build OAI RAN Binaries:**
    ```bash
    cd cmake_targets
    ./build_oai -I -w SIMU --gNB --build-e2 --ninja
    ./build_oai -I -w SIMU --nrUE --build-e2 --ninja
    ```
    This builds all the necessary components on each machine.

#### **4.3. Configure OAI Components**
Now, configure each OAI component by creating/editing the `.conf` files. Use `nano` or `vim` to edit the files.

*   **On `oai-cucp-vm` (`~/oai-cucp.conf`):**
    *   Create this file and paste the configuration. **Crucially, update the IP addresses.**

```conf
Active_gNBs = ( "oai-cu-cp");
# Asn1_verbosity, choice in: none, info, annoying
Asn1_verbosity = "info";

gNBs =
(
 {
    ////////// Identification parameters:
    gNB_ID = 0xe00;

#     cell_type =  "CELL_MACRO_GNB";

    gNB_name  =  "oai-cu-cp";

    // Tracking area code, 0x0000 and 0xfffe are reserved values
    tracking_area_code  =  1;
    plmn_list = ({ mcc = 999; mnc = 70; mnc_length = 2; snssaiList = ({ sst = 1, sd = 0x000001 }, { sst = 1, sd = 0xFFFFFF }, { sst = 2, sd = 0x111111 }) });


    nr_cellid = 12345678L;

    tr_s_preference = "f1";

    local_s_address = "172.17.0.93"; # cucp vm ip
    remote_s_address = "172.17.0.92"; # du vm ip
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


    ////////// AMF parameters:
    amf_ip_address = ({ ipv4 = "172.17.0.95"; });

    E1_INTERFACE =
    (
      {
        type = "cp";
        ipv4_cucp = "172.17.0.93"; # cucp vm ip
        port_cucp = 38462;
        ipv4_cuup = "172.17.0.94"; # cuup vm ip
        port_cuup = 38462;
      }
    )

    NETWORK_INTERFACES :
    {
        GNB_IPV4_ADDRESS_FOR_NG_AMF              = "172.17.0.93"; # cucp vm ip
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
     log_config :
     {
       global_log_level                      ="info";
       hw_log_level                          ="info";
       phy_log_level                         ="info";
       mac_log_level                         ="info";
       rlc_log_level                         ="debug";
       pdcp_log_level                        ="info";
       rrc_log_level                         ="info";
       f1ap_log_level                         ="info";
       ngap_log_level                         ="debug";
       sctp_log_level                         ="info";
    };

# e2_agent = {
#   near_ric_ip_addr = "10.0.9.20";
#   sm_dir = "/usr/local/lib/flexric/"
# }
```

*   **On `oai-cuup-vm` (`~/oai-cuup.conf`):**
```conf
Active_gNBs = ( "oai-cu-cp");
# Asn1_verbosity, choice in: none, info, annoying
Asn1_verbosity = "none";

gNBs =
(
 {
    ////////// Identification parameters:
    gNB_ID = 0xe00;
    gNB_CU_UP_ID = 0xe00;

#     cell_type =  "CELL_MACRO_GNB";

    gNB_name  =  "oai-cuup-sd1";

    // Tracking area code, 0x0000 and 0xfffe are reserved values
    tracking_area_code  =  1;
    plmn_list = ({ mcc = 999; mnc = 70; mnc_length = 2; snssaiList = ({ sst = 1, sd = 0x000001 },{ sst = 1, sd = 0xFFFFFF }) });


    tr_s_preference = "f1";

    local_s_address = "172.17.0.94";  #cu-up vm IP
    remote_s_address = "172.17.0.92"; #du vm IP
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
        type = "up";
        ipv4_cucp = "172.17.0.93"; #cu-cp vm IP
        ipv4_cuup = "172.17.0.94"; #cu-up vm IP
      }
    )

    NETWORK_INTERFACES :
    {
        GNB_IPV4_ADDRESS_FOR_NG_AMF              = "172.17.0.95"; # AMF IP
        GNB_IPV4_ADDRESS_FOR_NGU                 = "172.17.0.94"; # CU-UP IP
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

*   **On `oai-du-vm` (`~/oai-du.conf`):**
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

*   **On `oai-nr-ue-vm` (`~/nr-ue.conf`):**
```conf
uicc0 = {
  imsi = "999700000000001";
  key = "465B5CE8B199B49FAA5F0A2EE238A6BC";
  opc = "E8ED289DEBA952E4283B54E88E6183CA";
  dnn = "internet";
  nssai_sst = 1;
  nssai_sd = 0xFFFFFF;
}

```

### **Part 5: Starting and Testing the Network**

Start the components in the correct order. SSH into each VM and run the commands in its terminal.

1.  **Start Open5GS Core (on `open5gs-vm`):**
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
```

2.  **Start OAI RAN Components (in separate SSH sessions):**
    *   **On `oai-cucp-vm`:**
        ```bash
        cd ~/openairinterface5g/cmake_targets/ran_build/build
        
        sudo -E ./nr-softmodem -O /etc/oai/oai-cucp-ho.conf --sa --       telnetsrv --telnetsrv.shrmod ci
        
        sudo ./nr-softmodem -O /etc/oai/oai-cucp.conf --sa
        ```
    *   **On `oai-cuup-vm`:**
        ```bash
        cd ~/openairinterface5g/cmake_targets/ran_build/build

        sudo -E ./nr-cuup -O /etc/oai/oai-cuup-ho.conf --telnetsrv --telnetsrv.shrmod ci

        sudo ./nr-cuup -O /etc/oai/oai-cuup.conf --sa
        ```
    *   **On `oai-du-vm`:**
        ```bash
        cd ~/openairinterface5g/cmake_targets/ran_build/build

        sudo -E ./nr-softmodem -O /etc/oai/oai-du.conf --rfsim --sa --telnetsrv --telnetsrv.shrmod ci

        sudo ./nr-softmodem -O /etc/oai/oai-du.conf --rfsim --sa
        ```
    *   Wait a few seconds for the gNB components to connect to each other and the core.

3.  **Start OAI NR UE (on `oai-nr-ue-vm`):**
    ```bash
    cd ~/openairinterface5g/cmake_targets/ran_build/build
    sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --ssb 516 --rfsim --rfsimulator.serveraddr 172.17.0.92 -O /etc/oai/nr-ue.conf 
    ```

#### **5.1. Testing End-to-End Connectivity**

If the startup is successful, the UE will connect to the network. You will see log messages indicating RRC connection and registration success.

1.  **Check the Tunnel Interface (on `oai-nr-ue-vm`):**
    *   A new network interface `oaitun_ue1` will appear. Check it with:
        ```bash
        ip a
        ```
    *   You should see the interface has an IP address from the `10.45.0.0/16` pool (e.g., `10.45.0.2`).

2.  **Ping Google (on `oai-nr-ue-vm`):**
    *   Test internet connectivity through the 5G core and the Cloud NAT.
        ```bash
        ping -I oaitun_ue1 8.8.8.8
        ```
    *   You should see successful ping replies, confirming full end-to-end data plane connectivity!

![[Pasted image 20250801140232.png]]
![[Pasted image 20250801140209.png]]
![[Pasted image 20250801140153.png]]
### **Part 6: Conclusion and CRITICAL Cleanup**

You have successfully deployed a 5G Standalone network on GCP. The OAI RAN components are running on dedicated VMs, communicating with the Open5GS core, and providing internet connectivity to a simulated UE via a cloud-native NAT Gateway.

#### **To avoid ANY charges, you MUST tear down your environment:**

1.  **Delete the VMs:**
    *   Go to **Compute Engine > VM instances**.
    *   Select all five VMs (`open5gs-vm`, `oai-cucp-vm`, etc.).
    *   Click **DELETE**.

2.  **Delete the Cloud NAT Gateway:**
    *   Go to **Network services > Cloud NAT**.
    *   Select the `oai-5g-nat` gateway.
    *   Click **DELETE**.

3.  **(Optional) Delete the VPC, Firewall Rules, and Static IPs** if you do not plan to use them again. This ensures your project is completely clean.



---
### then 

log of amf when ue connects 
```bash
Aug 07 07:46:11 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/07 07:46:11.760: [amf] INFO: gNB-N2 accepted[172.17.0.93]:48229 in ng-path module (../src/amf/ngap-sctp.c:113)
Aug 07 07:46:11 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/07 07:46:11.760: [amf] INFO: gNB-N2 accepted[172.17.0.93] in master_sm module (../src/amf/amf-sm.c:894)
Aug 07 07:46:11 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/07 07:46:11.773: [amf] INFO: [Added] Number of gNBs is now 1 (../src/amf/context.c:1277)
Aug 07 07:46:11 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/07 07:46:11.773: [amf] INFO: gNB-N2[172.17.0.93] max_num_of_ostreams : 5 (../src/amf/amf-sm.c:941)
Aug 07 07:47:47 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/07 07:47:47.920: [amf] INFO: InitialUEMessage (../src/amf/ngap-handler.c:437)
Aug 07 07:47:47 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/07 07:47:47.920: [amf] INFO: [Added] Number of gNB-UEs is now 1 (../src/amf/context.c:2789)
Aug 07 07:47:47 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/07 07:47:47.920: [amf] INFO:     RAN_UE_NGAP_ID[1] AMF_UE_NGAP_ID[2] TAC[1] CellID[0xe0000] (../src/amf/ngap-handler.c:598)
Aug 07 07:47:47 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/07 07:47:47.920: [amf] INFO: [suci-0-999-70-0000-0-0-0000000001] known UE by SUCI (../src/amf/context.c:1904)
Aug 07 07:47:47 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/07 07:47:47.920: [gmm] INFO: Registration request (../src/amf/gmm-sm.c:1339)
Aug 07 07:47:47 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/07 07:47:47.920: [gmm] INFO: [suci-0-999-70-0000-0-0-0000000001]    SUCI (../src/amf/gmm-handler.c:183)
Aug 07 07:47:47 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/07 07:47:47.931: [amf] INFO: [imsi-999700000000001:10] Release SM context [204] (../src/amf/amf-sm.c:581)
Aug 07 07:47:47 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/07 07:47:47.931: [amf] INFO: [imsi-999700000000001:10] Release SM Context [state:31] (../src/amf/nsmf-handler.c:1168)
Aug 07 07:47:47 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/07 07:47:47.931: [amf] INFO: [Removed] Number of AMF-Sessions is now 0 (../src/amf/context.c:2817)
Aug 07 07:47:47 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/07 07:47:47.936: [amf] WARNING: UnRef NF EndPoint(addr) [127.0.0.11:7777] (../src/amf/nausf-handler.c:130)
Aug 07 07:47:47 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/07 07:47:47.936: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.11:7777] (../src/amf/nausf-handler.c:130)
Aug 07 07:47:48 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/07 07:47:48.031: [amf] WARNING: UnRef NF EndPoint(addr) [127.0.0.13:7777] (../src/amf/npcf-handler.c:143)
Aug 07 07:47:48 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/07 07:47:48.031: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.13:7777] (../src/amf/npcf-handler.c:143)
Aug 07 07:47:48 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/07 07:47:48.115: [gmm] INFO: [imsi-999700000000001] Registration complete (../src/amf/gmm-sm.c:2698)
Aug 07 07:47:48 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/07 07:47:48.115: [amf] INFO: [imsi-999700000000001] Configuration update command (../src/amf/nas-path.c:609)
Aug 07 07:47:48 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/07 07:47:48.115: [gmm] INFO:     UTC [2025-08-07T07:47:48] Timezone[0]/DST[0] (../src/amf/gmm-build.c:551)
Aug 07 07:47:48 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/07 07:47:48.115: [gmm] INFO:     LOCAL [2025-08-07T07:47:48] Timezone[0]/DST[0] (../src/amf/gmm-build.c:556)
Aug 07 07:47:48 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/07 07:47:48.115: [amf] INFO: [Added] Number of AMF-Sessions is now 1 (../src/amf/context.c:2810)
Aug 07 07:47:48 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/07 07:47:48.115: [gmm] INFO: UE SUPI[imsi-999700000000001] DNN[internet] S_NSSAI[SST:1 SD:0xffffff] smContextRef[NULL] smContextResourceURI[NULL] (../src/amf/gmm-handler.c:1383)
Aug 07 07:47:48 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/07 07:47:48.115: [amf] INFO: [a080ecc2-7356-41f0-a8bf-273b5ec79518] Setup NF Instance [type:SMF] (../src/amf/context.c:2433)
Aug 07 07:47:48 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/07 07:47:48.115: [gmm] INFO: SMF Instance [a080ecc2-7356-41f0-a8bf-273b5ec79518] (../src/amf/gmm-handler.c:1424)
Aug 07 07:47:48 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/07 07:47:48.123: [amf] INFO: Setup NF EndPoint(addr) [127.0.0.4:7777] (../src/amf/nsmf-handler.c:144)
Aug 07 07:47:48 open5gs-vm.us-central1-c.c.g-oai-open5gs-proj.internal open5gs-amfd[541]: 08/07 07:47:48.161: [amf] INFO: [imsi-999700000000001:10:11][0:0:NULL] /nsmf-pdusession/v1/sm-contexts/{smContextRef}/modify (../src/amf/nsmf-handler.c:947)
```

CUCP LOG
```bash
telcomaan@oai-cucp-vm:~/openairinterface5g/cmake_targets/ran_build/build$ sudo -E ./nr-softmodem -O /etc/oai/oai-cucp-ho.conf --sa --telnetsrv --telnetsrv.shrmod ci
CMDLINE: "./nr-softmodem" "-O" "/etc/oai/oai-cucp-ho.conf" "--sa" "--telnetsrv" "--telnetsrv.shrmod" "ci" 
[CONFIG] function config_libconfig_init returned 0
[LOADER] library libtelnetsrv_gnb.so is not loaded: libtelnetsrv_gnb.so: cannot open shared object file: No such file or directory
[TELNETSRV] Telnet server: module 0 = telnet added to shell
[TELNETSRV] Telnet server: module 1 = softmodem added to shell
[TELNETSRV] couldn't find add_phy_cmds for module phy 
[TELNETSRV] Telnet server: module 2 = loader added to shell
[TELNETSRV] Telnet server: module 3 = measur added to shell
policy set to other, priority 0
Error 3: No such process trying to get nice value of thread 442496704 
[TELNETSRV] 
Initializing telnet server...
[TELNETSRV] Telnet server: module 4 = ci added to shell
[UTIL]   running in SA mode (no --phy-test, --do-ra, --nsa option present)
[OPT]   OPT disabled
[HW]   Version: Branch: develop Abrev. Hash: b763692792 Date: Fri Aug 1 04:40:24 2025 +0000
[GNB_APP]   Initialized RAN Context: RC.nb_nr_inst = 1, RC.nb_nr_macrlc_inst = 0, RC.nb_nr_L1_inst = 0, RC.nb_RU = 0, RC.nb_nr_CC[0] = 0
[GNB_APP]   F1AP: gNB_CU_id[0] 3584
[GNB_APP]   F1AP: gNB_CU_name[0] oai-cu-cp
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
[UTIL]   threadCreate() for TASK_GNB_APP: creating thread with affinity ffffffff, priority 50
[NR_RRC]   Entering main loop of NR_RRC message task
[NGAP]   Send NGSetupRequest to AMF
[NGAP]   3584 -> 0000e000
[UTIL]   threadCreate() for TASK_CU_F1: creating thread with affinity ffffffff, priority 50
[F1AP]   Starting F1AP at CU
[UTIL]   threadCreate() for TASK_CUCP_E1: creating thread with affinity ffffffff, priority 50
[F1AP]   F1AP_CU_SCTP_REQ(create socket) for 172.17.0.93 len 12
[F1AP]   In F1AP connection, don't start GTP-U, as we have also E1AP
[UTIL]   threadCreate() for time source realtime: creating thread with affinity ffffffff, priority 2
[E1AP]   Starting E1AP at CU CP
[GTPU]   Configuring GTPu
[GTPU]   SA mode 
[E1AP]   E1AP_CUCP_SCTP_REQ(create socket) for 172.17.0.93 len 12
[NGAP]   Served GUAMIs for AMF (no name) (assoc_id=13):
[NGAP]    GUAMI:
[NGAP]      PLMN: MCC=999, MNC=70
[NGAP]      AMF Region ID: 2
[NGAP]      AMF Set ID: 1
[NGAP]      AMF Pointer: 0
[NGAP]   Supported PLMN 0: MCC=999 MNC=70
[NGAP]   Supported slice (PLMN 0): SST=0x01 SD=000
[NGAP]   Received NGSetupResponse from AMF
[GNB_APP]   [gNB 0] Received NGAP_REGISTER_GNB_CNF: associated AMF 1
[UTIL]   time manager configuration: [time source: reatime] [mode: standalone] [server IP: 127.0.0.1} [server port: 7374] (server IP/port not used)
TYPE <CTRL-C> TO TERMINATE
[NR_RRC]   Accepting new CU-UP ID 3584 name oai-cuup (assoc_id 15)
[NR_RRC]   Received F1 Setup Request from gNB_DU 3585 (oai-cu-cp) on assoc_id 16
[NR_RRC]   Accepting DU 3585 (oai-cu-cp), sending F1 Setup Response
[NR_RRC]   DU uses RRC version 17.3.0
[NR_RRC]   cell PLMN 999.70 Cell ID 12345678 is in service
[NR_RRC]   Decoding CCCH: RNTI 457a, payload_size 6
[NR_RRC]   [--] (cellID 0, UE ID 1 RNTI 457a) Create UE context: CU UE ID 1 DU UE ID 17786 (rnti: 457a, random ue id 307f353824000000)
[RRC]   activate SRB 1 of UE 1
[NR_RRC]   [DL] (cellID bc614e, UE ID 1 RNTI 457a) Send RRC Setup
[NR_RRC]   [UL] (cellID bc614e, UE ID 1 RNTI 457a) Received RRCSetupComplete (RRC_CONNECTED reached)
[NGAP]   Selected PLMN in the NG Initial UE Message: MCC 999, MNC 70
[NGAP]   UE 1: Chose AMF 'open5gs-amf0' (assoc_id 13) through selected PLMN MCC=999 MNC=70
[NGAP]   Create UE context (ID 1) for AMF 'open5gs-amf0' (assoc_id 13)
[NR_RRC]   [DL] (cellID bc614e, UE ID 1 RNTI 457a) Send DL Information Transfer [42 bytes]
[NR_RRC]   [UL] (cellID bc614e, UE ID 1 RNTI 457a) Received RRC UL Information Transfer [24 bytes]
[NR_RRC]   [DL] (cellID bc614e, UE ID 1 RNTI 457a) Send DL Information Transfer [19 bytes]
[NR_RRC]   [UL] (cellID bc614e, UE ID 1 RNTI 457a) Received RRC UL Information Transfer [60 bytes]
[NGAP]   NGAP_FIND_PROTOCOLIE_BY_ID ie is NULL (searching for ie: 110)
[NGAP]   NGAP_FIND_PROTOCOLIE_BY_ID ie is NULL (searching for ie: 71)
[NGAP]   AllowedNSSAI.list.count 1
[NGAP]   NGAP_FIND_PROTOCOLIE_BY_ID ie is NULL (searching for ie: 36)
[NR_RRC]   [--] (cellID bc614e, UE ID 1 RNTI 457a) Selected security algorithms: ciphering 0, integrity 2
[NR_RRC]   [UE 457a] Saved security key BC
[NR_RRC]   UE 1 Logical Channel DL-DCCH, Generate SecurityModeCommand (bytes 3)
[NR_RRC]   [UL] (cellID bc614e, UE ID 1 RNTI 457a) Received Security Mode Complete
[NR_RRC]   UE 1: Logical Channel DL-DCCH, Generate NR UECapabilityEnquiry (bytes 8, xid 1)
[NR_RRC]   [UL] (cellID bc614e, UE ID 1 RNTI 457a) Received UE capabilities
[NR_RRC]   Send message to ngap: NGAP_UE_CAPABILITIES_IND
[NR_RRC]   [DL] (cellID bc614e, UE ID 1 RNTI 457a) Send DL Information Transfer [46 bytes]
[NR_RRC]   Send message to sctp: NGAP_InitialContextSetupResponse
[NR_RRC]   [UL] (cellID bc614e, UE ID 1 RNTI 457a) Received RRC UL Information Transfer [13 bytes]
[NR_RRC]   [UL] (cellID bc614e, UE ID 1 RNTI 457a) Received RRC UL Information Transfer [40 bytes]
[NR_RRC]   [DL] (cellID bc614e, UE ID 1 RNTI 457a) Send DL Information Transfer [51 bytes]
[NGAP]   PDUSESSIONSetup initiating message
[NR_RRC]   UE 1: received PDU Session Resource Setup Request
[NR_RRC]   Bearer Context Setup: PDU Session ID=10, incoming TEID=0x0000bd14, Addr=172.17.0.95
[NR_RRC]   UE 1: configure DRB ID 1 for PDU session ID 10
[NR_RRC]   [--] (cellID bc614e, UE ID 1 RNTI 457a) second best match: CU-UP ID 3584 matches SST 1
[NR_RRC]   [--] (cellID bc614e, UE ID 1 RNTI 457a) selecting CU-UP ID 3584 based on exact NSSAI match (1:0xffffff)
[RRC]   UE 1 associating to CU-UP assoc_id 15 out of 1 CU-UPs
[RRC]   activate SRB 2 of UE 1
[RRC]   UE 1 trigger UE context setup request with 1 DRBs
[RRC]   UE 457a replacing existing CellGroupConfig with new one received from DU
[NR_RRC]   [DL] (cellID bc614e, UE ID 1 RNTI 457a) Generate RRCReconfiguration (bytes 283, xid 0)
[RRC]   UE 1: PDU session ID 10 modified 1 bearers
[NR_RRC]   [UL] (cellID bc614e, UE ID 1 RNTI 457a) Received RRCReconfigurationComplete
[NR_RRC]   PDU Session Setup Response: ID=10, outgoing TEID=0xc55ab9b4, Addr=172.17.0.93
[NR_RRC]   NGAP_PDUSESSION_SETUP_RESP: sending the message
[NGAP]   Encoded PDU Session Transfer (10): TEID=0xc55ab9b4, Addr=172.17.0.93
[NR_RRC]   Received F1 Setup Request from gNB_DU 3586 (oai-cu-cp) on assoc_id 17
[NR_RRC]   Accepting DU 3586 (oai-cu-cp), sending F1 Setup Response
[NR_RRC]   DU uses RRC version 17.3.0
[NR_RRC]   cell PLMN 999.70 Cell ID 12345679 is in service
[TELNETSRV] Telnet client connected....
[TELNETSRV] Command received: readc 17 filled 17 "ci trigger_f1_ho"
[NR_RRC]   Handover triggered for UE 1/RNTI 457a towards DU 3586/assoc_id 17/PCI 1
[RRC]   UE 457a replacing existing CellGroupConfig with new one received from DU
[NR_RRC]   HO acknowledged: Send reconfiguration for UE 1/RNTI 457a...
[PDCP]   SRB 2 re-established
[RRC]   UE 1: PDU session ID 10 modified 1 bearers
[NR_RRC]   UE 1 handover: update RNTI from 457a to f401
[NR_RRC]   UE 1: release request from source DU ID 3585 during HO, marking HO as complete
```

CUUP LOG
```bash
telcomaan@oai-cucp-vm:~/openairinterface5g/cmake_targets/ran_build/build$ sudo -E ./nr-cuup -O /etc/oai/oai-cuup-ho.conf --telnetsrv --telnetsrv.shrmod ci
CMDLINE: "./nr-cuup" "-O" "/etc/oai/oai-cuup-ho.conf" "--telnetsrv" "--telnetsrv.shrmod" "ci" 
[CONFIG] function config_libconfig_init returned 0
[UTIL]   threadCreate() for time source realtime: creating thread with affinity ffffffff, priority 2
[UTIL]   time manager configuration: [time source: reatime] [mode: standalone] [server IP: 127.0.0.1} [server port: 7374] (server IP/port not used)
[HW]   Version: Branch: develop Abrev. Hash: b763692792 Date: Fri Aug 1 04:40:24 2025 +0000
[UTIL]   threadCreate() for TASK_SCTP: creating thread with affinity ffffffff, priority 50
[UTIL]   threadCreate() for TASK_GTPV1_U: creating thread with affinity ffffffff, priority 50
[UTIL]   threadCreate() for TASK_CUUP_E1: creating thread with affinity ffffffff, priority 50
[E1AP]   Starting E1AP at CU UP
[GTPU]   Configuring GTPu
[GTPU]   SA mode 
TYPE <CTRL-C> TO TERMINATE
[GTPU]   Initializing UDP for local address 172.17.0.93 with port 2153
[GTPU]   Created gtpu instance id: 92
[GTPU]   Configuring GTPu address : 172.17.0.93, port : 2152
[GTPU]   Initializing UDP for local address 172.17.0.93 with port 2152
[GTPU]   Created gtpu instance id: 93
[E1AP]   E1 connection established (SCTP_STATE_ESTABLISHED)
[E1AP]   adding UE with CU-CP UE ID 1 and CU-UP UE ID 1
[E1AP]   UE 1: add PDU session ID 10 (1 bearers)
[GTPU]   [93] Created tunnel for UE ID 1, teid for incoming: c55ab9b4, teid for outgoing bd14 to remote IPv4: 172.17.0.95, IPv6 ::
[PDCP]   added drb 1 to UE ID 1
[SDAP]   Default DRB for the created SDAP entity: 1 
[GTPU]   [92] Created tunnel for UE ID 1, teid for incoming: 9ec42d5a, teid for outgoing ffff to remote IPv4: 0.0.0.0, IPv6 ::
[E1AP]   UE 1: updating PDU session ID 10 (1 bearers)
[PDCP]   DRB 1 re-established
[GTPU]   [92] Tunnel Outgoing TEID updated to 26e8e323 and address to 5c0011ac
[E1AP]   UE 1: updating PDU session ID 10 (1 bearers)
[PDCP]   DRB 1 re-established
[GTPU]   [92] Tunnel Outgoing TEID updated to 691a163e and address to 5e0011ac
[E1AP]   releasing UE 1
```

DU LOG
```bash
telcomaan@oai-du-vm:~/openairinterface5g/cmake_targets/ran_build/build$ sudo -E ./nr-softmodem -O /etc/oai/oai-du.conf --rfsim --sa --telnetsrv --telnetsrv.shrmod ci
CMDLINE: "./nr-softmodem" "-O" "/etc/oai/oai-du.conf" "--rfsim" "--sa" "--telnetsrv" "--telnetsrv.shrmod" "ci" 
[CONFIG] function config_libconfig_init returned 0
[LOADER] library libtelnetsrv_gnb.so is not loaded: libtelnetsrv_gnb.so: cannot open shared object file: No such file or directory
[TELNETSRV] Telnet server: module 0 = telnet added to shell
[TELNETSRV] Telnet server: module 1 = softmodem added to shell
[TELNETSRV] couldn't find add_phy_cmds for module phy 
[TELNETSRV] Telnet server: module 2 = loader added to shell
[TELNETSRV] Telnet server: module 3 = measur added to shell
policy set to other, priority 0
Error 3: No such process trying to get nice value of thread 826275520 
[TELNETSRV] 
Initializing telnet server...
[TELNETSRV] Telnet server: module 4 = ci added to shell
[UTIL]   running in SA mode (no --phy-test, --do-ra, --nsa option present)
[OPT]   OPT disabled
[HW]   Version: Branch: develop Abrev. Hash: b763692792 Date: Fri Aug 1 04:40:24 2025 +0000
[GNB_APP]   Initialized RAN Context: RC.nb_nr_inst = 1, RC.nb_nr_macrlc_inst = 1, RC.nb_nr_L1_inst = 1, RC.nb_RU = 1, RC.nb_nr_CC[0] = 1
[NR_PHY]   Initializing gNB RAN context: RC.nb_nr_L1_inst = 1 
[NR_PHY]   Registered with MAC interface module (0x64d9926b44e0)
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
[GTPU]   Created gtpu instance id: 97
[MAC]   received F1 Setup Response from CU oai-cu-cp
[MAC]   CU uses RRC version 17.3.0
[MAC]   Clearing the DU's UE states before, if any.
[MAC]   received gNB-DU configuration update acknowledge
[UTIL]   threadCreate() for time source iq samples: creating thread with affinity ffffffff, priority 2
[UTIL]   time manager configuration: [time source: iq_samples] [mode: standalone] [server IP: 127.0.0.1} [server port: 7374] (server IP/port not used)
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
[HW]   Running as server waiting opposite rfsimulators to connect
Initializing random number generator, seed 15409201791792170320
[TELNETSRV] Telnet server: module 5 = rfsimu added to shell
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
[PHY]   RU 0 rf device ready
[PHY]   RU 0 RF started cpu_meas_enabled 0
[HW]   No connected device, generating void samples...
[PHY]   Command line parameters for OAI UE: -C 3619200000 -r 106 --numerology 1 --ssb 516 
[NR_MAC]   Frame.Slot 128.0

[NR_MAC]   Frame.Slot 256.0

[NR_MAC]   Frame.Slot 384.0

[NR_MAC]   Frame.Slot 512.0

[NR_MAC]   Frame.Slot 640.0

[NR_MAC]   Frame.Slot 768.0

[HW]   Client connects from ::ffff:172.17.0.91:42153
[NR_MAC]   Frame.Slot 896.0

[NR_PHY]   [RAPROC] 981.19 Initiating RA procedure with preamble 22, energy 42.9 dB (I0 0, thres 200), delay 0 start symbol 8 freq index 0
[NR_MAC]   981.19 UE RA-RNTI 0113 TC-RNTI 457a: initiating RA procedure
[NR_MAC]   UE 457a: Msg3 scheduled at 982.17 (982.7 TDA 3) start 0 RBs 8
[NR_MAC]   UE 457a: 982.7 Generating RA-Msg2 DCI, RA RNTI 0x113, state 1, preamble_index(RAPID) 22, timing_offset = 0 (estimated distance 0.0 [m])
[NR_MAC]   982.7 Send RAR to RA-RNTI 0113
[NR_MAC]    982.17 PUSCH with TC_RNTI 0x457a received correctly
[MAC]   [RAPROC] Received SDU for CCCH length 6 for UE 457a
[RLC]   Activated srb0 for UE 17786
[RLC]   Added srb 1 to UE 17786
[NR_MAC]   Activating scheduling Msg4 for TC_RNTI 0x457a (state WAIT_Msg3)
[NR_MAC]   Cannot find free vrb_map for RNTI 457a!
[NR_MAC]   UE 457a Generate Msg4: feedback at  984. 7, payload 161 bytes, next state nrRA_WAIT_Msg4_MsgB_ACK
[NR_MAC]    984. 7 UE 457a: Received Ack of Msg4. CBRA procedure succeeded (UE Connected)
[NR_MAC]   Adding new UE context with RNTI 0x457a
[NR_MAC]   Frame.Slot 0.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 0 dB PCMAX 0 dBm, average RSRP -44 (4 meas)
UE 457a: dlsch_rounds 1/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE 457a: ulsch_rounds 40/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.07290 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 50.5 dB
UE 457a: MAC:    TX              0 RX            117 bytes
UE 457a: LCID 1: TX              0 RX              0 bytes

[RLC]   Added srb 2 to UE 17786
[RLC]   Added drb 1 to UE 17786
[RLC]   Added DRB to UE 17786
[GTPU]   [97] Created tunnel for UE ID 17786, teid for incoming: 26e8e323, teid for outgoing 9ec42d5a to remote IPv4: 172.17.0.93, IPv6 ::
[NR_MAC]   DU received confirmation of successful RRC Reconfiguration
[NR_MAC]   Frame.Slot 128.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 17/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.05905 MCS (0) 0
UE 457a: ulsch_rounds 172/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.02059 MCS (0) 7 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX            758 RX           1411 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX              0 RX              0 bytes

[NR_MAC]   Frame.Slot 256.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 19/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.04783 MCS (0) 0
UE 457a: ulsch_rounds 188/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00523 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX            800 RX           2057 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX              3 RX             64 bytes

[NR_MAC]   Frame.Slot 384.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 21/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.03874 MCS (0) 0
UE 457a: ulsch_rounds 201/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00133 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX            842 RX           2317 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX              3 RX             64 bytes

[NR_MAC]   Frame.Slot 512.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 23/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.03487 MCS (0) 0
UE 457a: ulsch_rounds 288/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00038 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX            884 RX           3129 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX              6 RX            128 bytes

[NR_MAC]   Frame.Slot 640.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 51 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 30/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.02542 MCS (0) 0
UE 457a: ulsch_rounds 339/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00010 MCS (0) 1 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           1371 RX           3945 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            372 RX            408 bytes

[NR_MAC]   Frame.Slot 768.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 34/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.01853 MCS (0) 0
UE 457a: ulsch_rounds 402/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00003 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           1540 RX           4683 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            465 RX            417 bytes

[NR_MAC]   Frame.Slot 896.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 36/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.01668 MCS (0) 0
UE 457a: ulsch_rounds 488/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00001 MCS (0) 2 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           1582 RX           5597 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            468 RX            481 bytes

[NR_MAC]   Frame.Slot 0.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 37/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.01501 MCS (0) 0
UE 457a: ulsch_rounds 501/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           1603 RX           5863 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            468 RX            481 bytes

[NR_MAC]   Frame.Slot 128.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 38/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.01351 MCS (0) 0
UE 457a: ulsch_rounds 513/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           1624 RX           6103 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            468 RX            481 bytes

[NR_MAC]   Frame.Slot 256.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 40/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.01094 MCS (0) 0
UE 457a: ulsch_rounds 526/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           1666 RX           6363 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            468 RX            481 bytes

[NR_MAC]   Frame.Slot 384.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 41/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00985 MCS (0) 0
UE 457a: ulsch_rounds 539/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           1687 RX           6623 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            468 RX            481 bytes

[NR_MAC]   Frame.Slot 512.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 42/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00886 MCS (0) 0
UE 457a: ulsch_rounds 552/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           1708 RX           6883 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            468 RX            481 bytes

[NR_MAC]   Frame.Slot 640.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 45/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00646 MCS (0) 0
UE 457a: ulsch_rounds 631/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 3 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           1771 RX           7732 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            471 RX            545 bytes

[NR_MAC]   Frame.Slot 768.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 46/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00581 MCS (0) 0
UE 457a: ulsch_rounds 644/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           1792 RX           8010 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            471 RX            545 bytes

[NR_MAC]   Frame.Slot 896.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 47/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00523 MCS (0) 0
UE 457a: ulsch_rounds 657/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           1813 RX           8270 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            471 RX            545 bytes

[NR_MAC]   Frame.Slot 0.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 48/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00471 MCS (0) 0
UE 457a: ulsch_rounds 670/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           1834 RX           8530 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            471 RX            545 bytes

[NR_MAC]   Frame.Slot 128.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 50/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00382 MCS (0) 0
UE 457a: ulsch_rounds 682/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           1876 RX           8770 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            471 RX            545 bytes

[NR_MAC]   Frame.Slot 256.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 51/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00343 MCS (0) 0
UE 457a: ulsch_rounds 695/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           1897 RX           9030 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            471 RX            545 bytes

[NR_MAC]   Frame.Slot 384.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 52/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00309 MCS (0) 0
UE 457a: ulsch_rounds 708/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           1918 RX           9290 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            471 RX            545 bytes

[NR_MAC]   Frame.Slot 512.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 54/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00250 MCS (0) 0
UE 457a: ulsch_rounds 721/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           1960 RX           9550 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            471 RX            545 bytes

[NR_MAC]   Frame.Slot 640.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 55/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00225 MCS (0) 0
UE 457a: ulsch_rounds 734/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           1981 RX           9810 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            471 RX            545 bytes

[NR_MAC]   Frame.Slot 768.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 56/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00203 MCS (0) 0
UE 457a: ulsch_rounds 746/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           2002 RX          10050 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            471 RX            545 bytes

[NR_MAC]   Frame.Slot 896.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 57/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00182 MCS (0) 0
UE 457a: ulsch_rounds 759/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           2023 RX          10310 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            471 RX            545 bytes

[NR_MAC]   Frame.Slot 0.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 59/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00148 MCS (0) 0
UE 457a: ulsch_rounds 772/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           2065 RX          10570 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            471 RX            545 bytes

[NR_MAC]   Frame.Slot 128.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 60/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00133 MCS (0) 0
UE 457a: ulsch_rounds 785/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           2086 RX          10830 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            471 RX            545 bytes

[NR_MAC]   Frame.Slot 256.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 62/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00108 MCS (0) 0
UE 457a: ulsch_rounds 866/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 2 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           2128 RX          11632 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            474 RX            609 bytes

[NR_MAC]   Frame.Slot 384.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 63/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00097 MCS (0) 0
UE 457a: ulsch_rounds 878/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           2149 RX          11878 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            474 RX            609 bytes

[NR_MAC]   Frame.Slot 512.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 65/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00079 MCS (0) 0
UE 457a: ulsch_rounds 891/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           2191 RX          12138 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            474 RX            609 bytes

[NR_MAC]   Frame.Slot 640.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 66/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00071 MCS (0) 0
UE 457a: ulsch_rounds 904/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           2212 RX          12398 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            474 RX            609 bytes

[NR_MAC]   Frame.Slot 768.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 67/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00064 MCS (0) 0
UE 457a: ulsch_rounds 917/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           2233 RX          12658 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            474 RX            609 bytes

[NR_MAC]   Frame.Slot 896.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 69/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00052 MCS (0) 0
UE 457a: ulsch_rounds 930/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           2275 RX          12918 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            474 RX            609 bytes

[NR_MAC]   Frame.Slot 0.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 70/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00046 MCS (0) 0
UE 457a: ulsch_rounds 942/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           2296 RX          13158 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            474 RX            609 bytes

[NR_MAC]   Frame.Slot 128.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 71/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00042 MCS (0) 0
UE 457a: ulsch_rounds 955/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           2317 RX          13418 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            474 RX            609 bytes

[NR_MAC]   Frame.Slot 256.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 72/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00038 MCS (0) 0
UE 457a: ulsch_rounds 968/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           2338 RX          13678 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            474 RX            609 bytes

[NR_MAC]   Frame.Slot 384.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 74/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00030 MCS (0) 0
UE 457a: ulsch_rounds 981/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           2380 RX          13938 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            474 RX            609 bytes

[NR_MAC]   Frame.Slot 512.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 75/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00027 MCS (0) 0
UE 457a: ulsch_rounds 994/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           2401 RX          14198 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            474 RX            609 bytes

[NR_MAC]   Frame.Slot 640.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 76/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00025 MCS (0) 0
UE 457a: ulsch_rounds 1006/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           2422 RX          14438 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            474 RX            609 bytes

[NR_MAC]   Frame.Slot 768.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 78/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00020 MCS (0) 0
UE 457a: ulsch_rounds 1019/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           2464 RX          14698 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            474 RX            609 bytes

[NR_MAC]   Frame.Slot 896.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 79/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00018 MCS (0) 0
UE 457a: ulsch_rounds 1032/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           2485 RX          14958 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            474 RX            609 bytes

[NR_MAC]   Frame.Slot 0.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 80/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00016 MCS (0) 0
UE 457a: ulsch_rounds 1045/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           2506 RX          15218 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            474 RX            609 bytes

[NR_MAC]   Frame.Slot 128.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 81/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00015 MCS (0) 0
UE 457a: ulsch_rounds 1058/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           2527 RX          15478 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            474 RX            609 bytes

[NR_MAC]   Frame.Slot 256.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 83/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00012 MCS (0) 0
UE 457a: ulsch_rounds 1070/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           2569 RX          15718 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            474 RX            609 bytes

[NR_MAC]   Frame.Slot 384.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 84/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00011 MCS (0) 0
UE 457a: ulsch_rounds 1083/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           2590 RX          15978 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            474 RX            609 bytes

[NR_MAC]   Frame.Slot 512.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 85/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00010 MCS (0) 0
UE 457a: ulsch_rounds 1096/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           2611 RX          16238 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            474 RX            609 bytes

[NR_MAC]   Frame.Slot 640.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 87/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00008 MCS (0) 0
UE 457a: ulsch_rounds 1109/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           2653 RX          16498 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            474 RX            609 bytes

[NR_MAC]   Frame.Slot 768.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 88/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00007 MCS (0) 0
UE 457a: ulsch_rounds 1122/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           2674 RX          16758 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            474 RX            609 bytes

[NR_MAC]   Frame.Slot 896.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 89/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00006 MCS (0) 0
UE 457a: ulsch_rounds 1134/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           2695 RX          16998 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            474 RX            609 bytes

[NR_MAC]   Frame.Slot 0.0
UE RNTI 457a CU-UE-ID 1 in-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (16 meas)
UE 457a: dlsch_rounds 91/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00005 MCS (0) 0
UE 457a: ulsch_rounds 1211/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 2 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           2737 RX          17778 bytes
UE 457a: LCID 1: TX            545 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            477 RX            673 bytes

[NR_MAC]   gNB-DU received the TransmissionActionIndicator with Stop value for UE 457a
[NR_MAC]   Frame.Slot 128.0
UE RNTI 457a CU-UE-ID 1 out-of-sync PH 48 dB PCMAX 20 dBm, average RSRP -44 (10 meas)
UE 457a: dlsch_rounds 93/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00004 MCS (0) 0
UE 457a: ulsch_rounds 1219/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           3099 RX          17944 bytes
UE 457a: LCID 1: TX            867 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            477 RX            673 bytes

[NR_MAC]   Frame.Slot 256.0
UE RNTI 457a CU-UE-ID 1 out-of-sync PH 48 dB PCMAX 20 dBm, average RSRP 0 (0 meas)
UE 457a: dlsch_rounds 93/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00004 MCS (0) 0
UE 457a: ulsch_rounds 1219/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           3099 RX          17944 bytes
UE 457a: LCID 1: TX            867 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            477 RX            673 bytes

[NR_MAC]   Frame.Slot 384.0
UE RNTI 457a CU-UE-ID 1 out-of-sync PH 48 dB PCMAX 20 dBm, average RSRP 0 (0 meas)
UE 457a: dlsch_rounds 93/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00004 MCS (0) 0
UE 457a: ulsch_rounds 1219/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           3099 RX          17944 bytes
UE 457a: LCID 1: TX            867 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            477 RX            673 bytes

[NR_MAC]   Frame.Slot 512.0
UE RNTI 457a CU-UE-ID 1 out-of-sync PH 48 dB PCMAX 20 dBm, average RSRP 0 (0 meas)
UE 457a: dlsch_rounds 93/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00004 MCS (0) 0
UE 457a: ulsch_rounds 1219/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           3099 RX          17944 bytes
UE 457a: LCID 1: TX            867 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            477 RX            673 bytes

[NR_MAC]   Frame.Slot 640.0
UE RNTI 457a CU-UE-ID 1 out-of-sync PH 48 dB PCMAX 20 dBm, average RSRP 0 (0 meas)
UE 457a: dlsch_rounds 93/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00004 MCS (0) 0
UE 457a: ulsch_rounds 1219/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           3099 RX          17944 bytes
UE 457a: LCID 1: TX            867 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            477 RX            673 bytes

[NR_MAC]   Frame.Slot 768.0
UE RNTI 457a CU-UE-ID 1 out-of-sync PH 48 dB PCMAX 20 dBm, average RSRP 0 (0 meas)
UE 457a: dlsch_rounds 93/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00004 MCS (0) 0
UE 457a: ulsch_rounds 1219/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           3099 RX          17944 bytes
UE 457a: LCID 1: TX            867 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            477 RX            673 bytes

[NR_MAC]   Frame.Slot 896.0
UE RNTI 457a CU-UE-ID 1 out-of-sync PH 48 dB PCMAX 20 dBm, average RSRP 0 (0 meas)
UE 457a: dlsch_rounds 93/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00004 MCS (0) 0
UE 457a: ulsch_rounds 1219/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           3099 RX          17944 bytes
UE 457a: LCID 1: TX            867 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            477 RX            673 bytes

[NR_MAC]   Frame.Slot 0.0
UE RNTI 457a CU-UE-ID 1 out-of-sync PH 48 dB PCMAX 20 dBm, average RSRP 0 (0 meas)
UE 457a: dlsch_rounds 93/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00004 MCS (0) 0
UE 457a: ulsch_rounds 1219/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           3099 RX          17944 bytes
UE 457a: LCID 1: TX            867 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            477 RX            673 bytes

[NR_MAC]   Frame.Slot 128.0
UE RNTI 457a CU-UE-ID 1 out-of-sync PH 48 dB PCMAX 20 dBm, average RSRP 0 (0 meas)
UE 457a: dlsch_rounds 93/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00004 MCS (0) 0
UE 457a: ulsch_rounds 1219/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           3099 RX          17944 bytes
UE 457a: LCID 1: TX            867 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            477 RX            673 bytes

[NR_MAC]   Frame.Slot 256.0
UE RNTI 457a CU-UE-ID 1 out-of-sync PH 48 dB PCMAX 20 dBm, average RSRP 0 (0 meas)
UE 457a: dlsch_rounds 93/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00004 MCS (0) 0
UE 457a: ulsch_rounds 1219/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           3099 RX          17944 bytes
UE 457a: LCID 1: TX            867 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            477 RX            673 bytes

[NR_MAC]   Frame.Slot 384.0
UE RNTI 457a CU-UE-ID 1 out-of-sync PH 48 dB PCMAX 20 dBm, average RSRP 0 (0 meas)
UE 457a: dlsch_rounds 93/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00004 MCS (0) 0
UE 457a: ulsch_rounds 1219/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           3099 RX          17944 bytes
UE 457a: LCID 1: TX            867 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            477 RX            673 bytes

[NR_MAC]   Frame.Slot 512.0
UE RNTI 457a CU-UE-ID 1 out-of-sync PH 48 dB PCMAX 20 dBm, average RSRP 0 (0 meas)
UE 457a: dlsch_rounds 93/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00004 MCS (0) 0
UE 457a: ulsch_rounds 1219/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           3099 RX          17944 bytes
UE 457a: LCID 1: TX            867 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            477 RX            673 bytes

[NR_MAC]   Frame.Slot 640.0
UE RNTI 457a CU-UE-ID 1 out-of-sync PH 48 dB PCMAX 20 dBm, average RSRP 0 (0 meas)
UE 457a: dlsch_rounds 93/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00004 MCS (0) 0
UE 457a: ulsch_rounds 1219/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           3099 RX          17944 bytes
UE 457a: LCID 1: TX            867 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            477 RX            673 bytes

[NR_MAC]   Frame.Slot 768.0
UE RNTI 457a CU-UE-ID 1 out-of-sync PH 48 dB PCMAX 20 dBm, average RSRP 0 (0 meas)
UE 457a: dlsch_rounds 93/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00004 MCS (0) 0
UE 457a: ulsch_rounds 1219/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           3099 RX          17944 bytes
UE 457a: LCID 1: TX            867 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            477 RX            673 bytes

[NR_MAC]   Frame.Slot 896.0
UE RNTI 457a CU-UE-ID 1 out-of-sync PH 48 dB PCMAX 20 dBm, average RSRP 0 (0 meas)
UE 457a: dlsch_rounds 93/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00004 MCS (0) 0
UE 457a: ulsch_rounds 1219/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           3099 RX          17944 bytes
UE 457a: LCID 1: TX            867 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            477 RX            673 bytes

[NR_MAC]   Frame.Slot 0.0
UE RNTI 457a CU-UE-ID 1 out-of-sync PH 48 dB PCMAX 20 dBm, average RSRP 0 (0 meas)
UE 457a: dlsch_rounds 93/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00004 MCS (0) 0
UE 457a: ulsch_rounds 1219/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           3099 RX          17944 bytes
UE 457a: LCID 1: TX            867 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            477 RX            673 bytes

[NR_MAC]   Frame.Slot 128.0
UE RNTI 457a CU-UE-ID 1 out-of-sync PH 48 dB PCMAX 20 dBm, average RSRP 0 (0 meas)
UE 457a: dlsch_rounds 93/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00004 MCS (0) 0
UE 457a: ulsch_rounds 1219/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           3099 RX          17944 bytes
UE 457a: LCID 1: TX            867 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            477 RX            673 bytes

[NR_MAC]   Frame.Slot 256.0
UE RNTI 457a CU-UE-ID 1 out-of-sync PH 48 dB PCMAX 20 dBm, average RSRP 0 (0 meas)
UE 457a: dlsch_rounds 93/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00004 MCS (0) 0
UE 457a: ulsch_rounds 1219/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           3099 RX          17944 bytes
UE 457a: LCID 1: TX            867 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            477 RX            673 bytes

[NR_MAC]   Frame.Slot 384.0
UE RNTI 457a CU-UE-ID 1 out-of-sync PH 48 dB PCMAX 20 dBm, average RSRP 0 (0 meas)
UE 457a: dlsch_rounds 93/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00004 MCS (0) 0
UE 457a: ulsch_rounds 1219/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           3099 RX          17944 bytes
UE 457a: LCID 1: TX            867 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            477 RX            673 bytes

[NR_MAC]   Frame.Slot 512.0
UE RNTI 457a CU-UE-ID 1 out-of-sync PH 48 dB PCMAX 20 dBm, average RSRP 0 (0 meas)
UE 457a: dlsch_rounds 93/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00004 MCS (0) 0
UE 457a: ulsch_rounds 1219/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           3099 RX          17944 bytes
UE 457a: LCID 1: TX            867 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            477 RX            673 bytes

[NR_MAC]   Frame.Slot 640.0
UE RNTI 457a CU-UE-ID 1 out-of-sync PH 48 dB PCMAX 20 dBm, average RSRP 0 (0 meas)
UE 457a: dlsch_rounds 93/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00004 MCS (0) 0
UE 457a: ulsch_rounds 1219/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           3099 RX          17944 bytes
UE 457a: LCID 1: TX            867 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            477 RX            673 bytes

[NR_MAC]   Frame.Slot 768.0
UE RNTI 457a CU-UE-ID 1 out-of-sync PH 48 dB PCMAX 20 dBm, average RSRP 0 (0 meas)
UE 457a: dlsch_rounds 93/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00004 MCS (0) 0
UE 457a: ulsch_rounds 1219/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           3099 RX          17944 bytes
UE 457a: LCID 1: TX            867 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            477 RX            673 bytes

[NR_MAC]   Frame.Slot 896.0
UE RNTI 457a CU-UE-ID 1 out-of-sync PH 48 dB PCMAX 20 dBm, average RSRP 0 (0 meas)
UE 457a: dlsch_rounds 93/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00004 MCS (0) 0
UE 457a: ulsch_rounds 1219/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           3099 RX          17944 bytes
UE 457a: LCID 1: TX            867 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            477 RX            673 bytes

[NR_MAC]   Frame.Slot 0.0
UE RNTI 457a CU-UE-ID 1 out-of-sync PH 48 dB PCMAX 20 dBm, average RSRP 0 (0 meas)
UE 457a: dlsch_rounds 93/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00004 MCS (0) 0
UE 457a: ulsch_rounds 1219/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           3099 RX          17944 bytes
UE 457a: LCID 1: TX            867 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            477 RX            673 bytes

[NR_MAC]   Frame.Slot 128.0
UE RNTI 457a CU-UE-ID 1 out-of-sync PH 48 dB PCMAX 20 dBm, average RSRP 0 (0 meas)
UE 457a: dlsch_rounds 93/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00004 MCS (0) 0
UE 457a: ulsch_rounds 1219/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           3099 RX          17944 bytes
UE 457a: LCID 1: TX            867 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            477 RX            673 bytes

[NR_MAC]   Frame.Slot 256.0
UE RNTI 457a CU-UE-ID 1 out-of-sync PH 48 dB PCMAX 20 dBm, average RSRP 0 (0 meas)
UE 457a: dlsch_rounds 93/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00004 MCS (0) 0
UE 457a: ulsch_rounds 1219/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           3099 RX          17944 bytes
UE 457a: LCID 1: TX            867 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            477 RX            673 bytes

[NR_MAC]   Frame.Slot 384.0
UE RNTI 457a CU-UE-ID 1 out-of-sync PH 48 dB PCMAX 20 dBm, average RSRP 0 (0 meas)
UE 457a: dlsch_rounds 93/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.00004 MCS (0) 0
UE 457a: ulsch_rounds 1219/0/0/0, ulsch_errors 0, ulsch_DTX 0, BLER 0.00000 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 5  SNR 51.0 dB
UE 457a: MAC:    TX           3099 RX          17944 bytes
UE 457a: LCID 1: TX            867 RX            353 bytes
UE 457a: LCID 2: TX              0 RX              0 bytes
UE 457a: LCID 4: TX            477 RX            673 bytes

[MAC]   UE 457a: request release after UL failure timer expiry
[GTPU]   [97] Deleted all tunnels for ue id 17786 (1 tunnels deleted)
[RLC]   Remove UE 17786
[NR_MAC]   Remove NR rnti 0x457a
[GTPU]   try to get a gtp-u not existing output
[NR_MAC]   Frame.Slot 512.0

```

UE LOG
```bash
telcomaan@oai-nr-ue-vm:~/openairinterface5g/cmake_targets/ran_build/build$ sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --ssb 516 --rfsim --rfsimulator.serveraddr 172.17.0.92 -O /etc/oai/nr-ue.conf
CMDLINE: "./nr-uesoftmodem" "-r" "106" "--numerology" "1" "--band" "78" "-C" "3619200000" "--ssb" "516" "--rfsim" "--rfsimulator.serveraddr" "172.17.0.92" "-O" "/etc/oai/nr-ue.conf" 
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
UE threads created by 6663
TYPE <CTRL-C> TO TERMINATE
[HW]   Running as client: will connect to a rfsimulator server side
Initializing random number generator, seed 10776370711420876736
[HW]   [RRU] has loaded RFSIMULATOR device.
[HW]   Trying to connect to 172.17.0.92:4043
[HW]   Connection to 172.17.0.92:4043 established
[PHY]   SSB position provided
[NR_PHY]   Starting sync detection
[PHY]   [UE thread Synch] Running Initial Synch 
[NR_PHY]   Starting cell search with center freq: 3619200000, bandwidth: 106. Scanning for 1 number of GSCN.
[NR_PHY]   Scanning GSCN: 0, with SSB offset: 516, SSB Freq: 0.000000
[PHY]   Initial sync: pbch decoded sucessfully, ssb index 0
[PHY]   pbch rx ok. rsrp:51 dB/RE, adjust_rxgain:-1 dB
[NR_PHY]   Cell Detected with GSCN: 0, SSB SC offset: 516, SSB Ref: 0.000000, PSS Corr peak: 99 dB, PSS Corr Average: 61
[PHY]   [UE0] In synch, rx_offset 491520 samples
[PHY]   [UE 0] Measured Carrier Frequency offset 5 Hz
[PHY]   Initial sync successful, PCI: 0
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_freq 3619200005 Hz, rx_freq 3619200005 Hz, tune_offset 0
[PHY]   Got synch: hw_slot_offset 32, carrier off 5 Hz, rxgain 0.000000 (DL 3619200005.000000 Hz, UL 3619200005.000000 Hz)
[PHY]   UE synchronized! decoded_frame_rx=874 UE->init_sync_frame=1 trashed_frames=104
[PHY]   Resynchronizing RX by 491520 samples
[HW]   received write reorder clear context
[NR_RRC]   SIB1 decoded
[NR_MAC]   TDD period index = 6, based on the sum of dl_UL_TransmissionPeriodicity from Pattern1 (5.000000 ms) and Pattern2 (0.000000 ms): Total = 5.000000 ms
[NR_MAC]   Set TDD configuration period to: 8 DL slots, 3 UL slots, 10 slots per period (NR_TDD_UL_DL_Pattern is 7 DL slots, 2 UL slots, 6 DL symbols, 4 UL symbols)
[NR_MAC]   Configured 1 TDD patterns (total slots: pattern1 = 10, pattern2 = 0)
[PHY]   N_TA_offset changed from 0 to 800
[MAC]   Initialization of 4-Step CBRA procedure
[NR_MAC]   PRACH scheduler: Selected RO Frame 981, Slot 19, Symbol 8, Fdm 0
[PHY]   PRACH [UE 0] in frame.slot 981.19, placing PRACH in position 2828, Msg1/MsgA-Preamble frequency start 0 (k1 0), preamble_offset 5, first_nonzero_root_idx 0, preambleIndex = 22
[PHY]   RAR-Msg2 decoded
[NR_MAC]   [UE 0][RAPROC][RA-RNTI 0113] Got BI RAR subPDU 5 ms
[NR_MAC]   [UE 0][RAPROC][RA-RNTI 0113] Got RAPID RAR subPDU
[NR_MAC]   [UE 0][RAPROC][982.7] Found RAR with the intended RAPID 22
[MAC]   received TA command 31
[NR_MAC]   [RAPROC][982.17] RA-Msg3 transmitted
[MAC]   [UE 0][984.1][RAPROC] 4-Step RA procedure succeeded. CBRA: Contention Resolution is successful.
[NR_RRC]   [UE0][RAPROC] Logical Channel DL-CCCH (SRB0), Received NR_RRCSetup
[RLC]   Added srb 1 to UE 0
[NR_RRC]   State = NR_RRC_CONNECTED
[NAS]   Generate Initial NAS Message: Registration Request
[NR_RRC]   [UE 0][RAPROC] Logical Channel UL-DCCH (SRB1), Generating RRCSetupComplete (bytes33)
[MAC]   [UE 0] Applying CellGroupConfig from gNodeB
[NAS]   [UE 0] Received NR_NAS_CONN_ESTABLISH_IND: asCause 0
[NR_MAC]   UE 0 RNTI 457a stats sfn: 0.8, cumulated bad DCI 0
    DL harq: 1/0
    Ul harq: 41/0 avg code rate 0.1, avg bit/symbol 3.0, avg per TB: (nb RBs 5.1, nb symbols 3.0)
[NAS]   [UE 0] Received NAS_DOWNLINK_DATA_IND type FGS_AUTHENTICATION_REQUEST with length 42
kausf:c9 11 d0 b0 21 5a b8 3 b9 10 33 2 4 d8 28 cf c7 56 61 21 55 74 7e 97 ef 50 6 10 c3 2e f5 bd 
kseaf:75 89 f9 3d 4e 11 d4 c 2e 7f ef ba 28 53 54 40 42 23 2d c6 17 1c 63 1d 45 fd d 14 57 40 fd d5 
kamf:ca 6 8e c5 55 9e 19 cf 77 7a fd 8f 54 7f 6d cf fe 77 ee 3f ca 9 cf ba 93 5a c6 86 2a d8 44 4e 
[NAS]   [UE 0] Received NAS_DOWNLINK_DATA_IND type FGS_SECURITY_MODE_COMMAND with length 19
knas_int: a9 9f 3c d0 43 5f 41 f9 a 59 6f b0 64 bc f0 66 
knas_enc: 2b 32 5c 28 cb 65 ef b3 20 10 8a aa b 19 9d ed 
[NAS]   Generate Initial NAS Message: Registration Request
mac c0 33 bd 69 
[NR_RRC]   Received securityModeCommand (gNB 0)
[NR_RRC]   Receiving from SRB1 (DL-DCCH), Processing securityModeCommand
[NR_RRC]   Security algorithm is set to nea0
[NR_RRC]   Integrity protection algorithm is set to nia2
[NR_RRC]   deriving kRRCenc, kRRCint from KgNB=bc e3 41 86 af bc 73 01 70 b7 9b f2 4c 7b c6 02 12 45 96 41 60 30 e6 eb 9e 05 f4 dd c0 10 12 d5 
[NR_RRC]   Receiving from SRB1 (DL-DCCH), encoding securityModeComplete, rrc_TransactionIdentifier: 0
[NR_RRC]   securityModeComplete payload: 28 00 00 00 00 00 00 00 c0 52 01 20 65 7c 00 00 
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
[NR_RRC]   5G-GUTI: AMF pointer 0, AMF Set ID 1, 5G-TMSI 3221225862 
mac a5 10 4f 15 
[NAS]   Send NAS_UPLINK_DATA_REQ message(RegistrationComplete)
mac 4b 2e 26 d0 
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
[MAC]   [UE 0] Applying CellGroupConfig from gNodeB
[NR_RRC]   RRCReconfiguration includes Measurement Configuration
[NR_RRC]   Measurement gaps not yet supported!
[NR_RRC]   rrcReconfigurationComplete Encoded 10 bits (2 bytes)
[NR_RRC]    Logical Channel UL-DCCH (SRB1), Generating RRCReconfigurationComplete (bytes 2)
[NAS]   [UE 0] Received NAS_CONN_ESTABLI_CNF: errCode 1, length 68
[NAS]   Received PDU Session Establishment Accept, UE IPv4: 10.45.0.3
[OIP]   Interface oaitun_ue1 successfully configured, IPv4 10.45.0.3, IPv6 (null)
[UTIL]   threadCreate() for ue_tun_read_0_p10: creating thread with affinity ffffffff, priority 1
[NR_MAC]   UE 0 RNTI 457a stats sfn: 128.8, cumulated bad DCI 0
    DL harq: 17/0
    Ul harq: 173/0 avg code rate 0.3, avg bit/symbol 2.2, avg per TB: (nb RBs 5.9, nb symbols 3.1)
Entering ITTI signals handler
TYPE <CTRL-C> TO TERMINATE
[NR_MAC]   UE 0 RNTI 457a stats sfn: 256.8, cumulated bad DCI 0
    DL harq: 19/0
    Ul harq: 189/0 avg code rate 0.3, avg bit/symbol 2.2, avg per TB: (nb RBs 5.9, nb symbols 3.7)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 384.8, cumulated bad DCI 0
    DL harq: 21/0
    Ul harq: 202/0 avg code rate 0.3, avg bit/symbol 2.1, avg per TB: (nb RBs 5.8, nb symbols 4.3)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 512.8, cumulated bad DCI 0
    DL harq: 23/0
    Ul harq: 289/0 avg code rate 0.3, avg bit/symbol 2.1, avg per TB: (nb RBs 6.0, nb symbols 4.2)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 640.8, cumulated bad DCI 0
    DL harq: 30/0
    Ul harq: 341/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.9, nb symbols 4.3)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 768.8, cumulated bad DCI 0
    DL harq: 34/0
    Ul harq: 403/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.6, nb symbols 4.4)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 896.8, cumulated bad DCI 0
    DL harq: 36/0
    Ul harq: 489/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.5, nb symbols 4.3)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 0.8, cumulated bad DCI 0
    DL harq: 37/0
    Ul harq: 502/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.5, nb symbols 4.5)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 128.8, cumulated bad DCI 0
    DL harq: 38/0
    Ul harq: 514/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.5, nb symbols 4.7)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 256.8, cumulated bad DCI 0
    DL harq: 40/0
    Ul harq: 527/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.4, nb symbols 4.9)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 384.8, cumulated bad DCI 0
    DL harq: 41/0
    Ul harq: 540/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.4, nb symbols 5.1)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 512.8, cumulated bad DCI 0
    DL harq: 42/0
    Ul harq: 553/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.4, nb symbols 5.3)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 640.8, cumulated bad DCI 0
    DL harq: 45/0
    Ul harq: 632/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.3, nb symbols 5.2)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 768.8, cumulated bad DCI 0
    DL harq: 46/0
    Ul harq: 645/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.3, nb symbols 5.3)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 896.8, cumulated bad DCI 0
    DL harq: 47/0
    Ul harq: 658/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.3, nb symbols 5.5)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 0.8, cumulated bad DCI 0
    DL harq: 48/0
    Ul harq: 671/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.3, nb symbols 5.6)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 128.8, cumulated bad DCI 0
    DL harq: 50/0
    Ul harq: 684/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.2, nb symbols 5.7)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 256.8, cumulated bad DCI 0
    DL harq: 51/0
    Ul harq: 696/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.2, nb symbols 5.9)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 384.8, cumulated bad DCI 0
    DL harq: 52/0
    Ul harq: 709/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.2, nb symbols 6.0)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 512.8, cumulated bad DCI 0
    DL harq: 54/0
    Ul harq: 722/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.2, nb symbols 6.1)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 640.8, cumulated bad DCI 0
    DL harq: 55/0
    Ul harq: 735/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.2, nb symbols 6.3)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 768.8, cumulated bad DCI 0
    DL harq: 56/0
    Ul harq: 748/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.1, nb symbols 6.4)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 896.8, cumulated bad DCI 0
    DL harq: 57/0
    Ul harq: 760/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.1, nb symbols 6.5)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 0.8, cumulated bad DCI 0
    DL harq: 59/0
    Ul harq: 773/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.1, nb symbols 6.6)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 128.8, cumulated bad DCI 0
    DL harq: 60/0
    Ul harq: 786/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.1, nb symbols 6.7)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 256.8, cumulated bad DCI 0
    DL harq: 62/0
    Ul harq: 867/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.1, nb symbols 6.4)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 384.8, cumulated bad DCI 0
    DL harq: 63/0
    Ul harq: 879/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.1, nb symbols 6.5)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 512.8, cumulated bad DCI 0
    DL harq: 65/0
    Ul harq: 892/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.1, nb symbols 6.6)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 640.8, cumulated bad DCI 0
    DL harq: 66/0
    Ul harq: 905/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.1, nb symbols 6.7)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 768.8, cumulated bad DCI 0
    DL harq: 67/0
    Ul harq: 918/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.0, nb symbols 6.8)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 896.8, cumulated bad DCI 0
    DL harq: 69/0
    Ul harq: 931/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.0, nb symbols 6.9)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 0.8, cumulated bad DCI 0
    DL harq: 70/0
    Ul harq: 943/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.0, nb symbols 7.0)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 128.8, cumulated bad DCI 0
    DL harq: 71/0
    Ul harq: 956/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.0, nb symbols 7.1)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 256.8, cumulated bad DCI 0
    DL harq: 72/0
    Ul harq: 969/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.0, nb symbols 7.1)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 384.8, cumulated bad DCI 0
    DL harq: 74/0
    Ul harq: 982/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.0, nb symbols 7.2)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 512.8, cumulated bad DCI 0
    DL harq: 75/0
    Ul harq: 995/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 6.0, nb symbols 7.3)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 640.8, cumulated bad DCI 0
    DL harq: 76/0
    Ul harq: 1007/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 5.9, nb symbols 7.4)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 768.8, cumulated bad DCI 0
    DL harq: 78/0
    Ul harq: 1020/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 5.9, nb symbols 7.4)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 896.8, cumulated bad DCI 0
    DL harq: 79/0
    Ul harq: 1033/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 5.9, nb symbols 7.5)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 0.8, cumulated bad DCI 0
    DL harq: 80/0
    Ul harq: 1046/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 5.9, nb symbols 7.6)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 128.8, cumulated bad DCI 0
    DL harq: 81/0
    Ul harq: 1059/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 5.9, nb symbols 7.6)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 256.8, cumulated bad DCI 0
    DL harq: 83/0
    Ul harq: 1071/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 5.9, nb symbols 7.7)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 384.8, cumulated bad DCI 0
    DL harq: 84/0
    Ul harq: 1084/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 5.9, nb symbols 7.8)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 512.8, cumulated bad DCI 0
    DL harq: 85/0
    Ul harq: 1097/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 5.9, nb symbols 7.8)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 640.8, cumulated bad DCI 0
    DL harq: 87/0
    Ul harq: 1110/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 5.9, nb symbols 7.9)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 768.8, cumulated bad DCI 0
    DL harq: 88/0
    Ul harq: 1123/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 5.8, nb symbols 7.9)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 896.8, cumulated bad DCI 0
    DL harq: 89/0
    Ul harq: 1135/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 5.8, nb symbols 8.0)
[NR_MAC]   UE 0 RNTI 457a stats sfn: 0.8, cumulated bad DCI 0
    DL harq: 91/0
    Ul harq: 1212/0 avg code rate 0.2, avg bit/symbol 2.1, avg per TB: (nb RBs 5.9, nb symbols 7.8)
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
[NR_MAC]   Configuring CRNTI f401
[NR_MAC]   TDD period index = 6, based on the sum of dl_UL_TransmissionPeriodicity from Pattern1 (5.000000 ms) and Pattern2 (0.000000 ms): Total = 5.000000 ms
[NR_MAC]   Set TDD configuration period to: 8 DL slots, 3 UL slots, 10 slots per period (NR_TDD_UL_DL_Pattern is 7 DL slots, 2 UL slots, 6 DL symbols, 4 UL symbols)
[NR_MAC]   Configured 1 TDD patterns (total slots: pattern1 = 10, pattern2 = 0)
[PHY]   SSB position provided
[NR_PHY]   Starting re-sync detection for target Nid_cell 1
[PHY]   [UE thread Synch] Running Initial Synch 
[NR_PHY]   Starting cell search with center freq: 3619200000, bandwidth: 106. Scanning for 1 number of GSCN.
[NR_PHY]   Scanning GSCN: 0, with SSB offset: 516, SSB Freq: 0.000000
[PHY]   synch Failed: 
[PHY]   SSB position provided
[NR_PHY]   Starting re-sync detection for target Nid_cell 1
[PHY]   [UE thread Synch] Running Initial Synch 
[NR_PHY]   Starting cell search with center freq: 3619200000, bandwidth: 106. Scanning for 1 number of GSCN.
[NR_PHY]   Scanning GSCN: 0, with SSB offset: 516, SSB Freq: 0.000000
[PHY]   synch Failed: 
[PHY]   SSB position provided
[NR_PHY]   Starting re-sync detection for target Nid_cell 1
[PHY]   [UE thread Synch] Running Initial Synch 
[NR_PHY]   Starting cell search with center freq: 3619200000, bandwidth: 106. Scanning for 1 number of GSCN.
[NR_PHY]   Scanning GSCN: 0, with SSB offset: 516, SSB Freq: 0.000000
[PHY]   synch Failed: 
[PHY]   SSB position provided
[NR_PHY]   Starting re-sync detection for target Nid_cell 1
[PHY]   [UE thread Synch] Running Initial Synch 
[NR_PHY]   Starting cell search with center freq: 3619200000, bandwidth: 106. Scanning for 1 number of GSCN.
[NR_PHY]   Scanning GSCN: 0, with SSB offset: 516, SSB Freq: 0.000000
[PHY]   synch Failed: 
[PHY]   SSB position provided
[NR_PHY]   Starting re-sync detection for target Nid_cell 1
[PHY]   [UE thread Synch] Running Initial Synch 
[NR_PHY]   Starting cell search with center freq: 3619200000, bandwidth: 106. Scanning for 1 number of GSCN.
[NR_PHY]   Scanning GSCN: 0, with SSB offset: 516, SSB Freq: 0.000000
[PHY]   synch Failed: 
[PHY]   SSB position provided
[NR_PHY]   Starting re-sync detection for target Nid_cell 1
[PHY]   [UE thread Synch] Running Initial Synch 
[NR_PHY]   Starting cell search with center freq: 3619200000, bandwidth: 106. Scanning for 1 number of GSCN.
[NR_PHY]   Scanning GSCN: 0, with SSB offset: 516, SSB Freq: 0.000000
[PHY]   synch Failed: 
[PHY]   SSB position provided
[NR_PHY]   Starting re-sync detection for target Nid_cell 1
[PHY]   [UE thread Synch] Running Initial Synch 
[NR_PHY]   Starting cell search with center freq: 3619200000, bandwidth: 106. Scanning for 1 number of GSCN.
[NR_PHY]   Scanning GSCN: 0, with SSB offset: 516, SSB Freq: 0.000000
[PHY]   synch Failed: 
[PHY]   SSB position provided
[NR_PHY]   Starting re-sync detection for target Nid_cell 1
[PHY]   [UE thread Synch] Running Initial Synch 
[NR_PHY]   Starting cell search with center freq: 3619200000, bandwidth: 106. Scanning for 1 number of GSCN.
[NR_PHY]   Scanning GSCN: 0, with SSB offset: 516, SSB Freq: 0.000000
[PHY]   synch Failed: 
[PHY]   SSB position provided
[NR_PHY]   Starting re-sync detection for target Nid_cell 1
[PHY]   [UE thread Synch] Running Initial Synch 
[NR_PHY]   Starting cell search with center freq: 3619200000, bandwidth: 106. Scanning for 1 number of GSCN.
[NR_PHY]   Scanning GSCN: 0, with SSB offset: 516, SSB Freq: 0.000000
[PHY]   synch Failed: 
[PHY]   SSB position provided
[NR_PHY]   Starting re-sync detection for target Nid_cell 1
[PHY]   [UE thread Synch] Running Initial Synch 
[NR_PHY]   Starting cell search with center freq: 3619200000, bandwidth: 106. Scanning for 1 number of GSCN.
[NR_PHY]   Scanning GSCN: 0, with SSB offset: 516, SSB Freq: 0.000000
```

DU-HO LOG
```bash
telcomaan@oai-du-ho-vm:~/openairinterface5g/cmake_targets/ran_build/build$ sudo -E ./nr-softmodem -O /etc/oai/oai-du-ho.conf --rfsim --sa --telnetsrv --telnetsrv.s
hrmod ci
CMDLINE: "./nr-softmodem" "-O" "/etc/oai/oai-du-ho.conf" "--rfsim" "--sa" "--telnetsrv" "--telnetsrv.shrmod" "ci" 
[CONFIG] function config_libconfig_init returned 0
[LOADER] library libtelnetsrv_gnb.so is not loaded: libtelnetsrv_gnb.so: cannot open shared object file: No such file or directory
[TELNETSRV] Telnet server: module 0 = telnet added to shell
[TELNETSRV] Telnet server: module 1 = softmodem added to shell
[TELNETSRV] couldn't find add_phy_cmds for module phy 
[TELNETSRV] Telnet server: module 2 = loader added to shell
[TELNETSRV] Telnet server: module 3 = measur added to shell
policy set to other, priority 0
Error 3: No such process trying to get nice value of thread 2323642048 
[TELNETSRV] 
Initializing telnet server...
[TELNETSRV] Telnet server: module 4 = ci added to shell
[UTIL]   running in SA mode (no --phy-test, --do-ra, --nsa option present)
[OPT]   OPT disabled
[HW]   Version: Branch: develop Abrev. Hash: b763692792 Date: Fri Aug 1 04:40:24 2025 +0000
[GNB_APP]   Initialized RAN Context: RC.nb_nr_inst = 1, RC.nb_nr_macrlc_inst = 1, RC.nb_nr_L1_inst = 1, RC.nb_RU = 1, RC.nb_nr_CC[0] = 1
[NR_PHY]   Initializing gNB RAN context: RC.nb_nr_L1_inst = 1 
[NR_PHY]   Registered with MAC interface module (0x6127483b24e0)
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
[UTIL]   threadCreate() for TASK_GTPV1_U: creating thread with affinity ffffffff, priority 50
[UTIL]   threadCreate() for TASK_DU_F1: creating thread with affinity ffffffff, priority 50
[UTIL]   threadCreate() for time source iq samples: creating thread with affinity ffffffff, priority 2
[F1AP]   F1-C DU IPaddr 172.17.0.94, connect to F1-C CU 172.17.0.93, binding GTP to 172.17.0.94
[F1AP]   Starting F1AP at DU
[GTPU]   Initializing UDP for local address 172.17.0.94 with port 2153
[GTPU]   Created gtpu instance id: 96
[UTIL]   time manager configuration: [time source: iq_samples] [mode: standalone] [server IP: 127.0.0.1} [server port: 7374] (server IP/port not used)
[MAC]   received F1 Setup Response from CU oai-cu-cp
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
[HW]   Running as server waiting opposite rfsimulators to connect
Initializing random number generator, seed 16916976453880010719
[TELNETSRV] Telnet server: module 5 = rfsimu added to shell
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
[PHY]   RU 0 rf device ready
[PHY]   RU 0 RF started cpu_meas_enabled 0
[HW]   No connected device, generating void samples...
[PHY]   Command line parameters for OAI UE: -C 3619200000 -r 106 --numerology 1 --ssb 516 
[NR_MAC]   Frame.Slot 128.0

[NR_MAC]   Frame.Slot 256.0

[NR_MAC]   Frame.Slot 384.0

[NR_MAC]   Frame.Slot 512.0

[NR_MAC]   Frame.Slot 640.0

[NR_MAC]   Frame.Slot 768.0

[NR_MAC]   Added new CFRA process for UE RNTI f401 with initial CellGroup
[RLC]   Activated srb0 for UE 62465
[RLC]   Added srb 1 to UE 62465
[RLC]   Added srb 2 to UE 62465
[RLC]   Added drb 1 to UE 62465
[RLC]   Added DRB to UE 62465
[GTPU]   [96] Created tunnel for UE ID 62465, teid for incoming: 691a163e, teid for outgoing 9ec42d5a to remote IPv4: 172.17.0.93, IPv6 ::
[NR_MAC]   Frame.Slot 896.0

[NR_MAC]   Frame.Slot 0.0

[NR_MAC]   Frame.Slot 128.0

[NR_MAC]   Frame.Slot 256.0

```

- [[F1 Handover guide on GCP setup]]
- [[F1 HANDOVER Revisited]]
- [[terraform code for this]]
- [[ansible code for automating this setup]]
