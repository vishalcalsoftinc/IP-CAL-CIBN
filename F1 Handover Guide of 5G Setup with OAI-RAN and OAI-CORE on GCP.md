## **F1 Handover Guide of 5G Setup with OAI-RAN and OAI-CORE on GCP**

This guide provides a complete, step-by-step walkthrough for deploying a virtualized 5G Standalone (SA) network on Google Cloud Platform (GCP). It uses OpenAirInterface (OAI) for both the Radio Access Network (RAN) and the 5G Core (5GC).

The final architecture will consist of a consolidated OAI CU-CP/CU-UP, two OAI DUs, a simulated OAI UE, and the OAI 5G Core network functions, all configured to demonstrate a successful F1 handover between the two DUs.

### Part 1: GCP Environment and Cost Management

#### 1.1. Leveraging the GCP Free Trial
Google Cloud offers a 90-day, $300 free trial for new users, which is more than sufficient to cover all costs for this deployment.

*   **Machine Types**: We will use a mix of `e2-medium` (for OAI-5GC) and `n2-standard-4` VMs. The OAI RAN components require the N2 series for critical CPU instruction set support (AVX-512).
*   **Cost Estimation**: With the $300 credit, the cost for this entire setup during the trial period will be **$0**.
*   **Important: Cleanup!** To avoid any charges after the trial ends, you **must** shut down and delete your created resources as outlined in the final section of this guide.

### Part 2: GCP Project and Network Setup

#### 2.1. Initial Project Setup
1.  Navigate to the [Google Cloud Console](https://console.cloud.google.com/).
2.  Create a new GCP project. Give it a memorable name like `oai-5g-handover-proj`.
3.  Ensure that billing is enabled for the project by linking your free trial billing account.

#### 2.2. VPC Network and Firewall Configuration
We will create an isolated Virtual Private Cloud (VPC) for our 5G components to communicate securely and with predictable IP addresses.

1.  **Create a VPC**:
    a. Navigate to **VPC network > VPC networks**.
    b. Click **CREATE VPC NETWORK**.
    c. **Name**: `oai-5g-vpc`
    d. **Subnet creation mode**: Custom
    e. **New subnet**:
        i. **Name**: `oai-subnet`
        ii. **Region**: `us-central1` (or another cost-effective US region)
        iii. **IP address range**: `172.17.0.0/24`
    f. Click **Create**.
    > **Why**: A custom subnet gives us full control over the IP addressing scheme. This is critical for our 5G lab, as we need to assign predictable, static IP addresses to our CU, DUs, and 5GC for their configuration files to work correctly.

2.  **Configure Firewall Rules**:
    a. Navigate to **VPC network > Firewall**.
    b. **Rule 1: Allow SSH (for management)**
        i. Click **CREATE FIREWALL RULE**.
        ii. **Name**: `allow-ssh`
        iii. **Network**: `oai-5g-vpc`
        iv. **Targets**: All instances in the network
        v. **Source IPv4 ranges**: `0.0.0.0/0` (allows SSH from any IP)
        vi. **Protocols and ports**: Specified protocols and ports > `tcp:22`
        vii. Click **Create**.
        > **Why**: This rule opens the standard SSH port, allowing us to connect to our VMs for installation and configuration. Access is still secured by mandatory SSH keys.
    c. **Rule 2: Allow Internal Communication**
        i. Click **CREATE FIREWALL RULE**.
        ii. **Name**: `allow-internal-all`
        iii. **Network**: `oai-5g-vpc`
        iv. **Targets**: All instances in the network
        v. **Source IPv4 ranges**: `172.17.0.0/24` (the range of our subnet)
        vi. **Protocols and ports**: Allow all
        vii. Click **Create**.
        > **Why**: This is the most critical rule for the 5G network itself. The CU, DUs, and 5GC communicate using various protocols (F1AP, NGAP, GTP-U, etc.). This single rule simplifies our setup by stating: "Any VM inside our private subnet is trusted and allowed to communicate freely with any other VM in that same subnet."

### Part 3: Provisioning Virtual Machines

Create five VMs with static internal IPs.

1.  **Reserve Static Internal IPs**:
    a. Navigate to **VPC network > IP addresses** and click **RESERVE INTERNAL STATIC IP ADDRESS** five times for the following IPs:
    *   `172.17.0.91` (for `oai-nr-ue-vm`)
    *   `172.17.0.92` (for `oai-du0-vm`)
    *   `172.17.0.93` (for `oai-cu-vm`)
    *   `172.17.0.94` (for `oai-du1-vm`)
    *   `172.17.0.96` (for `oai-5gc-vm`)

2.  **Create the Virtual Machines**:
    a. Navigate to **Compute Engine > VM instances** and click **CREATE INSTANCE** for each VM.

    b. **VM Creation: OAI 5G Core (`oai-5gc-vm`)**
        i. **Name**: `oai-5gc-vm`
        ii. **Machine type**: `e2-medium`
        iii. **Boot disk**: Ubuntu 22.04 LTS, 30 GB
        iv. Under **Advanced options > Networking**:
            1.  **Network**: `oai-5g-vpc`
            2.  **Primary internal IP**: The reserved `172.17.0.96`.
            3.  Check **Enable IP forwarding**.

    c. **VM Creation: All OAI RAN VMs**
        i. Create the four OAI RAN VMs (`oai-cu-vm`, `oai-du1-vm`, `oai-du0-vm`, `oai-nr-ue-vm`) with the following settings:
        ii. **Series**: N2 (Critical for AVX-512 support)
        iii. **Machine type**: `n2-standard-4` (4 vCPU, 16 GB memory)
        iv. **Boot disk**: Ubuntu 22.04 LTS, 30 GB
        v. Under **Advanced options > Networking**:
            4.  **Network**: `oai-5g-vpc`
            5.  **Primary internal IP**: Select the corresponding reserved IP from the table below.

#### VM Roles and IPs:
| Final Role | GCP VM Name | Reserved Static IP |
| :--- | :--- | :--- |
| OAI 5G Core | `oai-5gc-vm` | `172.17.0.96` |
| OAI CU-CP & CU-UP | `oai-cu-vm` | `172.17.0.93` |
| OAI Source DU (DU0) | `oai-du0-vm` | `172.17.0.92` |
| OAI Target DU (DU1) | `oai-du1-vm` | `172.17.0.94` |
| OAI NR-UE (RF Server) | `oai-nr-ue-vm` | `172.17.0.91` |

### Part 4: Setting up Secure SSH Access

Follow these steps to securely connect to your VMs using SSH keys, which is more secure than passwords.

1.  **Create SSH Key Pairs**: On your local computer's terminal, run:
    ```bash
    ssh-keygen -t rsa -b 4096 -C "your_username"
    ```
    Press Enter to accept the default file locations and provide a strong passphrase when prompted.

2.  **Add the Public Key on GCP**:
    a. Display your public key on your local terminal:
        ```bash
        cat ~/.ssh/id_rsa.pub
        ```
    b. Copy the entire output.
    c. In the GCP Console, navigate to **Compute Engine > Metadata**.
    d. Select the **SSH Keys** tab and click **ADD SSH KEY**.
    e. Paste your copied public key into the box and click **Save**. This grants access to all VMs in the project.

3.  **Set Up Remote SSH in VS Code**:
    a. Install the **Remote - SSH** extension by Microsoft in VS Code.
    b. Open the command palette (`Ctrl+Shift+P`), type `Remote-SSH: Open SSH Configuration File...`, and choose the `config` file in your user's `.ssh` directory.
    c. Add entries for each VM. Replace `<EXTERNAL_IP_...>` with the `External IP` of each VM from the GCP console.

    ```
    # OAI 5G Core VM
    Host gcp-oai-5gc-vm
        HostName <EXTERNAL_IP_OAI_5GC_VM>
        User your_username
        IdentityFile ~/.ssh/id_rsa

    # OAI CU VM
    Host gcp-oai-cu-vm
        HostName <EXTERNAL_IP_OAI_CU_VM>
        User your_username
        IdentityFile ~/.ssh/id_rsa

    # ... add similar entries for du0, du1, and ue VMs ...
    ```

### Part 5: OAI 5G Core Network Installation & Configuration

SSH into the `oai-5gc-vm` (`gcp-oai-5gc-vm` in VS Code) and follow these steps.

1.  **Install Prerequisites**:
    ```bash
    sudo apt update
    sudo apt install -y git docker.io docker-compose
    sudo systemctl start docker
    sudo systemctl enable docker
    # Add your user to the docker group to run docker commands without sudo
    sudo usermod -aG docker $USER
    # You will need to log out and log back in for this change to take effect
    ```

2.  **Clone OAI 5G Core Repository**:
    ```bash
    git clone https://gitlab.eurecom.fr/oai/cn5g/oai-cn5g-fed.git
    cd oai-cn5g-fed
    ```

3.  **Configure the OAI Core**:
    a. Edit the environment file to match our GCP setup.
        ```bash
        nano docker-compose/core-network.env
        ```
    b. Update the `OAI_HOSTNAME` and set the PLMN. The file should look like this:
```env
        OAI_HOSTNAME=oai-5gc-vm
        # YOUR PUBLIC DOMAIN. If you don't have one, just use a fake one.
        DOMAIN=openairinterface.org

        # MME Information
        MME_FQDN="mme.${DOMAIN}"
        MME_REALM="${DOMAIN}"
        MCC=999
        MNC=70
        MNC_LENGTH=2
        TAC=1
```

	c. Configure Subscriber Information:
```bash
        nano docker-compose/conf/db_init.sql
```

        Find the `INSERT INTO users` section and ensure a subscriber entry exists that matches the one we will use for the UE.
```sql
        -- IMSI, KEY, OPC for the UE
        INSERT INTO users (imsi, msisdn, security_key, opc, an_op, an_sqn, an_rand, an_auts, ue_ambr_ul, ue_ambr_dl, default_qos_profile_id, enabled) VALUES ('999700000000001', '33612345678', unhex('465B5CE8B199B49FAA5F0A2EE238A6BC'), unhex('E8ED289DEBA952E4283B54E88E6183CA'), NULL, '000000000000', '00000000000000000000000000000000', NULL, 100000000, 100000000, 1, 1);
```

4.  **Start the OAI 5G Core**:
    From the `oai-cn5g-fed/docker-compose` directory, run:
    ```bash
    python3 core-network.py --type start-basic --scenario 1
    ```
    This script will pull the necessary Docker images and start all the core network components. Verify they are running with `docker ps -a`.

5.  **Enable NAT for UE Internet Access**:
    These commands turn the `oai-5gc-vm` into a router for the UE.
    ```bash
    sudo sysctl -w net.ipv4.ip_forward=1
    sudo iptables -t nat -A POSTROUTING -o ens4 -j MASQUERADE
    sudo iptables -I FORWARD 1 -j ACCEPT
    ```

### Part 6: OAI RAN Installation & Configuration

On **each of the four OAI RAN VMs** (`oai-cu-vm`, `oai-du0-vm`, `oai-du1-vm`, `oai-nr-ue-vm`), perform the following steps.

1.  **Install OAI RAN**:
    ```bash
    # Run on all four OAI RAN VMs
    sudo apt update && sudo apt install -y git
    cd ~
    git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git
    cd openairinterface5g
    source oaienv
    cd cmake_targets
    # This command builds all components with telnet support for handover triggering
    ./build_oai --ninja --nrUE --gNB --build-lib telnetsrv
    ```

2.  **Create OAI RAN Configuration Files**:
    Create the files in `/etc/oai/` on their respective VMs.

    a. **On `oai-cu-vm` - Create `/etc/oai/oai-cu.conf`**:
       ```ini
       Active_gNBs = ("gNB-Eurecom-CU");
       Asn1_verbosity = "none";
       gNBs = ({
         gNB_ID = 0xe00;
         gNB_name = "gNB-Eurecom-CU";
         tracking_area_code = 1;
         plmn_list = ({ mcc = 999; mnc = 70; mnc_length = 2; snssaiList = ({ sst = 1, sd = 0x000001}) });
         
         // OAI 5G Core AMF IP
         amf_ip_address = ({ ipv4 = "172.17.0.96"; });

         // DU0 (Source DU) IP
         remote_s_address = "172.17.0.92";
         local_s_address = "172.17.0.93";
       });
       NETWORK_INTERFACES : {
         GNB_IPV4_ADDRESS_FOR_NG_AMF = "172.17.0.93/24";
         GNB_IPV4_ADDRESS_FOR_NGU = "172.17.0.93/24";
       };
       ```

    b. **On `oai-du0-vm` (Source DU) - Create `/etc/oai/oai-du0.conf`**:
       ```ini
       // This DU has physCellId = 0
       physCellId = 0;
       
       // RF Simulator points to the UE VM as the server
       rfsimulator: {
         serveraddr = "172.17.0.91";
         serverport = 4043;
       }

       // F1-C connection to CU
       local_n_address = "172.17.0.92";
       remote_n_address = "172.17.0.93";
       // -- Other DU parameters like frequency bands etc. --
       ```

    c. **On `oai-du1-vm` (Target DU) - Create `/etc/oai/oai-du1.conf`**:
       ```ini
       // This DU has physCellId = 1
       physCellId = 1;

       // RF Simulator also points to the UE VM as the server
       rfsimulator: {
         serveraddr = "172.17.0.91";
         serverport = 4043;
       }
       
       // F1-C connection to CU
       local_n_address = "172.17.0.94";
       remote_n_address = "172.17.0.93";
       // -- Other DU parameters --
       ```

    d. **On `oai-nr-ue-vm` - Create `/etc/oai/nr-ue.conf`**:
       ```ini
       uicc0 = {
         imsi = "999700000000001";
         key = "465B5CE8B199B49FAA5F0A2EE238A6BC";
         opc = "E8ED289DEBA952E4283B54E88E6183CA";
         dnn = "internet";
         nssai_sst = 1;
         nssai_sd = 0x000001;
       };
       
       // Configure UE as the RF Simulator Server
       rfsimulator: {
         serveraddr = "server";
         serverport = 4043;
       };
```

#### 6.1 The Inverted RF Simulator Model Explained
For a handover to work, the UE must be able to "hear" signals from both the source DU (DU0) and the target DU (DU1) simultaneously. The OAI RF simulator operates on a strict **one-server-to-many-clients** model. If the DUs were servers, the UE client could only connect to one at a time, making handover impossible.

By configuring the **UE as the server**, we create a single, centralized simulation environment. Both DUs act as clients that feed their signals into this environment. This "hub-and-spoke" architecture allows the UE to see both signals and seamlessly switch between them when commanded.

### Part 7: Starting the Network & Executing the Handover

This startup sequence is critical and must be followed exactly. Use a separate SSH terminal for each command.

1.  **Start OAI 5G Core Services** (on `oai-5gc-vm`):
    ```bash
    # In oai-cn5g-fed/docker-compose directory
    python3 core-network.py --type start-basic --scenario 1
    # Verify with 'docker ps -a'
    ```

2.  **Enable NAT** (on `oai-5gc-vm`):
    ```bash
    sudo sysctl -w net.ipv4.ip_forward=1
    sudo iptables -t nat -A POSTROUTING -o ens4 -j MASQUERADE
    sudo iptables -I FORWARD 1 -j ACCEPT
    ```

3.  **Start the CU** (on `oai-cu-vm`):
    ```bash
    cd ~/openairinterface5g/cmake_targets/ran_build/build
    sudo -E ./nr-softmodem -O /etc/oai/oai-cu.conf --sa --telnetsrv
    ```

4.  **Start the NR UE as the RF Server** (on `oai-nr-ue-vm`):
    ```bash
    cd ~/openairinterface5g/cmake_targets/ran_build/build
    sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --rfsim -O /etc/oai/nr-ue.conf
    ```

5.  **Start the Source DU (DU0) as a Client** (on `oai-du0-vm`):
    The DU will connect to the UE's RF server, and the UE will attach to the network.
    ```bash
    cd ~/openairinterface5g/cmake_targets/ran_build/build
    sudo ./nr-softmodem -O /etc/oai/oai-du0.conf --rfsim --sa
    ```

6.  **Start the Target DU (DU1) as a Client** (on `oai-du1-vm`):
    This DU also connects to the UE's RF server. The CU-CP log will show a second F1 setup.
    ```bash
    cd ~/openairinterface5g/cmake_targets/ran_build/build
    sudo ./nr-softmodem -O /etc/oai/oai-du1.conf --rfsim --sa
    ```

7.  **Trigger the F1 Handover**:
    Run this command from **any machine** that can reach the CU's internal IP.
    ```bash
    echo ci trigger_f1_ho | nc 172.17.0.93 9090 && echo "Handover Triggered"
    ```

### Part 8: Verifying the Successful Handover

1.  **Check UE Logs**: The UE log (`oai-nr-ue-vm`) will show it receiving `RRCReconfiguration`, successfully synchronizing to the new cell (PCI 1), and sending `RRCReconfigurationComplete`.
2.  **Check CU Logs**: Will show the "Handover triggered" message.
3.  **Check AMF Logs**: The AMF logs (`docker logs -f oai-amf`) will show **no new activity**. This is correct and expected, as an F1 handover is contained within the RAN and is transparent to the core network.
4.  **Test Data Connectivity**: The definitive test. On the `oai-nr-ue-vm`, a continuous ping should survive the handover.
    ```bash
    # On oai-nr-ue-vm, before, during, and after triggering the handover
    ping -I oaitun_ue1 8.8.8.8
    ```
    Successful, uninterrupted replies confirm a successful data plane switch.

### Part 9: CRITICAL Cleanup

To avoid ANY charges after you are finished, you **MUST** tear down your environment.

1.  **Delete the VMs**:
    a. Go to **Compute Engine > VM instances**.
    b. Select all five VMs and click **DELETE**.
2.  **Delete VPC, Firewall Rules, and Static IPs**:
    a. Go to **VPC Network > Firewall** and delete the firewall rules.
    b. Go to **VPC Network > IP Addresses** and release the static IPs.
    c. Go to **VPC Network > VPC Networks** and delete `oai-5g-vpc`. This will also delete the subnet.