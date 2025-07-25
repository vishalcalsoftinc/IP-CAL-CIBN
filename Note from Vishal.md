
---
26-6
Radio Access Network (NG-RAN) - Your OAI Setup
5G Core (5GC) - Your Open5GS Setup

---

27-6
Kubernetes Architecture & Components 
- Pod 
- Deployment 
- StatefulSet 
- Service 
- Secret 
- ConfigMap 
- Namespace
Kubernetes networking concepts 
- Container networking
- Pod networking
- CNI , Flannel

---

30-6
GTP tunnelling theory
Calico theory

---

1-7
[[Pod Creation Workflow]]
[[Kubernetes services]]
[[Kubernetes Network Policies]] 
Calico implementation on Multi-note cluster created using KIND in wsl

---
2-7
Install VirtualBox and tried to run kubeadm , got some issues related to virtualbox network drivers , got them resolved .
Kubernetes Commands Documentation

---
3-7
Created two vms , installed kubeadm on both nodes

---
4-7
Connected two clusters using Calico and BGP
and tested connection between them

30-6
Calico theory 

1-7
Calico implementation on Multi-note cluster created using KIND in WSL.

2-7
Install VirtualBox and tried to run kubeadm , got some issues related to virtualbox network drivers , got them resolved .
Kubernetes Commands Documentation

3-7
Created two vms , installed kubeadm on both nodes

4-7
Connected two clusters using Calico and BGP
and tested connection between them

---
9-7
Tried to build OAI locally - failed 

10-7
Successfully built the OAI setup locally in WSL and shared it with Sakshi and Sushmita.
Tried to built the same in VM but failed due to CPU restrictions.
Built docker images of the OAI components and shared them with Sakshi and Sushmita.
Started reading the configuration the OAI components .


---


9-7
Tried to build OAI locally - failed  

10-7
Successfully built the OAI setup locally in WSL and shared it with Sakshi and Sushmita.
Tried to built the same in VM but failed due to CPU restrictions.
Built docker images of the OAI components and shared them with Sakshi and Sushmita.
Started reading the configuration the OAI components .

11-7
Read the OAI deployment , service files 

14-7
Tried to deploy OAI components (depl , svc) on VM created using vagrant and VirtualBox . The deployment is failing again and again because the OAI image requests for AVX (CPU feature) but the VM is not AVX enabled . AVX is present on the host . Found out that VirtualBox is not passing that feature to the VM . Possible fixes either deploy on host machine or try alternative suitable virtualizer . 
Created a network diagram of OAI including all components and the data flow.

15-7
Tried to run open5gs on vm

16-7
Setup WSL
Installed open5gs 
Wrote configuration files for open5gs
Added a subcriber with webui

17-7
Build OAI components
wrote the OAI (cucp , cuup , du , nr-ue) conf files
ran OAI with the above conf ( failed )
Studying for semtech embedded 

18-7
Assigned to create a demo 5g setup with UERANSIM and Open5gs

Studying for semtech embedded 


This is a challenging reply from your manager, but it's not a definitive "no" yet. The phrase "will discuss" is your small window of opportunity. The key is to respond in a way that is polite, firm about your constraint, and focused on finding a solution.

You need to address his two main points: the project's need for "presence" and the "company policy."

Here are a few options for your reply, from most recommended to more direct.

---

### Option 1: The Collaborative & Proactive Approach (Highly Recommended)

This approach thanks him, reiterates your position, and proactively suggests a solution to his "presence" concern. This shows you are a problem-solver.

**"Thank you for the quick reply and for your willingness to discuss this, Vishal. I appreciate you explaining the project and policy requirements.**

**I want to reiterate that I am very enthusiastic about the Semtech project. Unfortunately, my family situation in Kolkata is inflexible at this time, making relocation genuinely impossible for me.**

**To address the need for 'presence,' I am more than willing to travel to Pune for key project milestones, like kick-offs, important workshops, or integration phases. For day-to-day work, I am confident I can establish a strong virtual presence through dedicated communication and collaboration with the team.**

**Could we have a brief call to discuss this possibility? I am hopeful we can find a solution that works for the project and allows me to contribute effectively."**

**Why this works:**
*   **Polite & Appreciative:** You start by thanking him.
*   **Firm but Professional:** You state that relocation is "genuinely impossible" without being demanding.
*   **Offers a Compromise:** The offer to travel is a powerful negotiating tool. It shows you are willing to meet him halfway.
*   **Proposes a Solution:** You are actively trying to solve the problem of "presence."
*   **Clear Call to Action:** You ask for a call to discuss it further.

---

### Option 2: The More Direct & Simple Approach

This is a shorter version if you prefer to be more to the point.

"Thanks for the context, Vishal. I understand the preference for on-site work. Given my situation, relocation isn't an option for me. Is there any flexibility in the company policy for a remote arrangement, perhaps with occasional travel to Pune for important meetings? I'm very keen to make this work and am confident I can be a valuable member of the Semtech team from Kolkata."

**Why this works:**
*   **Concise:** Gets straight to the point.
*   **Asks about Policy Flexibility:** Directly but politely questions the rigidity of the policy.
*   **Reiterates Value:** Reminds him you want to be a valuable contributor.

---

### Option 3: The "Final Stance" Approach (Use if you are prepared to lose the opportunity)

This is the most direct option and should be used if you've decided that if they don't bend, you will have to decline.

**"I appreciate the clarification, Vishal. Since I am unable to relocate due to my personal responsibilities, I will unfortunately have to decline the opportunity if the on-site requirement in Pune is non-negotiable. If any flexibility becomes possible, please let me know as I remain very interested in the project."**

**Why to be cautious with this:**
*   **Closes the Door:** This sounds more like an ultimatum and may end the discussion.
*   **Less Collaborative:** It doesn't offer solutions, it just states a condition.

**My strong recommendation is to use Option 1.** It keeps the conversation alive, shows your commitment, and frames you as a flexible and proactive employee who is trying to find a solution that benefits everyone.

