
---
### **High-Level Plan**

1.  **Install Docker Engine:** Ensure Docker is ready for building.
2.  **Get OAI Source Code:** Clone the official OAI repository.
3.  **Create Optimized Dockerfiles:** Write two separate, efficient "recipe" files: `Dockerfile.gnb` and `Dockerfile.nrue`.
4.  **Build the Images:** Run the `docker build` command for each Dockerfile.
5. **Push Images to Docker Hub:** Log in and upload your newly built images.

---

### **Execution: Building the OAI Images**

#### Step 1: Install Docker Engine (If Not Already Done)

If you have not already done so, install the Docker engine. If you've already completed this from the previous guide, you can skip to Step 2.


#### Step 2: Get the OAI Source Code

We'll clone the source code. If you already have it, you can skip this step.

```bash
cd ~
git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git
# Optional: checkout the develop branch if you prefer
# cd openairinterface5g && git checkout develop && cd ..
```

#### Step 3: Create the Dockerfiles

This is where we implement your requested change. We will create two separate, optimized Dockerfiles inside the `openairinterface5g` directory.

**First, create the Dockerfile for the gNB (CU/DU):**

```bash
nano ~/openairinterface5g/Dockerfile.gnb
```

Paste the following content. This is a refined version of your file, which builds *only* the gNB components.

```dockerfile
# Dockerfile for OAI gNB (CU-CP, CU-UP, DU)
# Using a multi-stage build to keep the final image clean

# --- Build Stage ---
FROM ubuntu:24.04 AS builder

# Set environment variables to non-interactive
ENV DEBIAN_FRONTEND=noninteractive

# Install build dependencies and tools
RUN apt-get update && apt-get install -y \
    git \
    wget \
    cmake \
    ninja-build \
    build-essential \
    python3-pip \
    software-properties-common \
    && rm -rf /var/lib/apt/lists/*

# Copy the OAI source code from the build context
COPY . /opt/oai
WORKDIR /opt/oai

# Install OAI-specific dependencies using the build script
RUN cd cmake_targets && ./build_oai -I --ninja

# Build gNB and E2 interface components
RUN cd cmake_targets && ./build_oai -w SIMU --gNB --build-e2 --ninja

# --- Final Stage ---
FROM ubuntu:24.04

# Set environment variables
ENV DEBIAN_FRONTEND=noninteractive

# Install only the runtime dependencies for OAI
# This creates a much smaller final image
RUN apt-get update && apt-get install -y \
    libsctp1 \
    libconfig9 \
    libfftw3-double3 \
    libssl3 \
    iperf3 \
    net-tools \
    iputils-ping \
    && rm -rf /var/lib/apt/lists/*

# Copy the compiled binaries and libraries from the builder stage
COPY --from=builder /opt/oai/cmake_targets/ran_build/build /opt/oai/
WORKDIR /opt/oai

# Set default command
CMD ["/bin/bash"]
```

Save and exit `nano` (Ctrl+O, Enter, Ctrl+X).

**Second, create the Dockerfile for the nr-UE:**

```bash
nano ~/openairinterface5g/Dockerfile.nrue
```

Paste this content. Note how it is nearly identical, but the final build step targets `--nrUE` instead of `--gNB`.

```dockerfile
# Dockerfile for OAI nr-UE
# Using a multi-stage build to keep the final image clean

# --- Build Stage ---
FROM ubuntu:24.04 AS builder

# Set environment variables to non-interactive
ENV DEBIAN_FRONTEND=noninteractive

# Install build dependencies and tools
RUN apt-get update && apt-get install -y \
    git \
    wget \
    cmake \
    ninja-build \
    build-essential \
    python3-pip \
    software-properties-common \
    && rm -rf /var/lib/apt/lists/*

# Copy the OAI source code from the build context
COPY . /opt/oai
WORKDIR /opt/oai

# Install OAI-specific dependencies using the build script
RUN cd cmake_targets && ./build_oai -I --ninja

# Build nrUE components
RUN cd cmake_targets && ./build_oai -w SIMU --nrUE --ninja

# --- Final Stage ---
FROM ubuntu:24.04

# Install only the runtime dependencies for OAI
RUN apt-get update && apt-get install -y \
    libsctp1 \
    libconfig9 \
    libfftw3-double3 \
    libssl3 \
    iperf3 \
    net-tools \
    iputils-ping \
    && rm -rf /var/lib/apt/lists/*

# Copy the compiled binaries and libraries from the builder stage
COPY --from=builder /opt/oai/cmake_targets/ran_build/build /opt/oai/
WORKDIR /opt/oai

# Set default command
CMD ["/bin/bash"]
```

Save and exit `nano`.

#### Step 4: Build the Docker Images

Now, let's build the two separate images. This will take a significant amount of time, as it compiles the OAI project twice.

```bash
# Navigate to the source code directory, which is our build context
cd ~/openairinterface5g

# Build the gNB image and tag it as 'oai-gnb:custom'
echo "Building gNB image... this will take a while."
docker build -f Dockerfile.gnb -t vishalcalsoftinc/oai-gnb:develop .

# Build the nr-UE image and tag it as 'oai-nrue:custom'
echo "Building nr-UE image... this will take a while."
docker build -f Dockerfile.nrue -t vishalcalsoftinc/oai-nrue:develop .
```

After the builds complete, verify the images exist: `docker images`. 

#### Step 5: Push the Images to Docker Hub

This is the new step. You need to authenticate with Docker Hub and then push the images.

**5.1: Log in to Docker Hub**

It is highly recommended to use a **Personal Access Token (PAT)** instead of your password.

1. Go to [hub.docker.com](https://www.google.com/url?sa=E&q=https%3A%2F%2Fhub.docker.com) -> Account Settings -> Security -> New Access Token.    
2. Create a token with Read, Write, Delete permissions.    
3. Copy the token immediately.
    

Now, log in from your terminal.

```bash
# Use your Docker Hub username
docker login -u vishalcalsoftinc
```

You will be prompted for your password. **Paste your Access Token here.** You should see a "Login Succeeded" message.

**5.2: Push the Images**

```bash
# Push the gNB image
echo "Pushing gNB image to Docker Hub..."
docker push vishalcalsoftinc/oai-gnb:develop

# Push the nr-UE image
echo "Pushing nr-UE image to Docker Hub..."
docker push vishalcalsoftinc/oai-nrue:develop
```

You can now visit your Docker Hub profile online to see your new oai-gnb and oai-nrue repositories.

