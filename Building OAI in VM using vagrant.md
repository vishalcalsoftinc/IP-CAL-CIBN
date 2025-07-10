
---
Of course. Migrating a complex setup like OpenAirInterface to a VM using Vagrant is an excellent idea for reproducibility, isolation, and easy cleanup.

Here is the rewritten guide, starting with a new prerequisite section that defines and uses a `Vagrantfile` to automate the environment setup.

***

# OpenAirInterface 5G Complete Setup Guide (with Vagrant & VirtualBox)

This guide will walk you through setting up a complete OpenAirInterface (OAI) 5G RAN simulation environment inside a dedicated Virtual Machine managed by Vagrant and VirtualBox. Using a VM ensures a clean, reproducible, and isolated workspace.

## Prerequisites: Your Host Machine

Before you begin, you need to install Vagrant and VirtualBox on your main computer (the "host").

- **VirtualBox**: A free virtualization tool. [Download VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- **Vagrant**: A tool for building and managing virtual machine environments. [Download Vagrant](https://www.vagrantup.com/downloads)

Install both tools for your operating system (Windows, macOS, or Linux).

## Step 1: Create and Launch the OAI Virtual Machine

We will use a `Vagrantfile` to define our VM's specifications and automatically install all the necessary dependencies.

1.  Create a new folder for your project on your host machine and navigate into it.

    ```bash
    mkdir oai-vagrant
    cd oai-vagrant
    ```

2.  Inside this folder, create a file named `Vagrantfile` and paste the following content into it.

    ```ruby
    # Vagrantfile
    Vagrant.configure("2") do |config|
      # Use the official Ubuntu 20.04 (Focal Fossa) 64-bit box
      config.vm.box = "ubuntu/focal64"
    
      # Configure the VirtualBox provider with system requirements
      config.vm.provider "virtualbox" do |vb|
        # Set a friendly name for the VM in the VirtualBox GUI
        vb.name = "oai-vm"
    
        # Allocate RAM. 8GB is minimum, 16GB is recommended for better performance.
        vb.memory = "8192" # 8192 MB = 8 GB
    
        # Allocate CPU cores. 4+ is recommended.
        vb.cpus = "4"
      end
    
      # Automate the installation of all required tools using a shell provisioner.
      # This runs once when the VM is first created with 'vagrant up'.
      config.vm.provision "shell", inline: <<-SHELL
        echo ">>> Starting provisioning..."
        
        # Prevent interactive prompts during installation
        export DEBIAN_FRONTEND=noninteractive
        
        # Update system package list
        apt-get update
        
        # Upgrade existing packages
        apt-get upgrade -y
        
        # Install all essential build tools and dependencies for OAI
        apt-get install -y git cmake build-essential pkg-config \
                           libfftw3-dev libmbedtls-dev libsctp-dev \
                           libpcap-dev ninja-build \
                           libcap-dev libatlas-base-dev
                           
        echo ">>> Provisioning complete. OAI dependencies are installed."
      SHELL
    end
    ```

3.  From your terminal, inside the `oai-vagrant` folder, start the virtual machine.

    ```bash
    vagrant up
    ```
    This command will download the Ubuntu 20.04 image (if you don't have it), create the VM with the specified RAM/CPU, and run the provisioning script to install all the tools. This may take several minutes on the first run.

## Step 2: Connect to the VM and Clone OAI

Once `vagrant up` is finished, your VM is running and ready.

1.  Connect to the VM using a secure shell (SSH).

    ```bash
    vagrant ssh
    ```
    You are now inside the Ubuntu 20.04 VM. The following commands are all run *inside* the VM.

2.  Clone the OpenAirInterface repository.

    ```bash
    # Clone the main OAI repository into a directory named 'oai'
    git clone https://gitlab.eurecom.fr/oai/openairinterface5g oai
    cd oai/
    
    # Optional: Switch to the 'develop' branch for the latest features
    # git checkout develop
    ```

## Step 3: Build OAI Components

The build script will compile the gNodeB and nrUE executables for simulation.

### Build Command Breakdown

Your build command includes these components:

-   `-I`: Install OAI-specific dependencies (managed by the build script).
-   `-w SIMU`: Build for RF simulation mode (no radio hardware required).
-   `--gNB`: Build the gNodeB (5G base station).
-   `--nrUE`: Build the 5G User Equipment (UE).
-   `--build-e2`: Build E2 interface components (for RIC integration).
-   `--ninja`: Use the Ninja build system for a faster compilation.

### Execute the Build

This process will take a significant amount of time, depending on your host machine's performance.

```bash
# Navigate to the build script directory
cd ~/oai/cmake_targets/

# Run the complete build command
# ./build_oai -I -w SIMU --gNB --nrUE --build-e2 --ninja

# ---- OR ----
# Alternative: Build step-by-step for easier debugging
./build_oai -I                    # Install dependencies first

 # this package is installed in build_oai -I and conflicts and needs to be removed
 # if using ubuntu 24.04 , this step is not required
 sudo apt remove -y libyaml-cpp-dev && sudo apt autoremove -y
 
 
./build_oai -w SIMU --gNB --ninja # Build gNB
# this is failing due 
# -- AVX512 intrinsics are ON
# -- AVX2 intrinsics are ON

./build_oai -w SIMU --nrUE --ninja # Build nrUE
./build_oai --build-e2 --ninja    # Build E2 components
```

### Expected Build Output

The build process will create executables in:
- `ran_build/build/nr-softmodem` (the gNodeB executable)
- `ran_build/build/nr-uesoftmodem` (the UE executable)

## Step 4: Prepare Configuration Files

We need separate configuration files for the gNodeB and the UE.

### gNodeB Configuration

```bash
# Navigate to the configuration directory
cd ~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF/

# Copy the template gNB configuration file
cp gnb.sa.band78.fr1.106PRB.usrpb210.conf my_gnb.conf

# You can edit the file if needed, but the default should work for simulation
# nano my_gnb.conf
```

### UE Configuration

```bash
# In the same configuration directory
# cd ~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF/

# Copy the template UE configuration file
cp ue.conf my_ue.conf

# You can edit the file if needed
# nano my_ue.conf
```

## Step 5: Run the 5G Simulation

To run the simulation, you need two separate terminals connected to the same VM.

-   **Terminal 1**: Will run the gNodeB.
-   **Terminal 2**: Will run the UE.

On your **host machine**, open a **second terminal window**, navigate to your `oai-5g-vagrant` directory, and run `vagrant ssh`. You now have two separate SSH sessions into your VM.

### Terminal 1: Start the gNodeB

In your first SSH session, run the following:

```bash
cd ~/oai/cmake_targets/ran_build/build/

# Run the gNodeB in simulation mode with your config file
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/my_gnb.conf --rfsim --sa
```
The gNodeB will start and wait for a UE to connect.

### Terminal 2: Start the UE

In your second SSH session, run the following:

```bash
cd ~/oai/cmake_targets/ran_build/build/

# Run the UE in simulation mode
# Parameters: -r (PRBs), --numerology, --band, -C (frequency), etc.
sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --rfsim --sa --nokrnmod 1
```

After a few moments, you should see logs in both terminals indicating that the UE has found the gNodeB and successfully established a connection (look for messages like `[UE] RACH Succeeded` and `[gNB] UE ... connected`).

## Managing Your VM

Here are some useful Vagrant commands to manage your VM from your host machine's terminal (in the `oai-5g-vagrant` directory):

-   `vagrant ssh`: Connect to the VM.
-   `vagrant halt`: Shut down the VM gracefully.
-   `vagrant resume`: Start a previously halted VM.
-   `vagrant destroy -f`: **Delete the VM completely.** This is useful if you want to start over from scratch. The next `vagrant up` will create a brand new, clean VM.