
---

These study notes provide a comprehensive summary of the concepts of 5G Network Slicing, based on the provided lecture transcript.

### 1.0 Introduction to Network Slicing

#### **1.1 What is Network Slicing?**
==Network slicing is a fundamental 5G technology that allows for the creation of multiple virtualized, logical networks on top of a single, shared physical network infrastructure. Each logical network, or "slice," is an independent, end-to-end network tailored with specific resources and functionalities to meet the diverse requirements of a particular application or service. This approach moves away from a "one-size-fits-all" network to a more flexible and efficient model.==

#### **1.2 Key Characteristics**
*   ==**End-to-End (E2E):**== A network slice is a complete, E2E logical network. This includes the Radio Access Network (RAN), the transport network (backhaul, mid-haul, front-haul), and the Core Network (CN), extending all the way to the application server.
*   ==**Service-Specific:**== Each slice is designed to deliver a specific Quality of Service (QoS) tailored to the needs of the service it supports. For instance, an IoT service slice would have different requirements than a high-definition video streaming slice.
*   ==**Isolation:**== Slices are logically separated to ensure that the performance and security of one slice do not impact others. Resources allocated to one slice cannot be consumed by another, and a security breach in one slice is confined to that slice.

#### **1.3 Example Scenario**
On a single 5G physical infrastructure, an operator can create multiple slices simultaneously:
*   **v2x (Vehicle-to-Everything) Slice:** For autonomous cars, requiring ultra-low latency and high reliability.
*   **Smartphone Service Slice:** For general mobile broadband users.
*   **Smart Meter Slice:** For massive IoT deployments with low data and power requirements.

### 2.0 Standardization and Evolution

#### **2.1 ITU-T Framework (IMT-2020)**
The International Telecommunication Union Telecommunication Standardization Sector (ITU-T) provided a foundational framework for network slicing in its **Y.3112** document (Dec 2018) for ==IMT-2020 (the ITU's name for 5G).==

*   **Core Concept:** The framework conceptualized slices based on 3GPP's three main usage scenarios:
    1.  ==**Enhanced Mobile Broadband (eMBB):**== For services like high-speed internet, voice, AR/VR.
    2.  ==**Massive Machine Type Communications (mMTC):**== For large-scale IoT, like logistics and smart agriculture.
    3.  ==**Ultra-Reliable Low Latency Communications (URLLC):**== For critical applications like autonomous vehicles and factory automation.
*   **Implementation:** The framework showed that a User Equipment (UE), like a car, could connect to multiple slices simultaneously (e.g., a uRLLC slice for driving functions and an eMBB slice for its infotainment system). It also introduced the concept of common network functions that can be shared across multiple slices.

#### **2.2 3GPP Standardization Journey**
The 3rd Generation Partnership Project (3GPP) began formalizing network slicing in **Release 14** (2016) with a feasibility study (TR 22.891). This study outlined two key sets of requirements.

##### **2.2.1 Service Requirements**
*   ==Operators must be able to compose and create network slices dynamically from a common infrastructure.==
*   Terminals (UEs) must be able to associate with and obtain services from specific network slices, typically based on their subscription.

##### **2.2.2 Operational Requirements**
*   ==**Creation & Management:**== Operators need the ability to create, manage, and authorize third parties (like virtual operators) to manage their own slice configurations.
*   ==**Isolation:**== Slices must be isolated in terms of resources and security. A cyber-attack on one slice should not affect others.
*   ==**Elasticity:**== The system must support dynamic scaling of resources for a slice without impacting other slices.
*   ==**Lifecycle Management:**== The system must allow for the addition, removal, or modification of slices with minimal impact on ongoing services.
*   ==**E2E Resource Management:**== Operators must be able to manage all resources (RAN and Core) allocated to a specific slice.

### 3.0 Release-wise Evolution in 3GPP

Network slicing capabilities have evolved progressively with each 3GPP release.
*   **Release 15:** Laid the foundation for network slicing, introducing the core concepts and architecture.
*   **Release 16:** Added key enhancements, including slice-level authentication and authorization.
*   **Release 17:** Introduced RAN-centric enhancements and slice admission control.

### 4.0 Core Concepts for Slicing

#### **4.1 5G Architecture & PDU Sessions**
*   ==**Service-Based Architecture (SBA):** The 5G Core Network separates the Control Plane (for signaling and management) from the User Plane (for data traffic). Network Functions (NFs) in the control plane expose services that other NFs can consume.==
*   ==**PDU Session:** A Protocol Data Unit (PDU) Session is the logical E2E connection between a UE and a Data Network (e.g., the internet) through the 5G system. All user data flows over a PDU session.==
*   ==**QoS Flow:** Within a single PDU session, there can be multiple Quality of Service (QoS) flows. Each QoS flow is associated with a specific service and has its own characteristics (e.g., guaranteed bit rate, latency). A network slice can support one or more QoS flows.==

#### **4.2 Network Functions (NFs) for Slicing**
3GPP introduced specific NFs to manage network slicing:

*   ==**NSSF (Network Slice Selection Function) (Rel-15):**==
    *   ==**Purpose:** Helps the Access and Mobility Management Function (AMF) select the correct network slice for a UE based on subscription data, requested services, and slice availability in the current area.==
    *   **Consumers:** AMF, NSSF in other networks (for roaming).

*   ==**NSSAAF (Network Slice Specific Authentication and Authorization Function) (Rel-16):**==
    *   ==**Purpose:** Enables an optional, secondary layer of authentication specific to a service or slice. This is used when a slice requires a higher level of security beyond the primary network authentication.==
    *   **Consumer:** AMF.

*   ==**NSACF (Network Slice Admission Control Function) (Rel-17):**==
    *   ==**Purpose:** Manages the load on a slice by controlling admissions. It enforces limits on the number of registered UEs or active PDU sessions allowed on a slice to prevent overload.==
    *   **Consumers:** AMF, Session Management Function (SMF).

### 5.0 Slice Identification, Selection, and Implementation

#### **5.1 Identifying a Slice: The S-NSSAI**
==Each network slice is uniquely identified by a **Single Network Slice Selection Assistance Information (S-NSSAI)**.==
*   **Structure:** The S-NSSAI consists of two parts:
    1.  ==**SST (Slice/Service Type):** An 8-bit value indicating the service type (e.g., eMBB, uRLLC).==
    2.  ==**SD (Slice Differentiator):** An optional value used to distinguish between multiple slices of the same SST.==
*   **Standard SST Values:** 3GPP has defined standard values for SST:
    *   `1`: eMBB
    *   `2`: uRLLC
    *   `3`: Massive IoT
    *   `4`: V2X
    *   `5`: High-Performance Machine Type Communications (HPMTC)

#### **5.2 Slice Selection Process**
The network uses several lists of S-NSSAIs to manage a UE's access to slices:
*   **Configured NSSAI:** A list of slices the UE is provisioned with for its home network.
*   **Default Configured NSSAI:** A standard list of slices expected to be widely available, crucial for roaming.
*   **Requested NSSAI:** The list of slices the UE requests during its registration procedure.
*   **Allowed NSSAI:** The list of slices the network grants the UE access to in its current location, based on subscription and availability.
*   **Rejected NSSAI:** Slices that were requested but denied, with a cause for rejection.

#### **5.3 Slice Implementation Details**
*   ==**Shared vs. Dedicated NFs:** A slice can be built with dedicated NFs for complete isolation, or it can share certain NFs (like AMF, NRF) with other slices to improve resource efficiency. The NSSF and Network Repository Function (NRF) are typically common to all slices.==
*   ==**UE, PDU Sessions, and Slices:**==
    *   ==A single PDU session belongs to exactly one network slice.==
    *   ==A UE can connect to multiple slices simultaneously by establishing different PDU sessions, one for each slice.==
    *   ==When a UE connects to multiple slices, it is served by a single, common AMF.==
*   ==**System Limits:**==
    *   ==A UE can be simultaneously connected to a maximum of **8 network slices**.==
    *   ==A UE can have a maximum of **15 PDU sessions** active simultaneously across all slices.==

### 6.0 Advanced Features

#### **6.1 Network Slice Simultaneous Registration Group (NSSRG)**
This feature allows a UE to register to a pre-defined group of network slices simultaneously during its initial registration procedure, streamlining the connection process. This information can be pre-provisioned in the UE or provided by the AMF.

#### **6.2 Data Rate Limitation per Slice (UE Slice AMBR)**
Introduced in Release 17, this is an optional parameter that allows operators to enforce an **Aggregated Maximum Bit Rate (AMBR)** for a specific UE *within* a particular slice. This ensures that even if a UE has multiple PDU sessions in one slice, its total data rate does not exceed the slice-specific limit.

### 7.0 RAN Slicing (Support in the Radio Access Network)

The RAN must also be slice-aware to ensure E2E QoS.
*   **Concept:** RAN resources (e.g., frequencies, radio blocks) are allocated and prioritized based on the slices they support. For example, a cell in a factory area might prioritize resources for a uRLLC slice over an eMBB slice.
*   **Slice-Aware Procedures:** This awareness influences key RAN procedures:
    *   **Cell Reselection:** A UE will prioritize reselecting a cell that supports its active slices, even if another nearby cell has a stronger signal but doesn't support the required slice.
    *   **RACH Configuration:** The Random Access (RACH) process can be configured differently for different slices.
*   **Network Slice Access Stratum Groups (NSAGs):** To manage this in the RAN, slices are bundled into NSAGs on a per-Tracking Area (TA) basis. This simplifies the signaling and management of slice support in the radio network.

### 8.0 Interworking and Commercialization

#### **8.1 Interworking with 4G/EPC**
Release 16 supports mobility between 5G network slices and 4G Dedicated Core Networks (DECOR). This allows for service continuity when a user moves from a 5G coverage area with a specific slice (e.g., IoT) to a 4G area that has a corresponding dedicated network for that service.

#### **8.2 GSMA Network Slice Templates**
While 3GPP defines the technical framework, it does not specify the exact performance parameters for a slice. The GSMA has filled this gap by creating templates to standardize slice characteristics for operators.
*   **GST (Generic Network Slice Template):** Provides a list of all possible attributes that can define a slice (e.g., availability, latency, device density).
*   **NeST (Network Slice Type):** A GST that has been filled with specific values to create a template for a commercial slice type. This is crucial for ensuring consistent service levels, especially in roaming scenarios.
    *   **Example (Massive IoT NeST):** May specify attributes like 99.9% availability, high device density (e.g., 100,000 devices/km²), and low device velocity (e.g., <3 km/h).