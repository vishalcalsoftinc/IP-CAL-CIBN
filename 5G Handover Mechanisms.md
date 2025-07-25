# Comprehensive Study on 5G Handover Mechanisms

## Introduction

Handover, or handoff, is a critical process in 5G New Radio (NR) networks that ensures a User Equipment (UE) maintains seamless connectivity as it moves between cells or network types. Given 5G's use of higher frequency bands (e.g., mmWave) and smaller cell sizes, handovers occur more frequently than in 4G LTE, making their efficiency paramount for uninterrupted voice, data, and mission-critical services. This study explores 5G handover types, procedures, advancements, and challenges, emphasizing their role in delivering ultra-reliable low-latency communications (URLLC) and enhanced mobile broadband (eMBB).

Compared to 4G, 5G handovers leverage advanced features like dual connectivity, network slicing, and the RRC Inactive state, improving latency and reliability. The 5G architecture, with disaggregated gNBs (comprising Central Units (CUs) and Distributed Units (DUs)) and a service-based 5G Core (5GC), introduces new handover scenarios and protocols, such as Xn and N2 interfaces, enhancing flexibility but adding complexity.

## Handover Fundamentals

Handovers occur when a UE in RRC Connected state transitions from a source cell to a target cell, driven by factors like signal quality, network load, or service requirements. Unlike cell reselection (RRC Idle/Inactive) or redirection (to idle mode), handovers maintain active sessions. Key triggers include:

- **Radio Conditions**: Events like A3 (neighbor cell signal stronger than serving cell by a threshold for a Time-to-Trigger (TTT) duration).
- **Load Balancing**: High gNB load prompts handover to a less congested cell.
- **Service Needs**: For example, voice calls may trigger EPS Fallback to 4G if Voice over NR (VoNR) is unavailable.

## Types of 5G Handovers

5G supports diverse handover types to accommodate its flexible architecture, including Standalone (SA) and Non-Standalone (NSA) deployments. The following table summarizes the main types:

|**Handover Type**|**Description**|
|---|---|
|**Intra-gNB**|UE moves between DUs within the same gNB CU, managed internally with minimal signaling.|
|**Xn-based Inter-gNB**|Between gNBs connected via Xn interface, coordinated within NG-RAN for low latency.|
|**N2-based Inter-gNB**|Coordinated via AMF using N2 interface when Xn is unavailable, involving more signaling.|
|**Inter-System**|Between 5GS and EPS (4G core) using N26 interface for seamless 5G-4G mobility.|
|**Inter-RAT**|Between NR and E-UTRA (LTE), staying on 5GC or switching to EPC, e.g., EPS Fallback.|
|**NSA/MR-DC**|In NSA or Multi-RAT Dual Connectivity, involving master (e.g., 4G eNB) and secondary (5G gNB) nodes.|
|**Non-3GPP Access**|To/from trusted or untrusted non-3GPP networks (e.g., Wi-Fi) for extended coverage.|

### Intra-gNB Handover

Managed within a single gNB, typically between DUs under one CU, this handover requires no core network involvement, minimizing latency. It is common in dense urban deployments with multiple DUs per CU.

### Xn-based Inter-gNB Handover

This handover occurs between gNBs with an active Xn interface, allowing direct communication for faster execution. It is analogous to 4G's X2-based handover but supports 5G-specific features like network slicing.

### N2-based Inter-gNB Handover

Used when no Xn interface exists or Xn handover is restricted, this involves the AMF via the N2 interface, increasing signaling overhead but ensuring compatibility across diverse network setups.

### Inter-System Handover

Facilitates mobility between 5GS and EPS using the N26 interface, enabling context transfer between AMF and MME for seamless 5G-4G transitions.

### Inter-RAT Handover

Handles transitions between NR and E-UTRA, often triggered by coverage gaps or service requirements (e.g., voice). It may involve staying on 5GC or switching to EPC.

### NSA/MR-DC Handover

In NSA deployments, the 4G eNB (master node) anchors control signaling, while the 5G gNB (secondary node) provides data. Handovers may involve changing the secondary node or switching roles.

### Non-3GPP Access Handover

Supports mobility to non-3GPP networks like Wi-Fi, enhancing coverage in hybrid environments.

## Detailed Handover Procedures

### Xn-based Handover Procedure

The Xn-based handover is efficient due to direct gNB communication. The procedure, as outlined in 3GPP TS 38.300, includes:

|**Step**|**Action**|**Details**|
|---|---|---|
|1|UE sends MeasurementReport|Includes serving and neighbor cell signal strength (RSRP, RSRQ).|
|2|Source gNB decides handover|Based on reports, cell load, UE capabilities, and mobility restrictions.|
|3|XnAP Handover Request to target gNB|Includes RRC container, target cell ID, PDU sessions.|
|4|Target gNB prepares|Allocates resources, sends XnAP Handover Acknowledge with RRC message.|
|5|Source gNB sends RRCReconfiguration|Contains Handover Command with target cell ID, C-RNTI, security settings.|
|6|SN Status Transfer|Source gNB sends PDCP SN, HFN, and forwards buffered data.|
|7|UE performs Random Access|Uses rach-ConfigDedicated to connect to target gNB.|
|8|UE sends RRCReconfigurationComplete|Completes handover, starts uplink data.|
|9|NGAP Path Switch Request to AMF|Updates PDU session paths; UPF sends End Marker to source and target gNB.|
|10|AMF sends Path Switch Acknowledge|Confirms path switch, lists switched/released PDU sessions.|
|11|Target gNB sends XnAP UE Context Release|Source gNB releases UE resources.|

**Notes**:

- Supports intra/inter-frequency handovers.
- Requires re-registration if the target gNB is in a different Tracking Area Code (TAC).
- Faster than N2-based due to minimal core network involvement [Techplayon, 2025](https://www.techplayon.com/5g-sa-inter-gnb-hanodver-xn-handover/).

### N2-based Handover Procedure

When Xn is unavailable, the N2-based (or NGAP-based) handover involves the AMF, similar to 4G's S1 handover. The steps are:

| **Step** | **Action**                                          | **Details**                                                          |
| -------- | --------------------------------------------------- | -------------------------------------------------------------------- |
| 1        | UE sends MeasurementReport                          | Includes serving and neighbor cell signal strength.                  |
| 2        | Source gNB decides handover                         | Sends N2 Handover Required to AMF with target gNB ID, PDU sessions.  |
| 3        | AMF sends Handover Request to target gNB            | Includes UE security context, capabilities, PDU session information. |
| 4        | Target gNB responds with HandoverRequestAcknowledge | Lists admitted PDU sessions, TargetToSource-TransparentContainer.    |
| 5        | AMF sends Handover Command to source gNB            | Triggers handover execution.                                         |
| 6        | Source gNB sends RRCReconfiguration                 | Includes target cell ID, C-RNTI, security algorithm identifiers.     |
| 7        | Source gNB sends UplinkRANStatusTransfer            | To AMF, includes PDCP SN for DRBs.                                   |
| 8        | AMF sends DownlinkRANStatusTransfer                 | To target gNB, forwards PDCP SN information.                         |
| 9        | UE performs Random Access                           | Using rach-ConfigDedicated on target gNB.                            |
| 10       | UE sends RRCReconfigurationComplete                 | Starts uplink data on target gNB.                                    |
| 11       | Target gNB sends Handover Notify                    | To AMF, includes UE location under TAC.                              |
| 12       | AMF sends UEContextReleaseCommand                   | To source gNB, indicating successful handover.                       |
| 13       | Source gNB sends UEContextReleaseComplete           | Releases UE context.                                                 |
|          |                                                     |                                                                      |

**Notes**:

- Supports direct or indirect data forwarding.
- Applicable for intra/inter-AMF mobility.
- Slower than Xn-based due to core network signaling [Techplayon, 2025](https://www.techplayon.com/5g-sa-inter-gnb-handover-n2-or-ngap-handover/).

### Inter-RAT Handover for EPS Fallback

EPS Fallback is an inter-RAT handover from 5G NR to 4G LTE when VoNR is unsupported, ensuring voice calls proceed via VoLTE. The procedure involves:

|**Step**|**Action**|**Details**|
|---|---|---|
|1|UE sends MeasurementReport|Triggered by A2/B2 events indicating 4G cell suitability.|
|2|gNB sends NGAP HandoverRequired|To AMF, includes target eNB ID, handover type.|
|3|AMF sends GTP-v2 Forward Relocation Request|To MME via N26 interface, transfers UE context.|
|4|MME sends S1AP HandoverRequest|To target eNB, sets up resources.|
|5|eNB sends S1AP HandoverRequestAcknowledge|Confirms resource allocation.|
|6|MME sends GTP-v2 Forward Relocation Response|To AMF, confirms setup.|
|7|AMF sends NGAP HandoverCommand|To gNB, triggers handover execution.|
|8|gNB sends mobilityFromNRCommand|To UE, includes targetRAT-Type (eutra), targetRAT Container.|
|9|UE performs RACH procedure|On target eNB, using rach-ConfigCommon or Dedicated.|
|10|UE sends RRCReconfigurationComplete|To eNB, completes handover.|
|11|eNB sends S1AP HandoverNotify|To MME, confirms UE connection.|
|12|MME sends GTP-v2 ForwardRelocationCompleteNotification|To AMF, signals completion.|
|13|AMF sends NGAP UEContextReleaseCommand|To gNB, releases 5G context.|
|14|MME sends GTP-v2 ForwardRelocationCompleteNotificationAcknowledge|To AMF, finalizes signaling.|
|15|gNB sends NGAP UEContextReleaseComplete|Confirms resource release.|

**Notes**:

- Introduces ~2-second call setup delay [3G4G Blog, 2020](https://blog.3g4g.co.uk/search/label/Handover).
- Triggered by B1/B2 events, depending on vendor implementation.
- Alternative: Release with Redirection, where 5G RRC Release instructs UE to reselect 4G [Techplayon, 2025](https://www.techplayon.com/5g-eps-fallback-5g-to-4g-handover/).

## Advancements in 5G Handovers

5G introduces several enhancements to improve handover performance, addressing challenges like frequent handovers in dense networks and high-mobility scenarios.

### Conditional Handover (CHO)

Introduced in 3GPP Release 16, CHO enhances mobility robustness by allowing the UE to execute handovers based on pre-configured conditions (e.g., RSRP/RSRQ thresholds). Key features:

- **Mechanism**: Multiple candidate target cells are prepared; UE stores RRCReconfiguration messages and applies them when conditions are met, reducing failure risks due to degraded links.
- **Benefits**: Reduces handover failures by ~40% at moderate speeds (~50 km/h) [Ericsson, 2020](https://www.ericsson.com/en/blog/2020/5/the-key-to-mobility-robustness-5g-networks).
- **Trade-offs**: Resource reservation for multiple cells may strain high-load networks.
- **Future Enhancements**: Release 17 supports CHO for PScell addition in MR-DC and RRC Inactive/Idle transitions.

### Dual Active Protocol Stack (DAPS) Handover

Also part of Release 16, DAPS achieves zero Handover Interruption Time (HIT) by maintaining dual protocol stacks for source and target cells. Key features:

- **Mechanism**: UE sends/receives data on both cells during handover; PDCP layer reorders/deduplicates packets. Target gNB signals source release.
- **Comparison**: Unlike 4G's Make-Before-Break, DAPS maintains dual stacks longer, ensuring no data loss [Devopedia, 2024](https://devopedia.org/5g-handover).
- **Applications**: Ideal for URLLC services like autonomous vehicles or industrial automation.

### Network Slicing

5G's network slicing allows tailored services (e.g., low-latency for gaming). Handovers consider slice availability in target cells, with admission control ensuring QoS continuity.

### RRC Inactive State

While not directly for handovers (which occur in RRC Connected), the RRC Inactive state enables quick connection resumption, reducing latency in mobility scenarios transitioning to Connected state.

### L1/L2 Triggered Mobility

Introduced in 5G Advanced (Release 18), this reduces signaling overhead and HIT (down to ~20 ms) by handling handovers at lower layers, ideal for high-mobility scenarios [Ericsson, 2024](https://www.ericsson.com/en/blog/2024/8/5g-advanced-handover-triggered-mobility).

## Challenges and Optimizations

- **Challenges**:
    - **Frequent Handovers**: mmWave and small cells increase handover frequency, risking ping-pong effects.
    - **Network Slicing**: Ensuring slice continuity adds complexity.
    - **High Mobility**: Fast-moving UEs (e.g., trains) challenge handover reliability.
    - **EPS Fallback**: Introduces delays for voice services.
- **Optimizations**:
    - **Adaptive Parameters**: TTT, Hysteresis Margin, and A3 offset adjustments reduce ping-pong and failures.
    - **Machine Learning**: ML algorithms (e.g., Kriging Interpolator) predict optimal handover targets, reducing unnecessary handovers by 5-7% [Devopedia, 2024](https://devopedia.org/5g-handover).
    - **CHO and DAPS**: Mitigate failures and interruptions.
    - **Beam Management**: Beam-level mobility (no RRC signaling) reduces HIT for intra-cell movements.

## Performance Metrics

|**Metric**|**Typical Value**|**Notes**|
|---|---|---|
|**Handover Interruption Time (HIT)**|30-60 ms|Shorter for intra-gNB or beam mobility; DAPS achieves 0 ms.|
|**Time-to-Trigger (TTT)**|Variable|Low TTT risks ping-pong; high TTT risks failures; adaptive TTT optimizes.|
|**Timer T304**|Configurable|Ensures handover completion before expiry to avoid failure.|

## Conclusion

5G handovers are pivotal for seamless connectivity in diverse scenarios, from intra-gNB transitions to inter-RAT fallbacks. Xn-based handovers offer low latency, while N2-based ensure compatibility. Inter-RAT handovers like EPS Fallback bridge 5G and 4G for voice services. Advancements like CHO and DAPS enhance reliability, addressing challenges in dense, high-mobility environments. Ongoing research into ML and L1/L2 mobility promises further improvements, ensuring 5G meets the demands of eMBB, URLLC, and massive IoT.

## References

- 3GPP TS 38.300: NR; NR and NG-RAN Overall Description [3GPP](https://www.3gpp.org/ftp/Specs/archive/38_series/38.300/)
- 3GPP TS 38.401: NG-RAN; Architecture Description
- 3GPP TS 38.412: NG-RAN; NG Signalling Transport
- 3GPP TS 38.331: NR; Radio Resource Control (RRC) Protocol Specification
- 3GPP TS 38.423: NG-RAN; Xn Application Protocol (XnAP)
- Devopedia: 5G Handover, 2024 [Devopedia](https://devopedia.org/5g-handover)
- Ericsson: The Key to Mobility Robustness in 5G Networks, 2020 [Ericsson](https://www.ericsson.com/en/blog/2020/5/the-key-to-mobility-robustness-5g-networks)
- Ericsson: 5G Advanced Handover: L1/L2 Triggered Mobility, 2024 [Ericsson](https://www.ericsson.com/en/blog/2024/8/5g-advanced-handover-triggered-mobility)
- Techplayon: 5G SA Inter gNB Handover - Xn Handover, 2025 [Techplayon](https://www.techplayon.com/5g-sa-inter-gnb-hanodver-xn-handover/)
- Techplayon: 5G SA Inter gNB Handover - N2 or NGAP Handover, 2025 [Techplayon](https://www.techplayon.com/5g-sa-inter-gnb-handover-n2-or-ngap-handover/)
- Techplayon: 5G EPS Fallback | 5G to 4G Handover, 2025 [Techplayon](https://www.techplayon.com/5g-eps-fallback-5g-to-4g-handover/)
- 3G4G Blog: Handover, 2020 [3G4G Blog](https://blog.3g4g.co.uk/search/label/Handover)