
---
## The Pod Creation Workflow: A Step-by-Step Walkthrough

This sequence demonstrates how the Kubernetes control plane components work together to bring a pod to life.

---

1. **User -> `kubectl`:**  
    A user runs:
    

```bash
kubectl create deployment hello-world --image=nginx
```

This command tells Kubernetes to create a Deployment resource managing pods that run the specified image.

---

2. **`kubectl` -> API Server:**  
    `kubectl` builds the Deployment manifest in YAML/JSON and sends it to the **API server** using a REST call.  
    The API server is the single entry point for all cluster requests.
    

---

3. **API Server -> `etcd`:**  
    The API server:
    

- Validates the request (e.g., checks schema, permissions, quota).
    
- Writes the new Deployment object as a JSON/protobuf blob to **etcd**, which is the cluster’s single source of truth.
    
- Responds to `kubectl` with “deployment created.”
    

---

4. **Deployment Controller:**  
    The Deployment controller (part of the **Controller Manager**) is continuously watching the API server for Deployment objects.  
    It sees the new Deployment and:
    

- Calculates that it needs to create a new ReplicaSet with `spec.replicas=1`.
    
- Sends a request to the API server to create the ReplicaSet.
    

The API server validates and writes the ReplicaSet object to **etcd**.

---

5. **ReplicaSet Controller:**  
    The ReplicaSet controller, also in the **Controller Manager**, is watching ReplicaSet objects.  
    It sees the new ReplicaSet and notices:
    

- `desired replicas: 1`
    
- `current replicas: 0`
    

So it:

- Creates a new **Pod** object (with `status: Pending`) by sending a request to the API server.
    

The API server validates and writes this Pod object to **etcd**.

---

6. **Scheduler:**  
    The **Scheduler** is watching the API server for pods with:
    

- `spec.nodeName` not set
    
- `status: Pending`
    

It sees the new pod and:

- Selects a suitable node (e.g., `node1`) based on resource availability, taints/tolerations, affinities, etc.
    
- Sends an **update request** to the API server to patch the pod’s `spec.nodeName` field to `node1`.
    

---

✅ **Intermediate step:**

- The **API server** validates this update and writes the updated pod object (now with `spec.nodeName: node1`) back to **etcd**.
    
- The kubelet does _not_ watch etcd directly; instead, it watches the API server.
    

---

7. **`kubelet` (on `node1`):**  
    The `kubelet` on `node1` is continuously watching the API server for new pods assigned to its node.  
    It sees the new pod that now has `spec.nodeName: node1`.
    

---

8. **`kubelet` -> CRI -> Container Runtime:**  
    The `kubelet` instructs the container runtime (via the **Container Runtime Interface**, e.g., containerd or CRI-O) to:
    

- Pull the container image (e.g., `nginx`) if not already present.
    
- Create and start the container.
    

---

9. **`kubelet` -> CNI Plugin:**  
    The `kubelet` then calls the **Container Network Interface (CNI) plugin** to:
    

- Create a network namespace for the pod.
    
- Set up a virtual Ethernet pair (veth).
    
- Assign an IP address.
    
- Add routes so the pod can communicate.
    

---

10. **`kubelet` -> API Server -> `etcd`:**  
    Once the pod is successfully running and networked:
    

- The `kubelet` updates the pod’s status (e.g., `status.phase: Running`) by sending a status update to the API server.
    
- The API server validates and writes this new status back to **etcd**.
    

At this point, the pod’s _desired state_ (one running replica) matches the _actual state_ in the cluster.

---

[![](https://mermaid.ink/img/pako:eNqFVMlu2zAQ_RWCJwewDS2OFwENUDg9GEUDw25QoPCFEccSG4lUuWSp4X_vaLEj20qiEzWcN-_Nxh2NFQcaUQN_HcgYbgVLNMs3kuBXMG1FLAomLbk3oC-tj-4BYptdXnxdLtagn7owYGN-ab2FIlOvOUg7V9JqlWVd2BV6iZit4UOvdZwCd51X31FwBvbyYr5adBjv0Fiby_wHNzdNwtEhcxJrYBYIP8onKWSZGjwrnXEyGIicJfBFJkK-1IEaIMY61igia5C8VQKSMym2YCzprb6tfxJddsfYqzrCEYcxymIi3CoNbbx6-IMk5-5t_S3nOgV-yLSrE6dq53XOb814h-9UXod7DehqaifhUvEGSnrGMutMRJZYOSGTT0rzhjyQHmfklOkXs3FKtkoTJ03jwkmhuPkQtqxgyDI0BcRDiUt1x3LsOx7896TdF_w8q2dhU3KKvjoobkb3Pb2lRsKMEYlExVaRTm5EN3GiirgDcM6Gm4G-LstINcuE4ahi9TUODraLCXnYsxbkblHOtHVFKYtIsLgOj9gm0pOYmClYDH3yBDbtk8Xy4wxbVap7jjvhpPy8564C8hZyI2mfJlpwGm1ZZqBPc9A5K__prgy2oTaFHDY0wiOHLXOZ3dCN3CMOH4TfSuU0stohUiuXpMc4NVfzeB5dcDRBz5WTlkb-eFrFoNGOvuCvNxnO_CAMrz0vmEzCmd-nrzQKgnA48qf-bDIZzUZhcH2979N_Fa03HE_DkT8KR57nz4JgjAjgAlP9UT_h1Uu-_w9jqPf8?type=png)
