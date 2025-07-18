
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
Studying for semtech embedded 

