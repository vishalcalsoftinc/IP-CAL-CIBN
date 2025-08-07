
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

CUUP LO

- [[F1 Handover guide on GCP setup]]
- [[F1 HANDOVER Revisited]]
- [[terraform code for this]]
- [[ansible code for automating this setup]]
