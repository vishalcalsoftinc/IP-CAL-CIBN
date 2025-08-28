Got it 👍  
Let’s build a **step-by-step guide** of what you did until the point where **two UPFs were successfully running and registered with SMF**, with **all commands, configs, and changes**.

---

# 📖 Guide: Multi-UPF Setup in Open5GS with OAI RAN (GCP Deployment)

This guide documents how we configured and ran **two UPFs** (UPF1 and UPF2) with Open5GS SMF on GCP, so they both register properly with SMF and can later be used for TAC-based F1 handover.

---

## 1️⃣ Base Setup

- Environment:
    
    - Open5GS core (AMF, SMF, UPF1, MongoDB) on **VM1**
        
    - OAI CU-CP/CU-UP/DU0/DU1 on separate VMs
        
    - UPF2 on the same VM as Open5GS (to avoid PFCP behind NAT issues)
        
- Network:
    
    - Core VM (Open5GS): `172.17.0.95`
        
    - CU-CP VM: `172.17.0.93`
        
    - DU0 VM: `172.17.0.92`
        
    - DU1 VM: `172.17.0.94`
        

---

## 2️⃣ Preparing UPF1 (Default UPF)

UPF1 continues to use the default `ogstun` interface.

### Config `/etc/open5gs/upf.yaml`

```yaml
upf:
  pfcp:
    server:
      - address: 127.0.0.7
    client:
      smf:
        - address: 127.0.0.4
  gtpu:
    server:
      - address: 172.17.0.95
        port: 2152
  session:
    - subnet: 10.45.0.0/16
      gateway: 10.45.0.1
      dev: ogstun
  metrics:
    server:
      - address: 172.17.0.95
        port: 9091
```

### Tunnel Setup

```bash
# Delete and recreate ogstun
sudo ip link del ogstun 2>/dev/null
sudo ip tuntap add name ogstun mode tun
sudo ip addr add 10.45.0.1/16 dev ogstun
sudo ip link set ogstun up
```

### Run UPF1

```bash
sudo open5gs-upfd -c /etc/open5gs/upf.yaml
```

---

## 3️⃣ Preparing UPF2 (Second UPF)

UPF2 needs its **own tunnel device** (`ogstun2`), different **subnet**, and **different PFCP address** so it doesn’t conflict with UPF1.

### Config `/etc/open5gs/upf2.yaml`

```yaml
upf:
  pfcp:
    server:
      - address: 127.0.0.8     # Unique PFCP loopback
    client:
      smf:
        - address: 127.0.0.4   # SMF PFCP
  gtpu:
    server:
      - address: 172.17.0.95
        port: 2153             # Different GTP-U port
  session:
    - subnet: 10.46.0.0/16
      gateway: 10.46.0.1
      dev: ogstun2             # Unique tunnel device
  metrics:
    server:
      - address: 172.17.0.95
        port: 9092
```

### Tunnel Setup for UPF2

```bash
# Delete and recreate ogstun2
sudo ip link del ogstun2 2>/dev/null
sudo ip tuntap add name ogstun2 mode tun
sudo ip addr add 10.46.0.1/16 dev ogstun2
sudo ip link set ogstun2 up
```

### Run UPF2

```bash
sudo open5gs-upfd -c /etc/open5gs/upf2.yaml
```

---

## 4️⃣ Updating SMF Configuration

SMF must know about **both UPFs**.  
We added TAC-based selection, so TAC=1 goes to UPF1, TAC=2 goes to UPF2.

### Config `/etc/open5gs/smf.yaml`

```yaml
smf:
  pfcp:
    server:
      - address: 127.0.0.4
  gtpc:
    server:
      - address: 127.0.0.4
  pfcp:
    client:
      upf:
        - address: 127.0.0.7    # UPF1
          dnn: internet
          tac: 1
        - address: 127.0.0.8    # UPF2
          dnn: internet
          tac: 2
```

---

## 5️⃣ Verification

### Check UPF Processes

```bash
ps -aux | grep upfd
```

You should see **two running processes**:

- `/etc/open5gs/upf.yaml` (UPF1)
    
- `/etc/open5gs/upf2.yaml` (UPF2)
    

### Logs show PFCP Association

- UPF1 log (`/var/log/open5gs/upf.log`)
    
    ```
    [pfcp] INFO: pfcp_server() [127.0.0.7]:8805
    [pfcp] INFO: PFCP Heartbeat sent
    ```
    
- UPF2 log (`/var/log/open5gs/upf2.log`)
    
    ```
    [pfcp] INFO: pfcp_server() [127.0.0.8]:8805
    [pfcp] INFO: PFCP Heartbeat sent
    ```
    

### MongoDB Session Check

```bash
mongo
> use open5gs
> db.upf.find().pretty()
```

Both UPFs should appear as registered nodes.

---

✅ At this point:

- **UPF1 and UPF2 are running**
    
- Both are **registered with SMF**
    
- SMF can route sessions by **TAC mapping**
    

👉 This completes the **Multi-UPF registration stage**.  
Next step (separate guide) is aligning CU/DU configs so TAC1 → DU0 → UPF1 and TAC2 → DU1 → UPF2 for **handover testing**.

---

Do you want me to now extend this guide into the **handover setup stage** (CU, DU0, DU1 configs + commands) so it’s one continuous document, or keep this UPF guide separate?