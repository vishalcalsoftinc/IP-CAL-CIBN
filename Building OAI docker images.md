
---
You've hit another common and important hurdle in the world of cloud-native development. You are absolutely right, the public images are proving unreliable. This is a perfect opportunity to take full control of your environment.

Building the images yourself is an excellent skill. It makes you independent of public repositories and allows you to customize the OAI software in the future.

Here is a complete, step-by-step guide to build the OAI RAN images directly on your `oai-k8s-vm` and make them available to your Kubernetes cluster.

---

### **High-Level Plan**

1.  **Install Docker Engine:** We need Docker to build the container images.
2.  **Get OAI Source Code:** We'll clone the official OAI repository.
3.  **Create Dockerfiles:** We will write two "recipe" files (`Dockerfile.gnb` and `Dockerfile.nrue`) that tell Docker exactly how to build our images.
4.  **Build the Images:** We'll run the `docker build` command.
5.  **Load Images into Kubernetes:** This is a crucial step. We need to move the images from Docker's storage into `containerd`'s storage, which is what Kubernetes uses. This bypasses the need for a remote registry.
6.  **Update and Deploy:** We'll update our `oai-ran-deployments.yaml` to use our new, locally-built images.

---

### **Execution: Building the OAI Images**

Perform all the following steps inside your `oai-k8s-vm` after logging in with `vagrant ssh oai-k8s-vm`.

#### Step 1: Install Docker Engine

Your VM has `containerd`, but not the full Docker engine needed for the `docker build` command. Let's install it.

```bash
# Install Docker prerequisites
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg

# Add Docker's official GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Set up the repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# Install Docker Engine
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Add your user to the 'docker' group to avoid using sudo for docker commands
sudo usermod -aG docker ${USER}

# IMPORTANT: You need to exit and re-enter the SSH session for the group change to take effect
echo "Docker installed. Please run 'exit' and then 'vagrant ssh oai-k8s-vm' again."
```

Now, **exit the VM and SSH back in**:

```bash
exit
vagrant ssh oai-k8s-vm
```

Verify that Docker is working by running `docker ps`. You should see an empty table without any permission errors.

#### Step 2: Get the OAI Source Code

We'll clone the source code into your home directory.

```bash
cd ~
git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git
```

#### Step 3: Create the Dockerfiles

We need two separate recipes.

**First, create the Dockerfile for the gNB components (CU/DU):**

```bash
nano ~/openairinterface5g/Dockerfile.gnb
```

Paste the following content into the file. The comments explain each step.

```dockerfile
# Dockerfile for OAI gNB components (CU-CP, CU-UP, DU)

# Use Ubuntu 20.04 as the base, matching our VM OS
FROM ubuntu:20.04

# Avoid interactive prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive

# Install all build dependencies for OAI
RUN apt-get update && \
    apt-get install -y \
    build-essential \
    cmake \
    git \
    libsctp-dev \
    libconfig-dev \
    libuhd-dev \
    libfftw3-dev \
    librtlsdr-dev \
    libncurses5-dev \
    libtool \
    autoconf \
    automake \
    libssl-dev \
    iperf \
    # Add any other dependencies OAI build script might need

# Copy the source code from our host into the image
# The build context will be the 'openairinterface5g' directory
COPY . /opt/oai

# Navigate to the build directory
WORKDIR /opt/oai/cmake_targets

# Compile the gNB and install it within the image
# We are building for simulation (-w SIMU) and including gNB components
RUN ./build_oai -I --gNB -w SIMU --ninja
```

Save and exit `nano` (Ctrl+O, Enter, Ctrl+X).

**Second, create the Dockerfile for the nr-UE component:**

```bash
nano ~/openairinterface5g/Dockerfile.nrue
```

Paste this content. It's very similar, but it builds the `--nrUE` target.

```dockerfile
# Dockerfile for OAI nr-UE component

# Use Ubuntu 20.04 as the base
FROM ubuntu:20.04

ENV DEBIAN_FRONTEND=noninteractive

# Install the same build dependencies
RUN apt-get update && \
    apt-get install -y \
    build-essential \
    cmake \
    git \
    libsctp-dev \
    libconfig-dev \
    libuhd-dev \
    libfftw3-dev \
    librtlsdr-dev \
    libncurses5-dev \
    libtool \
    autoconf \
    automake \
    libssl-dev \
    iperf

# Copy the source code
COPY . /opt/oai

# Navigate to the build directory
WORKDIR /opt/oai/cmake_targets

# Compile the nrUE and install it within the image
RUN ./build_oai -I --nrUE -w SIMU --ninja```

Save and exit `nano`.

#### Step 4: Build the Docker Images

Now we run the build commands. This will take a significant amount of time (15-30 minutes or more per image) as it is compiling the entire OAI project.

```bash
# Navigate to the source code directory which is our build context
cd ~/openairinterface5g

# Build the gNB image and tag it as 'oai-gnb:custom'
echo "Building gNB image... this will take a while."
docker build -f Dockerfile.gnb -t oai-gnb:custom .

# Build the UE image and tag it as 'oai-nrue:custom'
echo "Building nr-UE image... this will take a while."
docker build -f Dockerfile.nrue -t oai-nrue:custom .
```

After the builds complete, verify the images exist in Docker's local storage:

```bash
docker images
```
You should see `oai-gnb` and `oai-nrue` with the tag `custom` in the list.

#### Step 5: Make the Images Available to Kubernetes (`containerd`)

This is the key step to avoid needing a registry. We export the image from Docker and import it directly into `containerd`.

```bash
# Export the gNB image from Docker and import into containerd's 'k8s.io' namespace
echo "Importing gNB image into Kubernetes..."
docker save oai-gnb:custom | sudo ctr -n k8s.io images import -

# Export the UE image from Docker and import into containerd
echo "Importing UE image into Kubernetes..."
docker save oai-nrue:custom | sudo ctr -n k8s.io images import -
```

#### Step 6: Update Kubernetes Deployments and Redeploy

Finally, we edit our deployment file to use our new `custom` images.

First, ensure any old failed pods are gone:
```bash
kubectl delete -f ~/oai-ran-deployments.yaml
```

Now, edit the file:
```bash
nano ~/oai-ran-deployments.yaml
```

You need to:
1.  **Change the image names:**
    *   For `oai-cucp-deployment`, `oai-cuup-deployment`, and `oai-du-deployment`, change the image to `oai-gnb:custom`.
    *   For `oai-nrue-deployment`, change the image to `oai-nrue:custom`.
2.  **Add an `imagePullPolicy`:** This is critical. We must tell Kubernetes **not** to try and pull our custom image from the internet. Add `imagePullPolicy: IfNotPresent` to the container spec for all four deployments.

Your CU-CP deployment spec should look like this (the other gNB ones will be similar):
```yaml
      containers:
        - name: oai-cucp
          image: oai-gnb:custom         # <-- CHANGED
          imagePullPolicy: IfNotPresent # <-- ADDED
          securityContext:
            privileged: true
          # ... rest of the spec
```

Your NR-UE deployment spec should look like this:
```yaml
      containers:
        - name: oai-nrue
          image: oai-nrue:custom        # <-- CHANGED
          imagePullPolicy: IfNotPresent # <-- ADDED
          securityContext:
            privileged: true
          # ... rest of the spec
```

After saving the changes, re-apply the manifest:

```bash
kubectl apply -f ~/oai-ran-deployments.yaml
```

#### Step 7: Final Verification

Now, watch the pods one last time.

```bash
watch kubectl get pods -n oai
```

Because the images are already present locally in `containerd`, the pods should go from `Pending` to `ContainerCreating` to `Running` very quickly. There will be no download time.

If they all become `Running`, you have successfully built your own OAI images and deployed them! You can now proceed to check the logs and then move on to deploying the Open5GS core.