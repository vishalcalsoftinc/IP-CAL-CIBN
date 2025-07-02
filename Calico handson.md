
---
Of course! This is an excellent way to solidify your understanding. Here is a detailed, step-by-step guide to replicate the Calico setup and analysis shown in the videos.

This guide will walk you through:
1.  Setting up a 2-node Kubernetes cluster.
2.  Installing Calico as the CNI.
3.  Deploying a sample application.
4.  Analyzing the default **IP-in-IP** overlay network.
5.  Changing the configuration to use a **non-overlay (direct)** network.
6.  Verifying the changes.

---

### **Part 0: Prerequisites**

Before you begin, you need a local environment to create virtual machines. This guide uses Vagrant and VirtualBox, which automate the process of creating and configuring VMs.

1.  **Install VirtualBox:** Download and install it from [virtualbox.org](https://www.virtualbox.org/wiki/Downloads).
2.  **Install Vagrant:** Download and install it from [vagrantup.com](https://www.vagrantup.com/downloads).
3.  **Install `kubectl`:** Follow the official Kubernetes documentation to install the `kubectl` command-line tool on your local machine.

---

### **Part 1: Setting up a 2-Node Kubernetes Cluster**

We'll create one master node and one worker node.

1.  **Create a Project Directory:** On your computer, create a new folder for this project and navigate into it.
    ```bash
    mkdir calico-lab
    cd calico-lab
    ```

2.  **Create a `Vagrantfile`:** Inside this directory, create a file named `Vagrantfile` and paste the following content into it. This file tells Vagrant how to create our two virtual machines.

    ```ruby
    # -*- mode: ruby -*-
    # vi: set ft=ruby :

    Vagrant.configure("2") do |config|
      config.vm.box = "ubuntu/focal64"
      config.vm.provider "virtualbox" do |vb|
        vb.memory = "2048"
        vb.cpus = "2"
      end

      # Master Node
      config.vm.define "master" do |master|
        master.vm.hostname = "master"
        master.vm.network "private_network", ip: "192.168.56.10"
      end

      # Worker Node
      config.vm.define "node1" do |node|
        node.vm.hostname = "node1"
        node.vm.network "private_network", ip: "192.168.56.11"
      end
    end
    ```

3.  **Start the Virtual Machines:** Run the following command. It will download the Ubuntu image (if you don't have it) and create the two VMs. This may take a few minutes.
    ```bash
    vagrant up
    ```

4.  **Install Kubernetes on Both Nodes:** We need to SSH into each machine and run a setup script to install `kubeadm`, `kubelet`, and a container runtime (containerd).

    *   **SSH into the master node:**
        ```bash
        vagrant ssh master
        ```
    *   **Inside the master node, run these commands:**
        ```bash
        # Run all commands as root
        sudo su -

        # Install required packages
        apt-get update && apt-get install -y apt-transport-https curl
        curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
        cat <<EOF | tee /etc/apt/sources.list.d/kubernetes.list
        deb https://apt.kubernetes.io/ kubernetes-xenial main
        EOF
        apt-get update

        # Install containerd, kubeadm, kubelet, kubectl
        apt-get install -y containerd kubelet=1.23.5-00 kubeadm=1.23.5-00 kubectl=1.23.5-00
        apt-mark hold kubelet kubeadm kubectl

        # Configure containerd
        mkdir -p /etc/containerd
        containerd config default | tee /etc/containerd/config.toml
        sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
        systemctl restart containerd

        # Configure Kubernetes networking prerequisites
        cat <<EOF | tee /etc/sysctl.d/k8s.conf
        net.bridge.bridge-nf-call-ip6tables = 1
        net.bridge.bridge-nf-call-iptables = 1
        net.ipv4.ip_forward = 1
        EOF
        sysctl --system

        # Exit from root and from the SSH session
        exit
        exit
        ```
    *   **Repeat the exact same steps for the worker node (`node1`):**
        ```bash
        vagrant ssh node1
        # --- Run the same block of commands from `sudo su -` to `sysctl --system` ---
        exit
        exit
        ```

5.  **Initialize the Cluster on the Master Node:**
    *   SSH back into the master node: `vagrant ssh master`
    *   Initialize the cluster using `kubeadm`. The CIDR `10.244.0.0/16` is a common choice for Calico.
        ```bash
        sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.56.10
        ```
    *   After it finishes, it will print a `kubeadm join` command. **Copy this entire command**, you will need it for the worker node.
    *   It will also tell you to run three commands to set up `kubectl` for your user. Run them now:
        ```bash
        mkdir -p $HOME/.kube
        sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
        sudo chown $(id -u):$(id -g) $HOME/.kube/config
        ```

6.  **Join the Worker Node to the Cluster:**
    *   SSH into the worker node: `vagrant ssh node1`
    *   Paste the `kubeadm join` command you copied from the master node. You'll need to add `sudo` at the beginning. It will look something like this:
        ```bash
        sudo kubeadm join 192.168.56.10:6443 --token ... --discovery-token-ca-cert-hash ...
        ```

7.  **Verify Cluster Status:**
    *   Go back to your **master node's** SSH session.
    *   Check the nodes. They will be in a `NotReady` state because the network plugin (CNI) isn't installed yet. This is expected.
        ```bash
        kubectl get nodes
        ```
        Output should be similar to:
        ```
        NAME     STATUS     ROLES                  AGE   VERSION
        master   NotReady   control-plane,master   5m    v1.23.5
        node1    NotReady   <none>                 2m    v1.23.5
        ```

---

### **Part 2: Installing Calico**

1.  **On the master node**, apply the Calico manifest. This will install all the necessary Calico components.
    ```bash
    kubectl apply -f https://projectcalico.docs.tigera.io/manifests/calico.yaml
    ```

2.  **Verify the Installation:** Watch the Calico pods start up in the `kube-system` namespace.
    ```bash
    kubectl get pods -n kube-system -w
    ```
    Wait until `calico-node` is running on both nodes and `calico-kube-controllers` is running. Press `Ctrl+C` to exit the watch.

3.  **Check the Nodes Again:** Now that the CNI is running, the nodes should transition to the `Ready` state.
    ```bash
    kubectl get nodes
    ```
    Output:
    ```
    NAME     STATUS   ROLES                  AGE   VERSION
    master   Ready    control-plane,master   8m    v1.23.5
    node1    Ready    <none>                 5m    v1.23.5
    ```

Your cluster is now ready with Calico installed!

---

### **Part 3: Deploying a Sample Application**

We'll deploy a simple Nginx web server with 4 replicas to ensure pods are scheduled on both nodes.

1.  **On the master node**, create a file named `nginx-app.yaml`.
    ```bash
    nano nginx-app.yaml
    ```

2.  Paste the following YAML into the file:
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
    spec:
      replicas: 4
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - name: nginx
            image: nginx
            ports:
            - containerPort: 80
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: nginx-service
    spec:
      type: NodePort
      selector:
        app: nginx
      ports:
      - protocol: TCP
        port: 80
        targetPort: 80
        nodePort: 30080
    ```
    Save and exit (`Ctrl+X`, then `Y`, then `Enter`).

3.  **Apply the Manifest:**
    ```bash
    kubectl apply -f nginx-app.yaml
    ```

4.  **Verify the Application:** Check that the pods are running and distributed across both nodes. The `-o wide` flag is crucial as it shows pod IPs and node names.
    ```bash
    kubectl get pods -o wide
    ```
    You should see 4 pods, some on `master` and some on `node1`, each with a unique IP from the `10.244.x.x` range.

---

### **Part 4: Exploring the Default Calico Setup (IP-in-IP)**

Let's replicate the analysis from the video. We will perform these steps on `node1`.

1.  **SSH into the worker node:**
    ```bash
    # If you're not already there
    vagrant ssh node1
    ```

2.  **Check Network Interfaces:**
    ```bash
    ip addr
    ```
    Look for these key interfaces:
    *   **`cali...` interfaces:** You'll see one for each Nginx pod running on `node1`. These are the `veth` pairs connecting the pods to the host. Notice they don't have an IP on the host side.
    *   **`tunnel0` interface:** This is the key interface for the IP-in-IP overlay. Note its IP address (e.g., `10.244.195.128`).

3.  **Check the Routing Table:** This is the most important step.
    ```bash
    ip route
    ```
    Analyze the output:
    *   **Routes to local pods:** You will see a route for each pod on `node1`.
      `10.244.195.130 dev calixxxxxxx scope link`
      This means traffic to this local pod IP is sent directly to its specific `veth` device (`calixxxxxxx`).
    *   **Route to remote pods:** You will see a route for the entire subnet of the *other* node (the master node).
      `10.244.75.192/26 via 192.168.56.10 dev tunnel0 proto bird`
      **This line is critical:**
        *   `10.244.75.192/26`: The pod subnet on the `master` node.
        *   `via 192.168.56.10`: The traffic is forwarded to the master node's IP.
        *   `dev tunnel0`: The traffic is sent through the **IP-in-IP tunnel**.
        *   `proto bird`: This route was learned via the BGP protocol.

---

### **Part 5: Changing to Non-Overlay (Direct) Mode**

Now we will disable the IP-in-IP overlay, which is possible because our nodes are on the same L2 subnet (`192.168.56.0/24`).

1.  **Install `calicoctl` on the master node:**
    *   Go to your **master node's** SSH session.
    *   Download and install the `calicoctl` binary.
    ```bash
    curl -L https://github.com/projectcalico/calico/releases/download/v3.24.1/calicoctl-linux-amd64 -o calicoctl
    chmod +x calicoctl
    sudo mv calicoctl /usr/local/bin/
    ```

2.  **Get the IP Pool Configuration:**
    ```bash
    calicoctl get ippool default-ipv4-pool -o yaml > pool.yaml
    ```

3.  **View and Modify the Configuration:**
    *   Look at the file to see the default setting:
        ```bash
        cat pool.yaml
        ```
        You will see `ipipMode: Always`.
    *   Now, disable IP-in-IP by changing `Always` to `Never`.
        ```bash
        sed -i 's/ipipMode: Always/ipipMode: Never/' pool.yaml
        ```

4.  **Apply the New Configuration:**
    ```bash
    calicoctl apply -f pool.yaml
    ```
    You should see `Successfully applied 1 'IPPool' resource(s).`

5.  **VERIFY THE CHANGE:** This is the most important part.
    *   Go back to your **worker node's** SSH session (`node1`).
    *   Check the routing table again.
        ```bash
        ip route
        ```
    *   Look for the route to the remote pod subnet. It will have changed!
      **Before:** `10.244.75.192/26 via 192.168.56.10 dev tunnel0 proto bird`
      **After:** `10.244.75.192/26 via 192.168.56.10 dev eth1 proto bird`
      Notice that `dev tunnel0` has been replaced with `dev eth1` (your node's main network interface). The `tunnel0` interface may even disappear entirely from the `ip addr` output.

**This proves that you have successfully disabled the overlay network. Calico now sends pod traffic directly between the nodes without encapsulation.**

---

### **Part 6: Cleanup**

When you are finished with the lab, you can destroy the virtual machines to free up resources on your computer.

1.  From your `calico-lab` directory on your local machine (not inside an SSH session), run:
    ```bash
    vagrant destroy -f
    ```

This guide provides a complete, hands-on experience that mirrors the concepts and commands shown in the videos, helping you build a solid, practical understanding of how Calico works.