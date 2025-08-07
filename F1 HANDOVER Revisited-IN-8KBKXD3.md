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

1.  Create the file `~/oai-cuup.conf` and copy the content from previous setup.
#### **2.3. First DU (DU0) Configuration (on `oai-du-vm`)**

Create a new handover-specific configuration for the first DU. The `remote_n_address` now points to the consolidated CU VM (`172.17.0.93`).

1.  Create the file `~/oai-du.conf`:
    ```bash
    # On oai-du-vm
    nano /etc/oai/oai-du.conf
    ```

2.  Paste the following content. This DU uses **PCI 0**.
 
```conf


```

#### **2.4. Second DU (DU1) Configuration (on `oai-cuup-vm`)**

Create the configuration for the second DU on the repurposed `oai-cuup-vm`.

1.  Create the file `~/oai-du-ho.conf`:
    ```bash
    # On oai-cuup-vm
    nano ~/oai-du-ho.conf
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
sudo -E ./nr-softmodem -O /etc/oai/oai-du.conf --rfsim --sa --telnetsrv --telnetsrv.shrmod ci
```
The CU-CP log will confirm the F1 SETUP from DU ID `0xe00`.

**5. Start the NR UE**
On `oai-nr-ue-vm`, start the UE. It will act as the RF simulator server.

```bash
# On oai-nr-ue-vm
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --rfsim --sa
```
Wait for the UE to attach to DU0 (PCI 0). You will see logs for RRC connection, Registration Accept, and PDU session establishment.

**6. Start the Second DU (DU1)**
On `oai-cuup-vm` (the repurposed VM), start the second DU.

```bash
# On oai-cuup-vm
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo ./nr-softmodem -O ~/oai-du1-ho.conf --rfsim --sa
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