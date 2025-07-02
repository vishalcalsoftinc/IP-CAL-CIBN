
---
### **GTP Tunneling - The Network's Postal Service**

This topic explains the GPRS Tunneling Protocol (GTP), which is fundamental to how your 5G network moves user data. In your architecture, you see GTP-U on the F1-U and N3 interfaces. This protocol is the reason a mobile user can maintain a constant IP address and a seamless internet session while moving around.

#### **A. What is Tunneling? The Core Concept**

*   **Definition:** ==IP Tunneling is the process of putting an entire IP packet inside *another* IP packet. This is called **encapsulation**.==
*   **Analogy (from the video):** Imagine a letter addressed from person A to person B. Instead of mailing it directly, you take that sealed envelope and put it inside a *new, larger envelope*. You then address this outer envelope from a "Source" to a "Destination".
    *   **Inner Packet:** The original letter (e.g., from `arstechnica.com` to `Your_Phone_IP`).
    *   **Outer Packet:** The larger envelope (e.g., from `CU-UP` to `UPF`).
*   **Why do this?**
    1.  **Mobility (Primary Reason for GTP):** To route a packet along a path that is not "topologically correct." Your phone has a public IP address, but it's physically connected to a base station (eNB/gNB) on a private network. Normal IP routing would fail. The tunnel creates a virtual path from the core network's edge (P-GW/UPF) directly to the base station serving the user. The video uses the analogy of mail forwarding: when you move, you tell the post office to put your mail into a new envelope and send it to your new address.
    2.  **Security (Not the main goal of GTP):** Encapsulation can hide the original source and destination IPs from anyone sniffing the network between the tunnel endpoints.

---

#### **B. GTP in Action: The User Plane (GTP-U)**

GTP-U is responsible for carrying the actual user data (your web traffic, video stream, etc.). It is simply **IP/UDP Tunneling**.

**The Downlink Data Flow (Internet to Phone):**
![[Pasted image 20250630120031.png]]

1.  A packet from the internet arrives at the **P-GW / UPF** (Packet Gateway / User Plane Function). The packet is destined for your phone's IP address (e.g., `46.1.78.189`).
2.  The P-GW/UPF knows this user is currently being served by a specific **S-GW / CU-UP** (Serving Gateway / Centralized Unit-User Plane). It can't just send the packet onto the network; the S-GW's IP address (`10.10.10.x`) is on a different network from the phone's IP.
3.  **Encapsulation #1 (P-GW → S-GW):**
    *   The P-GW takes the original packet.
    *   It wraps it in a **GTP-U Header**.
    *   It wraps that in a **UDP Header**. The destination port is **2152** (the IANA-assigned port for GTP-U).
    *   It wraps that in a new **IP Header**. The source is the P-GW's IP, and the destination is the S-GW's IP.
4.  The S-GW receives this UDP packet on port 2152. It knows this is GTP-U traffic.
5.  **The Problem of Many Users:** The S-GW is handling traffic for *thousands* of users. How does it know this specific packet is for your phone ("Mathmet") and not someone else's ("Yusuf")?
    *   **Solution: The Tunnel Endpoint Identifier (TEID).** The **GTP-U header** contains a unique 32-bit number called the TEID. This TEID acts as a specific identifier for your phone's data session on that interface.
    *   When the S-GW sees a packet arrive with `TEID = K`, its lookup table says, "TEID K belongs to Mathmet, who is at eNodeB X."
6.  **Encapsulation #2 (S-GW → eNB/gNB):**
    *   The S-GW strips off the outer IP/UDP/GTP headers.
    *   It re-encapsulates the original packet into a *new* tunnel destined for the **eNB/gNB** (the base station). This new tunnel will have a different TEID (`TEID = X`) that the eNB uses to identify the user.
7.  The eNB receives the packet, looks at the TEID, knows it's for your phone, strips all the tunneling headers, and sends the original, clean IP packet over the air to your device.

**Key Takeaway:** GTP-U is a chain of tunnels that forwards a user's data packet from the edge of the core network to the specific base station the user is connected to, using TEIDs at each hop to distinguish between users.

---

#### **C. Setting Up the Tunnels: The Control Plane (GTP-C)**

The tunnels don't create themselves. A signaling protocol is needed to negotiate the TEIDs and establish the path. This is the job of **GTP-C (GTP Control Plane)**.

![[Pasted image 20250630120139.png]]

*   **Protocol:** Like GTP-U, GTP-C also runs on top of **UDP**. It uses a different destination port: **2123**.
*   **Purpose:** To create, modify, and delete the user plane tunnels.
*   **Key Messages:**
    *   ==**`Create Session Request`:** Sent from a "downstream" node (like an S-GW) to an "upstream" node (like a P-GW) to initiate a tunnel for a new user session.==
    *   ==**`Create Session Response`:** The reply from the upstream node.==
    *   **`Modify Bearer Request`:** Used to change the parameters of an existing tunnel (e.g., after a handover).

**The Tunnel Setup Flow (Simplified):**
![[Pasted image 20250630121626.png]]

1.  A user ("Mathmet") connects. The **S-GW** needs to create a tunnel to the **P-GW**.
2.  **S-GW sends `Create Session Request`:** This message is sent to the P-GW's GTP-C port (2123). Inside this message, the S-GW says:
    *   "I have a user with IMSI `XXX`."
    *   "For any **downlink user plane traffic** for this user, please send it to my user-plane IP address (`IP_SGW_U`) and use **TEID_SGW**." (This is the S-GW's downlink TEID).
    *   "For any **control plane replies** regarding this user, send them to my control-plane IP address (`IP_SGW_C`) and use **TEID_SGW_C**."
3.  **P-GW receives the request and sends `Create Session Response`:** The P-GW replies to the S-GW. Inside this message, it says:
    *   "Request accepted."
    *   "For any **uplink user plane traffic** from this user, please send it to my user-plane IP address (`IP_PGW_U`) and use **TEID_PGW**." (This is the P-GW's uplink TEID).
    *   "For any subsequent **control plane messages** regarding this user, send them to my control-plane IP (`IP_PGW_C`) and use **TEID_PGW_C**."

**Result:** ==A bi-directional control plane path and a bi-directional user plane path are now established. Each direction has a destination IP and a destination TEID, which uniquely identifies the user's session at the receiving end.==

---

#### **D. Practical Details & Wireshark Analysis**

*   **GTP-C v1 vs. v2:**
    *   3G networks used GTP-C v1.
    *   4G/5G (EPC/5GC) networks use **GTP-C v2** (specification **29.274**). The signaling changed significantly to support new concepts like EPS Bearers.
    *   The `Create Session Request` message you see in the Wireshark trace is a GTP-C v2 message.
*   **GTP-U v1:**
    *   The user plane tunneling mechanism itself is so simple and effective that it **did not need to change** from 3G to 4G/5G.
    *   Therefore, modern networks still use **GTP-U v1** (specification **29.281**). This saved operators from having to upgrade their user plane hardware.
*   **Initial `Create Session Request` TEID:** The very first request to establish a session for a user is sent with a **TEID of 0**. This acts like a well-known "reception desk" or "butler" port. The response from the P-GW then assigns a specific, non-zero TEID for all future signaling for that user.
*   **Fully Qualified TEID (F-TEID):** In the signaling messages, you'll see the term F-TEID. This is simply the combination of **{IP Address + TEID}**, which together uniquely identifies a tunnel endpoint.
*   **Path Management (Heartbeat):**
    *   How do two nodes know if the other is still alive? They use a heartbeat mechanism.
    *   Periodically, one node sends an **`Echo Request`** message to the other.
    *   The other node must reply with an **`Echo Response`**.
    *   If no response is received after a few tries, the node assumes the path is down and will tear down all the user tunnels associated with it. This is a critical failure detection mechanism.

#### **E. GTP Interfaces in Your 5G Architecture**

*   **F1-U Interface:** Between the **OAI-DU** and **OAI-CU-UP**. This is a GTP-U tunnel used to carry user data within the disaggregated gNB.
*   **N3 Interface:** Between the **OAI-CU-UP** and the **Open5GS-UPF**. This is the main GTP-U tunnel between the RAN and the 5G Core.
*   **Control Plane Interfaces (N2, S5/N11):** ==While your architecture focuses on the RAN-Core split, the control plane interfaces like N2 (gNB to AMF) and S11/N11 (MME/AMF to S-GW/SMF) and S5/N4 (S-GW/SMF to P-GW/UPF) will use GTP-C (or protocols like PFCP in pure 5G SA for N4) to set up the corresponding user plane tunnels. The `Create Session` flow described above is a classic example of what happens across these interfaces.==