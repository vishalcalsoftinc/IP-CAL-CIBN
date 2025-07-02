
---
Yes, you absolutely can, and for many, it's a much faster and more lightweight way to do it!

Running this lab in WSL (Windows Subsystem for Linux) is an excellent choice. Instead of using Vagrant and VirtualBox to create full virtual machines, we will use a modern tool called **`kind` (Kubernetes in Docker)**. `kind` runs each Kubernetes "node" as a Docker container, which is a perfect fit for the WSL2 environment.

Here is a detailed guide tailored specifically for WSL2 + Ubuntu.

### **What's Different from the Vagrant Guide?**

*   **No VirtualBox/Vagrant:** We will use Docker Desktop and `kind`.
*   **"Nodes" are Containers:** Your Kubernetes master and worker nodes will be Docker containers running on your WSL2 instance.
*   **"SSHing into a Node":** Instead of `vagrant ssh`, you will use `docker exec` to get a shell inside a node container.
*   **Cleanup:** Instead of `vagrant destroy`, you will use `kind delete cluster`.

---

### **Part 0: Prerequisites for WSL2**

1.  **Install WSL2 and a Linux Distro:** Ensure you have WSL2 enabled and a distribution like Ubuntu installed from the Microsoft Store. If you're using `wsl --version`, you already have this.

2.  **Install Docker Desktop for Windows:**
    *   Download and install it from [docker.com](https://www.docker.com/products/docker-desktop).
    *   During setup, make sure it is configured to use the **WSL2 based engine**.
    *   In Docker Desktop settings (`Settings > Resources > WSL Integration`), ensure that integration is **enabled** for your Ubuntu distribution. This allows you to run `docker` commands from within your WSL Ubuntu terminal.

3.  **Open your WSL Ubuntu Terminal:** All subsequent commands will be run from here.

---

### **Part 1: Setting up a 2-Node Cluster with `kind`**

1.  **Install `kubectl`:**
    ```bash
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    chmod +x ./kubectl
    sudo mv ./kubectl /usr/local/bin/kubectl
    ```

2.  **Install `kind`:**
    ```bash
    curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.14.0/kind-linux-amd64
    chmod +x ./kind
    sudo mv ./kind /usr/local/bin/kind
    ```

3.  **Create a `kind` Cluster Configuration File:** This is the most important step. We need to tell `kind` to *not* install its default network plugin (so we can install Calico) and to configure the pod IP range that Calico will use.

    *   Create a file named `kind-calico.yaml`:
        ```bash
        nano kind-calico.yaml
        ```
    *   Paste the following content into it:
        ```yaml
        # This config file tells kind to create a cluster with 1 control-plane and 1 worker node.
        # It also disables the default CNI plugin and sets the pod IP range for Calico.
        kind: Cluster
        apiVersion: kind.x-k8s.io/v1alpha4
        networking:
          disableDefaultCNI: true # Important!
          podSubnet: "10.244.0.0/16" # Important! Must match Calico's default
        nodes:
        - role: control-plane
        - role: worker
        ```
    *   Save and exit (`Ctrl+X`, then `Y`, then `Enter`).

4.  **Create the Kubernetes Cluster:** This command will use the configuration file to create two Docker containers (one for the master, one for the worker) and set them up as a Kubernetes cluster.
    ```bash
    kind create cluster --name calico-lab --config kind-calico.yaml
    ```
    This will take a minute or two. Once done, your `kubectl` is automatically configured to talk to this new cluster.

5. Other changes :
	```
	# Changing the context
	kubectl config get-contexts
	kubectl config use-context kind-calico-lab
	
	# backing up minikube's congfig and creating new config
	mv ~/.kube/config ~/.kube/config.backup
	kind get kubeconfig --name calico-lab > ~/.kube/config 
	
	# remove alias on kubectl
	```
6.  **Verify Cluster Status:** Check the nodes. Just like with Vagrant, they will be in a `NotReady` state because the CNI is missing.
    ```bash
    kubectl get nodes
    ```
    Output:
    ```
    NAME                         STATUS     ROLES           AGE   VERSION
    calico-lab-control-plane     NotReady   control-plane   64s   v1.24.0
    calico-lab-worker            NotReady   <none>          33s   v1.24.0
    ```

---

### **Part 2: Installing Calico**

This is exactly the same as before.

1.  **Apply the Calico manifest:**
    ```bash
    kubectl apply -f https://projectcalico.docs.tigera.io/manifests/calico.yaml
    ```

2.  **Verify the Nodes are Ready:** Wait about a minute for the Calico pods to start, then check the nodes again. They should now be `Ready`.
    ```bash
    kubectl get nodes
    ```
    Output:
    ```
    NAME                         STATUS   ROLES           AGE   VERSION
    calico-lab-control-plane     Ready    control-plane   3m    v1.24.0
    calico-lab-worker            Ready    <none>          2m    v1.24.0
    ```

---

### **Part 3: Deploying the Sample Application**

Again, this is exactly the same. We'll deploy the Nginx application.

1.  **Create the `nginx-app.yaml` file** (you can copy it from the previous guide or use `nano` to create it again).

2.  **Apply the Manifest:**
    ```bash
    kubectl apply -f nginx-app.yaml
    ```

3.  **Verify the Application:** Check that pods are running on both "nodes" (which are now containers).
    ```bash
    kubectl get pods -o wide
    ```

---

### **Part 4: Exploring the Default Calico Setup (IP-in-IP)**

Here's how we "get inside" a node.

1.  **List the Node Containers:** First, find the names of your node containers.
    ```bash
    docker ps
    ```
    Output will look like:
    ```
    CONTAINER ID   IMAGE                  COMMAND                  CREATED         STATUS         PORTS                       NAMES
    c1a2b3c4d5e6   kindest/node:v1.24.0   "/usr/local/bin/entry…"   5 minutes ago   Up 5 minutes   127.0.0.1:32769->6443/tcp   calico-lab-control-plane
    f6e5d4c3b2a1   kindest/node:v1.24.0   "/usr/local/bin/entry…"   5 minutes ago   Up 5 minutes                               calico-lab-worker
    ```
    Our worker node is named `calico-lab-worker`.

2.  **Get a Shell Inside the Worker Node Container:**
    ```bash
    docker exec -it calico-lab-worker /bin/bash
    ```
    Your terminal prompt will change, indicating you are now inside the container.

3.  **Run the Analysis Commands (inside the container):**
    *   **Check Interfaces:**
        ```bash
        ip addr
        ```
        Look for the `cali...` interfaces and the `tunnel0` interface.
    *   **Check Routes:**
        ```bash
        ip route
        ```
        Confirm that the route to the other node's pod subnet goes via `dev tunnel0`.

4.  **Exit the container shell when you're done:**
    ```bash
    exit
    ```

---

### **Part 5: Changing to Non-Overlay (Direct) Mode**

1.  **Install `calicoctl` (in your main WSL Ubuntu terminal, not inside a container):**
    ```bash
    curl -L https://github.com/projectcalico/calico/releases/download/v3.25.0/calicoctl-linux-amd64 -o calicoctl
    chmod +x calicoctl
    sudo mv calicoctl /usr/local/bin/
    ```

2.  **Get, Modify, and Apply the IP Pool Config:** These steps are identical to the previous guide.
    ```bash
    # Get the config
    calicoctl get ippool default-ipv4-ippool -o yaml > pool.yaml

    # Modify the config to disable ipipMode
    sed -i 's/ipipMode: Always/ipipMode: Never/' pool.yaml

    # Apply the new config
    calicoctl apply -f pool.yaml
    ```

3.  **VERIFY THE CHANGE:**
    *   Get a shell inside the worker node container again:
        ```bash
        docker exec -it calico-lab-worker /bin/bash
        ```
    *   Check the routing table inside the container:
        ```bash
        ip route
        ```
    *   You will see that the route to the remote pod subnet now goes via `dev eth0` (the container's main virtual network interface) instead of `dev tunnel0`. This confirms the overlay is off!
    *   `exit` the container when done.

---

### **Part 6: Cleanup**

This is the easiest part. To delete your entire Kubernetes cluster and all its associated containers:

```bash
kind delete cluster --name calico-lab
```

You have now successfully completed the entire lab using only WSL2, Docker Desktop, and `kind`, demonstrating a very powerful and modern workflow for local Kubernetes development.