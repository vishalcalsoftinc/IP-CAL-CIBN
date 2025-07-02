
---

You’ll see **why** kube-proxy sets up these `iptables` rules and **how** they make ClusterIP Services work.

---

## 🧩 **Big picture first:**

When you create a Service in Kubernetes:

- It gets a _virtual_ IP (e.g., `10.96.92.20`).
    
- This IP does **not** belong to any real network interface.
    
- Instead, it’s purely "virtual": traffic to this IP must somehow be forwarded to real pod IPs behind the service.
    

**kube-proxy** is the component that makes this work, usually by writing `iptables` rules.

---

## 🧰 **Step by step: how kube-proxy uses iptables**

Let’s imagine:

- You create a ClusterIP service with IP `10.96.92.20`.
    
- It has 4 backend pods.
    

Here’s what happens, simplified:

---

### ✅ Step 1: Traffic comes in

Some process inside the cluster tries to talk to `10.96.92.20` (the ClusterIP of your Service).

---

### 🏗️ Step 2: kube-proxy sets up a _main_ chain

kube-proxy maintains a top-level custom chain called:

```
KUBE-SERVICES
```

It installs a rule that says:

```
If packet’s destination is 10.96.92.20,
then jump to KUBE-SVC-XYZ
```

> Think of `KUBE-SVC-XYZ` as a chain just for this specific Service.

---

### ⚖️ Step 3: load balancing chain

In `KUBE-SVC-XYZ`, kube-proxy adds rules to distribute traffic to the 4 backend pods.

It does this using _probabilistic rules_:

- First rule: with 25% chance → jump to `KUBE-SEP-ABC` (pod 1)
    
- Second rule: if not matched, with 33.3% chance → jump to `KUBE-SEP-DEF` (pod 2)
    
- Third rule: if not matched, with 50% chance → jump to `KUBE-SEP-GHI` (pod 3)
    
- Final rule: else → jump to `KUBE-SEP-JKL` (pod 4)
    

> These probabilities make sure traffic is roughly evenly distributed over all 4 pods.

---

### 🎯 Step 4: pod-specific DNAT

Each pod has its own chain like `KUBE-SEP-ABC`.

In that chain, kube-proxy writes a rule:

```
Change destination IP from 10.96.92.20 → pod's real IP, e.g., 10.244.1.5
```

This is called:  
✅ **DNAT** (Destination Network Address Translation)

Now, the packet is addressed to the real pod IP.

---

### 📦 Step 5: packet reaches the pod

After DNAT:

- The Linux kernel forwards the packet to the real pod.
    
- The pod sees the incoming connection and responds.
    

---

## 🧠 **Why all this?**

- The Service’s ClusterIP is just a virtual front.
    
- kube-proxy uses iptables to:
    
    - Match packets sent to the Service IP.
        
    - Load balance them across real pod IPs.
        
    - Rewrite (DNAT) packets so pods can actually receive them.
        

> This all happens _very fast_ in kernel space, without needing kube-proxy to touch every packet at runtime.

---

## ⚙️ **Tables & Chains recap:**

- `nat` table: used for address translation (DNAT, SNAT).
    
- Custom chains (`KUBE-SERVICES`, `KUBE-SVC-*`, `KUBE-SEP-*`): created by kube-proxy to organize rules per Service and per Pod.
    

---

## ✅ **In summary (in simple words):**

> kube-proxy writes iptables rules so that:
> 
> - Traffic to the virtual ClusterIP is intercepted.
>     
> - It picks (by probability) one backend pod.
>     
> - It rewrites the packet to the real pod IP.
>     
> - Then the pod receives the packet as if it were sent directly to it.
>     

---

## Flow Diagram
[![](https://mermaid.ink/img/pako:eNqNlG1vmzAUhf_KlaVJmZQwsCEUf6jUptnWtZ2ida9VpMkBl7CCYWC2ZlX--67DyxKVVeXTtXWeY-xz7QcS5pEknFTyZy1VKM8SEZciWyrArxClTsKkEErDLE2k0os8AlG1A8DRkLCutCzPF42wG4wc2wqmVkAtar98TJ0XWqxSWRko6epXcFev5KQo8_vNY-Ti0-n8-_XnmUFMPcF68vXbDYwqWf5KQgnhWiRqYDH8b-cfNV9MTk5nMCpwb86wmh6oz-avGzUdVrMD9Zu3542aDavdA_W7i8tG7Q6oP0iRtgmY0tAwklZsjQFPl7qu5Vgecg3ZJzY5Pu5z4HAtVQSlibvSoHPYy6XjWi1yXSwcFiK8kxrWia5AYc-MIc4xokShRR-YEmhpysapo9GoC4vDldDhGrVlnaLJjzorKvMb-wk2tEg1XOYigpVIhQoTFTfz5uvs0NmEyeFLotdAvRcmc2zjxtj47lJtOuF_OOUwTytkfhsTxiw2bEOftmEHNp49bMKeNnE7k33G3WcwvqYwG0eobQoOZ-9PPnKY4ZqxhMiki9euPwFzBQdapTWizzWiWPcUey7F9in3uZTbU618suvktqs5zlZFrirZdOIKGxRG1-gHmdiAKIp0A8ktKCkjGeFWyZjEZRIRfivMCZNMlpkwY_JgVlkSvZaZXBKOZSRvRZ3qJVmqLXJ4-27yPCNclzWSZV7H696nLiKhu4ezny0xJ1nO8lppwpnt7kwIfyD3OJxaDvW9gLLp1GdHU2dMNoQ7gWf5PnVwzvf8I89n2zH5s1vWsZhtMxr4getS2wkoEjJKdF5eNc_37hXf_gUmecgh?type=png)](https://mermaid.live/edit#pako:eNqNlG1vmzAUhf_KlaVJmZQwsCEUf6jUptnWtZ2ida9VpMkBl7CCYWC2ZlX--67DyxKVVeXTtXWeY-xz7QcS5pEknFTyZy1VKM8SEZciWyrArxClTsKkEErDLE2k0os8AlG1A8DRkLCutCzPF42wG4wc2wqmVkAtar98TJ0XWqxSWRko6epXcFev5KQo8_vNY-Ti0-n8-_XnmUFMPcF68vXbDYwqWf5KQgnhWiRqYDH8b-cfNV9MTk5nMCpwb86wmh6oz-avGzUdVrMD9Zu3542aDavdA_W7i8tG7Q6oP0iRtgmY0tAwklZsjQFPl7qu5Vgecg3ZJzY5Pu5z4HAtVQSlibvSoHPYy6XjWi1yXSwcFiK8kxrWia5AYc-MIc4xokShRR-YEmhpysapo9GoC4vDldDhGrVlnaLJjzorKvMb-wk2tEg1XOYigpVIhQoTFTfz5uvs0NmEyeFLotdAvRcmc2zjxtj47lJtOuF_OOUwTytkfhsTxiw2bEOftmEHNp49bMKeNnE7k33G3WcwvqYwG0eobQoOZ-9PPnKY4ZqxhMiki9euPwFzBQdapTWizzWiWPcUey7F9in3uZTbU618suvktqs5zlZFrirZdOIKGxRG1-gHmdiAKIp0A8ktKCkjGeFWyZjEZRIRfivMCZNMlpkwY_JgVlkSvZaZXBKOZSRvRZ3qJVmqLXJ4-27yPCNclzWSZV7H696nLiKhu4ezny0xJ1nO8lppwpnt7kwIfyD3OJxaDvW9gLLp1GdHU2dMNoQ7gWf5PnVwzvf8I89n2zH5s1vWsZhtMxr4getS2wkoEjJKdF5eNc_37hXf_gUmecgh)