## **5G Xn-based Inter-NG-RAN Handover**

This document provides a comprehensive overview of the Xn-based handover procedure in 5G networks, focusing specifically on the scenario where the User Plane Function (UPF) is not reallocated.

#### **1. Introduction to 5G Handovers**

**What is a Handover?**
==A handover is a critical process in mobile networks that ensures a user's device (UE) maintains continuous connectivity as it moves between different network cells. It seamlessly transfers the data connection from one base station (gNB) to another without interrupting the user's service.==

*   **Key Purpose:** To provide seamless mobility and prevent service disruptions like dropped calls or buffering videos.
*   **Mechanism:** ==The network coordinates the transition by transferring the user's session context, Quality of Service (QoS) parameters, and any buffered data from the source gNB to the target gNB.==

**Understanding the Xn Interface**
==The Xn interface is a direct communication link between two 5G base stations (gNBs). Its primary purpose is to allow gNBs to communicate directly with each other without involving the core network. This is crucial for fast and efficient handovers.==

*   **Xn vs. N2 Handovers:**
    *   **Xn Handover:** Direct gNB-to-gNB communication. It is faster and has lower latency.
    *   **N2 Handover:** Communication is routed through the core network (specifically the AMF). This is slower and used when a direct Xn link is unavailable.

---
#### **2. The Role of the User Plane Function (UPF)**

==The UPF is a core component of the 5G network. It acts as the anchor point for a user's data session and is responsible for:==
*   ==Packet routing and forwarding.==
*   ==Enforcing Quality of Service (QoS) policies.==
*   ==Connecting the mobile network to external data networks (like the internet).==

**Why Avoid UPF Reallocation?**
==In many handover scenarios, the UE's data path continues to be routed through the *same* UPF even after moving to a new gNB. This is known as a handover **without UPF reallocation**.== The network, specifically the Session Management Function (SMF), decides to do this for several reasons:

1.  ==**Geographic Proximity:** If the source and target gNBs are close to each other, the existing UPF is likely still well-positioned to handle the data traffic efficiently.==
2.  ==**Minimize Signaling Overhead:** Keeping the same UPF avoids the complex and time-consuming process of tearing down the data path to the old UPF and setting up a new one. This reduces the number of signaling messages required, making the network more efficient.==
3.  **Service Continuity:** If the user's current services (e.g., streaming, browsing) can be adequately supported by the existing UPF, there is no need to change it.

---
#### **3. Step-by-Step Xn-based Handover Procedure (Without UPF Reallocation)**

##### **Phase 1: Preparation Phase**

**Step 1: Measurement Reporting**
*   **Action:** The UE informs the network about its radio conditions.
*   **Message:** ==`RRC: MeasurementReport`==
*   **From -> To:** UE -> Source gNB
*   **Purpose (Why):** ==The source gNB instructs the UE to measure the signal strength and quality of nearby cells. When the UE finds a cell that is significantly stronger than its current one, it sends a `MeasurementReport`. This report acts as the trigger, telling the source gNB that a handover may be necessary to maintain good service quality.==

**Step 2: Handover Decision**
*   **Action:** The source gNB decides to initiate the handover.
*   **Message:** (Internal gNB Process)
*   **From -> To:** Source gNB -> Source gNB
*   **Purpose (Why):** ==Upon receiving the report, the source gNB's internal algorithm evaluates if a handover is truly needed. It selects the best candidate cell from the report and confirms that a direct Xn interface exists with the target gNB controlling that cell.==

**Step 3: Handover Request**
*   **Action:** The source gNB asks the target gNB to prepare for the UE.
*   **Message:** ==`XnAP: Handover Request`==
*   **From -> To:** Source gNB -> Target gNB
*   **Purpose (Why):** ==The source gNB uses the Xn interface to formally ask the target gNB to prepare to accept the UE. This message is critical as it contains the complete UE context, including its security credentials, active PDU sessions (data bearers), and QoS requirements, so the target gNB knows exactly what resources to prepare.==

**Step 4 & 5: Admission Control and Acknowledgment**
*   **Action:** The target gNB reserves resources and confirms its readiness.
*   **Message:** ==`XnAP: Handover Request Acknowledge`==
*   **From -> To:** Target gNB -> Source gNB
*   **Purpose (Why):** ==The target gNB performs **Admission Control** to ensure it has enough capacity. If it does, it allocates the necessary radio resources. It then sends an acknowledgment back to the source gNB. This message contains the radio configuration (like the new C-RNTI) that the UE will need to connect. This configuration is placed in a "transparent container," ready to be forwarded to the UE.==

**Step 6: Handover Command to the UE**
*   **Action:** The source gNB commands the UE to switch cells.
*   **Message:** ==`RRC: RRCReconfiguration`==
*   **From -> To:** Source gNB -> UE
*   **Purpose (Why):** ==The source gNB sends the `RRCReconfiguration` message (which contains the information from the target gNB's transparent container) to the UE. This is the official command for the UE to detach from the source cell and connect to the target cell.==

**Step 7: Data Forwarding Preparation**
*   **Action:** The source gNB prepares to forward data to prevent loss during the transition.
*   **Message:** ==`XnAP: SN Status Transfer`==
*   **From -> To:** Source gNB -> Target gNB
*   **Purpose (Why):** ==To ensure a lossless handover, the source gNB informs the target gNB of the sequence numbers (SN) of the last transmitted and received data packets. This allows the target gNB to know exactly where to resume the data flow. The source gNB also begins forwarding any buffered downlink data packets to the target gNB over the Xn interface.==

---

##### **Phase 2: Execution Phase**

**Step 8 & 9: UE Connects and Confirms**
*   **Action:** The UE synchronizes with the target gNB and confirms the handover.
*   **Message:** ==`RRC: RRCReconfigurationComplete`==
*   **From -> To:** UE -> Target gNB
*   **Purpose (Why):** ==The UE performs a Random Access (RACH) procedure on the target cell to establish a connection. Once synchronized, it sends the `RRCReconfigurationComplete` message. This signals to the target gNB that the UE is now successfully under its control and ready to transmit and receive data. This completes the radio-level part of the handover.==

---

##### **Phase 3: Completion Phase**

**Step 10: Path Switch Request**
*   **Action:** The target gNB informs the core network of the UE's new location.
*   **Message:** ==`NGAP: Path Switch Request`==
*   **From -> To:** Target gNB -> AMF
*   **Purpose (Why):** ==At this point, the core network (UPF) is still sending data to the old source gNB. The target gNB sends this message to the AMF to update the UE's location and request that the user plane path be "switched" to point to itself.==

**Step 11: Path Switch Acknowledgment**
*   **Action:** The core network confirms the data path has been updated.
*   **Message:** ==`NGAP: Path Switch Request Acknowledge`==
*   **From -> To:** AMF -> Target gNB
*   **Purpose (Why):** ==The AMF instructs the UPF to redirect the downlink data flow to the target gNB. Once confirmed, the AMF sends this acknowledgment. The data path is now officially switched, and all new data flows directly to the target gNB.==

**Step 12: Release of Resources**
*   **Action:** The source gNB is told to release all resources related to the UE.
*   **Message:** ==`XnAP: UE Context Release`==
*   **From -> To:** Target gNB -> Source gNB
*   **Purpose (Why):** ==With the handover fully complete, the target gNB sends a final message over the Xn interface to the source gNB. This command allows the source gNB to safely delete the UE's context and free up its radio resources, making them available for other users and ensuring the network remains efficient.==

---
#### **4. Benefits and Implications**

This handover procedure offers significant advantages:

| Implication Category | Benefit |
| :--- | :--- |
| **Network Implications** | **Reduced Signaling Load:** Fewer messages are sent to the core network, reducing congestion. <br> **Simplified Management:** The process is less complex than one involving UPF reallocation. <br> **Efficient Resource Use:** Frees up resources on the source gNB promptly. |
| **User Implications** | **Reduced Latency:** Direct gNB-to-gNB communication is faster, resulting in handover times of ~30ms compared to ~100ms for procedures requiring UPF reallocation. <br> **Improved Quality of Experience:** Less service interruption, ensuring a smooth experience for applications like video streaming and online gaming. |

***