
---

### **Preparation Before the Presentation**

(This remains the same)

1.  **Arrange Your Screen:** Have all five terminal windows open and neatly arranged.
2.  **Start the Network:** Have the network fully up and running with the UE attached *before* your demo slot.
3.  **Practice:** Do a full run-through of the new, longer demo script.

---

### **Live Demo Steps and Script (Updated)**

**(After you finish presenting Slide 8: "Demonstration & Verification")**

**Presenter:** "==What you saw on the slide is the summary, but now I'd like to show you this happening live. Let me switch to my terminal view.=="

---

### **Step 1: Set the Scene - The "Before" State**

**(Arrange your screen so the audience can see all terminal windows.)**

**Presenter:** "==Okay, what you see here is our live 5G network running on Google Cloud.==
*   ==In the **top-left**, we have the **Centralized Unit (CU)**, the brain of our radio network.==
*   ==In the **bottom-left**, you see our **Source DU (DU0)**, where the UE is currently connected.==
*   ==In the **bottom-right** is our **Target DU (DU1)**, which is ready and waiting.==
*   ==And in the **top-right**, this is our **simulated User Equipment (UE)**.=="

---

### **Step 2: Explain the "How" - Key Configurations**

**Presenter:** "==Before I trigger the handover, I want to quickly explain the two most important configuration changes we made to make this possible==."

**(Point to the UE terminal first, then the two DU terminals.)**

**Presenter:** "==First, and most critically, is how we handled the radio simulation. For a handover, the UE needs to 'hear' from both DUs at once. The standard model doesn't allow this. So, we inverted it:==
*   ==We configured the **UE as the RF server**—the central hub of the simulation.==
*   ==Then, we configured **both DUs as clients** that connect *to* the UE. This 'hub-and-spoke' model creates a single, shared environment where the handover can happen.=="

**(Point to the Source and Target DU terminals.)**

**Presenter:** "==Second, the network needs to tell the two cells apart. In the configuration file for each DU, we assigned a unique **Physical Cell ID**. The Source DU is Cell ID 0, and the Target DU is Cell ID 1. This is how the UE knows exactly which cell it's moving to during the handover.=="

**(Point to the CU terminal.)**

**Presenter:** "And third, the CU needs to be able to orchestrate everything. So, in the CU's configuration file, we explicitly defined the IP addresses and connection details for **both DUs**. This makes the CU aware of the entire radio environment and allows it to command the switch."

**Presenter:** "So, it's this combination—the UE as the server, unique cell IDs, and the CU managing both DUs—that sets the stage for the handover we're about to see."

---

### **Step 3: The Live Test - The Ping**

**Presenter:** "==Okay, with that setup explained, let's prove it works. I'm now going to start a continuous ping from the UE to the internet to show its live data connection.=="

**(In the OAI nr-UE terminal, type and run the command):**

```bash
ping -I oaitun_ue1 8.8.8.8
```

**Presenter:** "==As you can see, the ping is successful. We are getting replies, which means our data session is active and flowing through the Source DU.=="

**(Let the ping run for 5-10 seconds.)**

---

### **Step 4: The Main Event - Trigger the Handover**

**Presenter:** "==Now for the main event. The network is stable. I am going to manually trigger the F1 handover by sending a command to the CU==."

**Presenter:** "==**Please keep your eyes on the ping replies in the UE window.** We want to see if the connection survives the switch.=="

**(Move to your clean command terminal. Type and execute the handover command.)**

```bash
echo ci trigger_f1_ho | nc 172.17.0.93 9090 && echo "Handover Triggered!"
```

---

### **Step 5: The Result and Verification**

**(The logs will scroll rapidly. The ping in the UE window should continue with at most one or two dropped packets.)**

**Presenter:** "==And there it is! The handover is complete. As you can see, the ping continued with almost no interruption. The data session is still alive and running.=="

**(Point to the logs in the Target DU1 terminal.)**

**Presenter:** "==If you look at the logs for our **Target DU**, you can see it's now actively handling the UE's traffic. This confirms that the data path has been successfully switched==."

**(Point to the CU terminal.)**

**Presenter:** "==And the CU log confirms it as well, showing the **'handover... complete!'** message.=="

**(In the UE terminal, stop the ping with `Ctrl+C`.)**

**Presenter:** "==This live test confirms that our configuration was successful. We've performed a seamless F1 handover, switching the data path without dropping the user's session.=="

---

**(Switch back to your presentation.)**

**Presenter:** "That concludes the live demonstration. Now, let's talk a bit about why this is significant..."

**(Continue with Slide 9: "Use Cases & Significance".)**