
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

---
### Logs

```bash
vishalkumar_shaw@IN-8KBKXD3:~/openairinterface5g$ ls
CHANGELOG.md       CONTRIBUTING.md  README.md   cmake_targets  docker       nfapi     openair2   pre-commit-clang  tools
CMakeLists.txt     LICENSE          charts      common         executables  oaienv    openair3   radio
CMakePresets.json  NOTICE.md        ci-scripts  doc            maketags     openair1  openshift  targets
vishalkumar_shaw@IN-8KBKXD3:~/openairinterface5g$ nano ~/openairinterface5g/Dockerfile.gnb
vishalkumar_shaw@IN-8KBKXD3:~/openairinterface5g$ nano ~/openairinterface5g/Dockerfile.nrue
vishalkumar_shaw@IN-8KBKXD3:~/openairinterface5g$
vishalkumar_shaw@IN-8KBKXD3:~/openairinterface5g$
vishalkumar_shaw@IN-8KBKXD3:~/openairinterface5g$
vishalkumar_shaw@IN-8KBKXD3:~/openairinterface5g$ docker build -f Dockerfile.gnb -t vishalcalsoftinc/oai-gnb:develop .
[+] Building 753.3s (15/15) FINISHED                                                                                  docker:default
 => [internal] load build definition from Dockerfile.gnb                                                                        0.0s
 => => transferring dockerfile: 1.41kB                                                                                          0.0s
 => [internal] load metadata for docker.io/library/ubuntu:24.04                                                                 2.2s
 => [auth] library/ubuntu:pull token for registry-1.docker.io                                                                   0.0s
 => [internal] load .dockerignore                                                                                               0.0s
 => => transferring context: 193B                                                                                               0.0s
 => [internal] load build context                                                                                               6.5s
 => => transferring context: 561.49MB                                                                                           6.4s
 => CACHED [builder 1/6] FROM docker.io/library/ubuntu:24.04@sha256:440dcf6a5640b2ae5c77724e68787a906afb8ddee98bf86db94eea8528  0.0s
 => [stage-1 2/4] RUN apt-get update && apt-get install -y     libsctp1     libconfig9     libfftw3-double3     libssl3     i  23.3s
 => [builder 2/6] RUN apt-get update && apt-get install -y     git     wget     cmake     ninja-build     build-essential     178.0s
 => [builder 3/6] COPY . /opt/oai                                                                                               3.6s
 => [builder 4/6] WORKDIR /opt/oai                                                                                              0.1s
 => [builder 5/6] RUN cd cmake_targets && ./build_oai -I --ninja                                                               79.7s
 => [builder 6/6] RUN cd cmake_targets && ./build_oai -w SIMU --gNB --build-e2 --ninja                                        458.6s
 => [stage-1 3/4] COPY --from=builder /opt/oai/cmake_targets/ran_build/build /opt/oai/                                          8.2s
 => [stage-1 4/4] WORKDIR /opt/oai                                                                                              0.1s
 => exporting to image                                                                                                          6.9s
 => => exporting layers                                                                                                         6.9s
 => => writing image sha256:917baec5342341ed08b05854acab4ba8439f1fc22ad888df878792f3d4952b09                                    0.0s
 => => naming to docker.io/vishalcalsoftinc/oai-gnb:develop                                                                     0.0s
vishalkumar_shaw@IN-8KBKXD3:~/openairinterface5g$
vishalkumar_shaw@IN-8KBKXD3:~/openairinterface5g$
vishalkumar_shaw@IN-8KBKXD3:~/openairinterface5g$ docker build -f Dockerfile.nrue -t vishalcalsoftinc/oai-nrue:develop .
[+] Building 378.0s (15/15) FINISHED                                                                                  docker:default
 => [internal] load build definition from Dockerfile.nrue                                                                       0.0s
 => => transferring dockerfile: 1.26kB                                                                                          0.0s
 => [internal] load metadata for docker.io/library/ubuntu:24.04                                                                 3.4s
 => [auth] library/ubuntu:pull token for registry-1.docker.io                                                                   0.0s
 => [internal] load .dockerignore                                                                                               0.0s
 => => transferring context: 193B                                                                                               0.0s
 => [internal] load build context                                                                                               0.2s
 => => transferring context: 272.06kB                                                                                           0.2s
 => CACHED [builder 1/6] FROM docker.io/library/ubuntu:24.04@sha256:440dcf6a5640b2ae5c77724e68787a906afb8ddee98bf86db94eea8528  0.0s
 => [stage-1 2/4] RUN apt-get update && apt-get install -y     libsctp1     libconfig9     libfftw3-double3     libssl3     i  27.4s
 => CACHED [builder 2/6] RUN apt-get update && apt-get install -y     git     wget     cmake     ninja-build     build-essenti  0.0s
 => CACHED [builder 3/6] COPY . /opt/oai                                                                                        0.0s
 => CACHED [builder 4/6] WORKDIR /opt/oai                                                                                       0.0s
 => CACHED [builder 5/6] RUN cd cmake_targets && ./build_oai -I --ninja                                                         0.0s
 => [builder 6/6] RUN cd cmake_targets && ./build_oai -w SIMU --nrUE --ninja                                                  343.8s
 => [stage-1 3/4] COPY --from=builder /opt/oai/cmake_targets/ran_build/build /opt/oai/                                          9.3s
 => [stage-1 4/4] WORKDIR /opt/oai                                                                                              0.1s
 => exporting to image                                                                                                          9.9s
 => => exporting layers                                                                                                         9.8s
 => => writing image sha256:3f842e86a392e2af4526d2f5a2de789bddb3069605e84d3731b510bf1ff47970                                    0.0s
 => => naming to docker.io/vishalcalsoftinc/oai-nrue:develop                                                                    0.0s
vishalkumar_shaw@IN-8KBKXD3:~/openairinterface5g$
vishalkumar_shaw@IN-8KBKXD3:~/openairinterface5g$
vishalkumar_shaw@IN-8KBKXD3:~/openairinterface5g$
vishalkumar_shaw@IN-8KBKXD3:~/openairinterface5g$ docker push vishalcalsoftinc/oai-gnb:develop
The push refers to repository [docker.io/vishalcalsoftinc/oai-gnb]
5f70bf18a086: Mounted from vishalcalsoftinc/quiz-app
bc1d856562ae: Pushed
6ab9215ba1f9: Pushed
45a01f98e78c: Mounted from library/ubuntu
develop: digest: sha256:4438b3f25e974aa182b2dcdc18ca6be2bf57edb3467487c27ef80d1162a7bdb6 size: 1159
vishalkumar_shaw@IN-8KBKXD3:~/openairinterface5g$
vishalkumar_shaw@IN-8KBKXD3:~/openairinterface5g$
vishalkumar_shaw@IN-8KBKXD3:~/openairinterface5g$ docker push vishalcalsoftinc/oai-nrue:develop
The push refers to repository [docker.io/vishalcalsoftinc/oai-nrue]
5f70bf18a086: Mounted from vishalcalsoftinc/oai-gnb
fecda858abcd: Pushed
69ea210718c4: Pushed
45a01f98e78c: Mounted from vishalcalsoftinc/oai-gnb
develop: digest: sha256:51b3b90fb7065d594517a38d529b1dd5a26373fa28d0f4d7dc07aa9065c1f5d4 size: 1159
vishalkumar_shaw@IN-8KBKXD3:~/openairinterface5g$
```
