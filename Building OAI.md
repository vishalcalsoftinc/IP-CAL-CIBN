
---
# OpenAirInterface 5G Complete Setup Guide

## Prerequisites

### System Requirements

- **OS**: Ubuntu 20.04 LTS 
- **RAM**: Minimum 8GB, recommended 16GB+
- **Storage**: At least 50GB free space
- **CPU**: Multi-core processor (4+ cores recommended)
- **Network**: Stable internet connection for dependencies

### Required Tools

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install essential build tools
sudo apt install -y git cmake build-essential pkg-config
sudo apt install -y libfftw3-dev libmbedtls-dev libsctp-dev
sudo apt install -y libyaml-cpp-dev libpcap-dev ninja-build
sudo apt install -y libcap-dev  
sudo apt install -y libatlas-base-dev
```

## Step 1: Clone the Repository

```bash
# Clone the main OAI repository
git clone https://gitlab.eurecom.fr/oai/openairinterface5g oai
cd oai/

# Check available branches (optional)
# git branch -a

# Switch to develop branch for latest features (recommended)
# git checkout develop
```

## Step 2: Build OAI Components

### Build Command Breakdown

Your build command includes these components:

- `-I`: Install dependencies
- `-w SIMU`: Build for simulation mode
- `--gNB`: Build gNodeB (5G base station)
- `--nrUE`: Build 5G User Equipment
- `--build-e2`: Build E2 interface components
- `--ninja`: Use Ninja build system (faster)

### Execute the Build

```bash
# Navigate to build directory
cd ~/oai/cmake_targets/

# Run the complete build command
./build_oai -I -w SIMU --gNB --nrUE --build-e2 --ninja

# Alternative: Build step by step for debugging
./build_oai -I                    # Install dependencies first
./build_oai -w SIMU --gNB --ninja # Build gNB
./build_oai -w SIMU --nrUE --ninja # Build nrUE
./build_oai --build-e2 --ninja    # Build E2 components
```

### Expected Build Output

The build process will create executables in:

- `ran_build/build/`: Main build directory
- `ran_build/build/nr-softmodem`: gNodeB executable
- `ran_build/build/nr-uesoftmodem`: UE executable

## Step 3: Configuration Files

### gNodeB Configuration

```bash
# Navigate to configuration directory
cd ~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF/

# Copy and modify gNB configuration
cp gnb.sa.band78.fr1.106PRB.usrpb210.conf my_gnb.conf

# Edit configuration file
nano my_gnb.conf
```

### UE Configuration

```bash
# Navigate to configuration directory
cd ~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF/

# Copy UE configuration
cp ue.conf my_ue.conf

# Edit UE configuration
nano my_ue.conf
```

## Step 4: Testing 

### Terminal 2: Start gNodeB

bash

```bash
cd ~/oai/cmake_targets/ran_build/build/

# Run gNodeB in simulation mode
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/my_gnb.conf --rfsim --sa
```

### Terminal 3: Start UE

bash

```bash
cd ~/oai/cmake_targets/ran_build/build/

# Run UE in simulation mode
sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --rfsim --sa --nokrnmod 1
```

![[Pasted image 20250709131327.png]]


---
## Logs
### Commands I ran
Of course. Here is a list of all the commands you ran, extracted from your terminal session log:

#### System Setup & Updates
1.  `# Update system`
2.  `sudo apt update && sudo apt upgrade -y`
3.  `# Install essential build tools`
4.  `sudo apt install -y git cmake build-essential pkg-config`
5.  `sudo apt install -y libfftw3-dev libmbedtls-dev libsctp-dev`
6.  `sudo apt install -y libyaml-cpp-dev libpcap-dev ninja-build`

#### Cloning and Building OpenAirInterface5G
7.  `# Clone the main OAI repository`
8.  `git clone https://gitlab.eurecom.fr/oai/openairinterface5g oai`
9.  `cd oai/`
10. `# Navigate to build scripts`
11. `cd cmake_targets/`
12. `# Execute : Build step by step for debugging`
13. `./build_oai -I                    # Install dependencies first`
14. `./build_oai -w SIMU --gNB --ninja # Build gNB` (This command was interrupted with `^C`)

#### Installing Missing Dependencies
15. `sudo apt install -y libcap-dev libcblas-dev`
16. `sudo apt install -y libatlas-base-dev`
17. `sudo apt install -y libcap-dev`

#### Resuming the Build Process
18. `./build_oai -w SIMU --gNB --ninja # Build gNB`
19. `./build_oai -w SIMU --nrUE --ninja # Build nrUE`
20. `./build_oai --build-e2 --ninja    # Build E2 components`

#### Configuration
21. `# Navigate to configuration directory`
22. `cd ~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF/`
23. `ls`
24. `# Copy and modify gNB configuration`
25. `cp gnb.sa.band78.fr1.106PRB.usrpb210.conf my_gnb.conf`
26. `nano my_gnb.conf`
27. `# Copy UE configuration`
28. `cp ue.conf my_ue.conf`
29. `nano my_gnb.conf`
30. `rm my_gnb.conf`
31. `cp gnb.sa.band78.fr1.106PRB.usrpb210.conf my_gnb.conf`

#### System and Process Checks
32. `# Check if gNodeB is running`
33. `ps aux | grep nr-softmodem`
34. `# Check network interfaces`
35. `ip addr show`
36. `# Check UE status`
37. `ps aux | grep nr-uesoftmodem`
38. `# Check if UE interface is created`
39. `ip addr show oaitun_ue1`
40. `# Ping test from UE`
41. `ping -I oaitun_ue1 8.8.8.8`

### Building

```bash
vishalkumar_shaw@IN-8KBKXD3:~$ # Update system
sudo apt update && sudo apt upgrade -y
[sudo] password for vishalkumar_shaw:
Hit:1 https://download.docker.com/linux/ubuntu noble InRelease
Hit:2 http://archive.ubuntu.com/ubuntu noble InRelease
Get:3 http://archive.ubuntu.com/ubuntu noble-updates InRelease [126 kB]
Get:4 http://security.ubuntu.com/ubuntu noble-security InRelease [126 kB]
Get:5 http://archive.ubuntu.com/ubuntu noble-backports InRelease [126 kB]
Get:6 http://security.ubuntu.com/ubuntu noble-security/main amd64 Packages [983 kB]
Get:7 http://security.ubuntu.com/ubuntu noble-security/main Translation-en [174 kB]
Get:8 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages [1234 kB]
Get:9 http://security.ubuntu.com/ubuntu noble-security/main amd64 Components [21.6 kB]
Get:10 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 Components [162 kB]
Get:11 http://security.ubuntu.com/ubuntu noble-security/universe amd64 Packages [869 kB]
Get:12 http://archive.ubuntu.com/ubuntu noble-updates/universe amd64 Packages [1104 kB]
Get:13 http://security.ubuntu.com/ubuntu noble-security/universe Translation-en [191 kB]
Get:14 http://archive.ubuntu.com/ubuntu noble-updates/universe Translation-en [281 kB]
Get:15 http://security.ubuntu.com/ubuntu noble-security/universe amd64 Components [52.3 kB]
Get:16 http://archive.ubuntu.com/ubuntu noble-updates/universe amd64 Components [377 kB]
Get:17 http://archive.ubuntu.com/ubuntu noble-updates/restricted amd64 Components [212 B]
Get:18 http://archive.ubuntu.com/ubuntu noble-updates/multiverse amd64 Components [940 B]
Get:19 http://security.ubuntu.com/ubuntu noble-security/restricted amd64 Packages [1368 kB]
Get:20 http://archive.ubuntu.com/ubuntu noble-backports/main amd64 Packages [63.1 kB]
Get:21 http://security.ubuntu.com/ubuntu noble-security/restricted Translation-en [292 kB]
Get:22 http://archive.ubuntu.com/ubuntu noble-backports/main Translation-en [9152 B]
Get:23 http://security.ubuntu.com/ubuntu noble-security/restricted amd64 Components [212 B]
Get:24 http://archive.ubuntu.com/ubuntu noble-backports/main amd64 Components [7084 B]
Get:25 http://security.ubuntu.com/ubuntu noble-security/multiverse amd64 Components [212 B]
Get:26 http://archive.ubuntu.com/ubuntu noble-backports/universe amd64 Packages [31.4 kB]
Get:27 http://archive.ubuntu.com/ubuntu noble-backports/universe Translation-en [17.1 kB]
Get:28 http://archive.ubuntu.com/ubuntu noble-backports/universe amd64 Components [28.3 kB]
Get:29 http://archive.ubuntu.com/ubuntu noble-backports/restricted amd64 Components [216 B]
Get:30 http://archive.ubuntu.com/ubuntu noble-backports/multiverse amd64 Components [212 B]
Fetched 7647 kB in 9s (813 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
46 packages can be upgraded. Run 'apt list --upgradable' to see them.
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Calculating upgrade... Done
The following upgrades have been deferred due to phasing:
  ubuntu-pro-client ubuntu-pro-client-l10n
The following packages will be upgraded:
  alsa-ucm-conf apparmor apt apt-utils bsdextrautils bsdutils
  cloud-init distro-info-data docker-buildx-plugin docker-ce
  docker-ce-cli docker-ce-rootless-extras
  docker-compose-plugin eject fdisk git git-man
  gtk-update-icon-cache gzip ibverbs-providers libapparmor1
  libapt-pkg6.0t64 libasound2-data libasound2t64 libblkid1
  libfdisk1 libgtk-3-0t64 libgtk-3-bin libgtk-3-common
  libibverbs1 libmount1 libnetplan1 libpciaccess0
  libsmartcols1 libuuid1 mount netplan-generator netplan.io
  openssh-client python3-netplan python3-update-manager
  update-manager-core util-linux uuid-runtime
44 upgraded, 0 newly installed, 0 to remove and 2 not upgraded.
2 standard LTS security updates
Need to get 89.9 MB of archives.
After this operation, 12.0 MB disk space will be freed.
Get:1 https://download.docker.com/linux/ubuntu noble/stable amd64 docker-ce-cli amd64 5:28.3.1-1~ubuntu.24.04~noble [16.5 MB]
Get:2 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 bsdutils amd64 1:2.39.3-9ubuntu6.3 [95.9 kB]
Get:3 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 gzip amd64 1.12-1ubuntu3.1 [99.0 kB]
Get:4 https://download.docker.com/linux/ubuntu noble/stable amd64 docker-ce amd64 5:28.3.1-1~ubuntu.24.04~noble [19.7 MB]
Get:5 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 util-linux amd64 2.39.3-9ubuntu6.3 [1128 kB]
Get:6 https://download.docker.com/linux/ubuntu noble/stable amd64 docker-buildx-plugin amd64 0.25.0-1~ubuntu.24.04~noble [15.6 MB]
Get:7 https://download.docker.com/linux/ubuntu noble/stable amd64 docker-ce-rootless-extras amd64 5:28.3.1-1~ubuntu.24.04~noble [6479 kB]
Get:8 https://download.docker.com/linux/ubuntu noble/stable amd64 docker-compose-plugin amd64 2.38.1-1~ubuntu.24.04~noble [14.2 MB]
Get:9 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libapt-pkg6.0t64 amd64 2.8.3 [985 kB]
Get:10 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 apt amd64 2.8.3 [1376 kB]
Get:11 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 apt-utils amd64 2.8.3 [216 kB]
Get:12 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 mount amd64 2.39.3-9ubuntu6.3 [118 kB]
Get:13 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libsmartcols1 amd64 2.39.3-9ubuntu6.3 [65.4 kB]
Get:14 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libuuid1 amd64 2.39.3-9ubuntu6.3 [35.8 kB]
Get:15 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 uuid-runtime amd64 2.39.3-9ubuntu6.3 [33.1 kB]
Get:16 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libblkid1 amd64 2.39.3-9ubuntu6.3 [123 kB]
Get:17 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libmount1 amd64 2.39.3-9ubuntu6.3 [134 kB]
Get:18 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 distro-info-data all 0.60ubuntu0.3 [6686 B]
Get:19 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 eject amd64 2.39.3-9ubuntu6.3 [26.3 kB]
Get:20 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libapparmor1 amd64 4.0.1really4.0.1-0ubuntu0.24.04.4 [50.6 kB]
Get:21 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libfdisk1 amd64 2.39.3-9ubuntu6.3 [146 kB]
Get:22 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 netplan-generator amd64 1.1.2-2~ubuntu24.04.1 [61.1 kB]
Get:23 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 python3-netplan amd64 1.1.2-2~ubuntu24.04.1 [24.3 kB]
Get:24 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 netplan.io amd64 1.1.2-2~ubuntu24.04.1 [69.7 kB]
Get:25 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libnetplan1 amd64 1.1.2-2~ubuntu24.04.1 [132 kB]
Get:26 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 apparmor amd64 4.0.1really4.0.1-0ubuntu0.24.04.4 [637 kB]
Get:27 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 bsdextrautils amd64 2.39.3-9ubuntu6.3 [73.7 kB]
Get:28 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libibverbs1 amd64 50.0-2ubuntu0.2 [68.0 kB]
Get:29 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 ibverbs-providers amd64 50.0-2ubuntu0.2 [381 kB]
Get:30 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 git-man all 1:2.43.0-1ubuntu7.3 [1100 kB]
Get:31 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 git amd64 1:2.43.0-1ubuntu7.3 [3680 kB]
Get:32 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 openssh-client amd64 1:9.6p1-3ubuntu13.12 [905 kB]
Get:33 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 python3-update-manager all 1:24.04.12 [43.4 kB]
Get:34 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 update-manager-core all 1:24.04.12 [11.6 kB]
Get:35 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libasound2t64 amd64 1.2.11-1ubuntu0.1 [399 kB]
Get:36 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libasound2-data all 1.2.11-1ubuntu0.1 [21.1 kB]
Get:37 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 alsa-ucm-conf all 1.2.10-1ubuntu5.7 [66.4 kB]
Get:38 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 fdisk amd64 2.39.3-9ubuntu6.3 [122 kB]
Get:39 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 gtk-update-icon-cache amd64 3.24.41-4ubuntu1.3 [51.9 kB]
Get:40 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libgtk-3-common all 3.24.41-4ubuntu1.3 [1426 kB]
Get:41 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libgtk-3-0t64 amd64 3.24.41-4ubuntu1.3 [2913 kB]
Get:42 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libgtk-3-bin amd64 3.24.41-4ubuntu1.3 [73.9 kB]
Get:43 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libpciaccess0 amd64 0.17-3ubuntu0.24.04.2 [18.9 kB]
Get:44 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 cloud-init all 25.1.2-0ubuntu0~24.04.1 [607 kB]
Fetched 89.9 MB in 60s (1504 kB/s)
Extracting templates from packages: 100%
Preconfiguring packages ...
(Reading database ... 56809 files and directories currently installed.)
Preparing to unpack .../bsdutils_1%3a2.39.3-9ubuntu6.3_amd64.deb
 ...
Unpacking bsdutils (1:2.39.3-9ubuntu6.3) over (1:2.39.3-9ubuntu6.2) ...
Setting up bsdutils (1:2.39.3-9ubuntu6.3) ...
(Reading database ... 56809 files and directories currently installed.)
Preparing to unpack .../gzip_1.12-1ubuntu3.1_amd64.deb ...
Unpacking gzip (1.12-1ubuntu3.1) over (1.12-1ubuntu3) ...
Setting up gzip (1.12-1ubuntu3.1) ...
(Reading database ... 56809 files and directories currently installed.)
Preparing to unpack .../util-linux_2.39.3-9ubuntu6.3_amd64.deb ...
Unpacking util-linux (2.39.3-9ubuntu6.3) over (2.39.3-9ubuntu6.2) ...
Setting up util-linux (2.39.3-9ubuntu6.3) ...
fstrim.service is a disabled or a static unit not running, not starting it.
(Reading database ... 56809 files and directories currently installed.)
Preparing to unpack .../libapt-pkg6.0t64_2.8.3_amd64.deb ...
Unpacking libapt-pkg6.0t64:amd64 (2.8.3) over (2.7.14build2) ...
Setting up libapt-pkg6.0t64:amd64 (2.8.3) ...
(Reading database ... 56809 files and directories currently installed.)
Preparing to unpack .../archives/apt_2.8.3_amd64.deb ...
Unpacking apt (2.8.3) over (2.7.14build2) ...
Setting up apt (2.8.3) ...
(Reading database ... 56809 files and directories currently installed.)
Preparing to unpack .../apt-utils_2.8.3_amd64.deb ...
Unpacking apt-utils (2.8.3) over (2.7.14build2) ...
Preparing to unpack .../mount_2.39.3-9ubuntu6.3_amd64.deb ...
Unpacking mount (2.39.3-9ubuntu6.3) over (2.39.3-9ubuntu6.2) ...
Preparing to unpack .../libsmartcols1_2.39.3-9ubuntu6.3_amd64.deb ...
Unpacking libsmartcols1:amd64 (2.39.3-9ubuntu6.3) over (2.39.3-9
ubuntu6.2) ...
Setting up libsmartcols1:amd64 (2.39.3-9ubuntu6.3) ...
(Reading database ... 56809 files and directories currently installed.)
Preparing to unpack .../libuuid1_2.39.3-9ubuntu6.3_amd64.deb ...
Unpacking libuuid1:amd64 (2.39.3-9ubuntu6.3) over (2.39.3-9ubunt
u6.2) ...
Setting up libuuid1:amd64 (2.39.3-9ubuntu6.3) ...
(Reading database ... 56809 files and directories currently installed.)
Preparing to unpack .../uuid-runtime_2.39.3-9ubuntu6.3_amd64.deb
 ...
Unpacking uuid-runtime (2.39.3-9ubuntu6.3) over (2.39.3-9ubuntu6.2) ...
Preparing to unpack .../docker-ce-cli_5%3a28.3.1-1~ubuntu.24.04~noble_amd64.deb ...
Unpacking docker-ce-cli (5:28.3.1-1~ubuntu.24.04~noble) over (5:28.0.4-1~ubuntu.24.04~noble) ...
Preparing to unpack .../docker-ce_5%3a28.3.1-1~ubuntu.24.04~noble_amd64.deb ...
Unpacking docker-ce (5:28.3.1-1~ubuntu.24.04~noble) over (5:28.0.4-1~ubuntu.24.04~noble) ...
Preparing to unpack .../libblkid1_2.39.3-9ubuntu6.3_amd64.deb ...
Unpacking libblkid1:amd64 (2.39.3-9ubuntu6.3) over (2.39.3-9ubuntu6.2) ...
Setting up libblkid1:amd64 (2.39.3-9ubuntu6.3) ...
(Reading database ... 56810 files and directories currently installed.)
Preparing to unpack .../libmount1_2.39.3-9ubuntu6.3_amd64.deb ..
.
Unpacking libmount1:amd64 (2.39.3-9ubuntu6.3) over (2.39.3-9ubun
tu6.2) ...
Setting up libmount1:amd64 (2.39.3-9ubuntu6.3) ...
(Reading database ... 56810 files and directories currently installed.)
Preparing to unpack .../00-distro-info-data_0.60ubuntu0.3_all.de
b ...
Unpacking distro-info-data (0.60ubuntu0.3) over (0.60ubuntu0.2) ...
Preparing to unpack .../01-eject_2.39.3-9ubuntu6.3_amd64.deb ...
Unpacking eject (2.39.3-9ubuntu6.3) over (2.39.3-9ubuntu6.2) ...
Preparing to unpack .../02-libapparmor1_4.0.1really4.0.1-0ubuntu0.24.04.4_amd64.deb ...
Unpacking libapparmor1:amd64 (4.0.1really4.0.1-0ubuntu0.24.04.4) over (4.0.1really4.0.1-0ubuntu0.24.04.3) ...
Preparing to unpack .../03-libfdisk1_2.39.3-9ubuntu6.3_amd64.deb ...
Unpacking libfdisk1:amd64 (2.39.3-9ubuntu6.3) over (2.39.3-9ubuntu6.2) ...
Preparing to unpack .../04-netplan-generator_1.1.2-2~ubuntu24.04.1_amd64.deb ...
Adding 'diversion of /lib/systemd/system-generators/netplan to /lib/systemd/system-generators/netplan.usr-is-merged by netplan-g
enerator'
Unpacking netplan-generator (1.1.2-2~ubuntu24.04.1) over (1.1.1-
1~ubuntu24.04.1) ...
Preparing to unpack .../05-python3-netplan_1.1.2-2~ubuntu24.04.1_amd64.deb ...
Unpacking python3-netplan (1.1.2-2~ubuntu24.04.1) over (1.1.1-1~
ubuntu24.04.1) ...
Preparing to unpack .../06-netplan.io_1.1.2-2~ubuntu24.04.1_amd64.deb ...
Unpacking netplan.io (1.1.2-2~ubuntu24.04.1) over (1.1.1-1~ubuntu24.04.1) ...
Preparing to unpack .../07-libnetplan1_1.1.2-2~ubuntu24.04.1_amd64.deb ...
Unpacking libnetplan1:amd64 (1.1.2-2~ubuntu24.04.1) over (1.1.1-1~ubuntu24.04.1) ...
Preparing to unpack .../08-apparmor_4.0.1really4.0.1-0ubuntu0.24.04.4_amd64.deb ...
Unpacking apparmor (4.0.1really4.0.1-0ubuntu0.24.04.4) over (4.0.1really4.0.1-0ubuntu0.24.04.3) ...
Preparing to unpack .../09-bsdextrautils_2.39.3-9ubuntu6.3_amd64.deb ...
Unpacking bsdextrautils (2.39.3-9ubuntu6.3) over (2.39.3-9ubuntu6.2) ...
Preparing to unpack .../10-libibverbs1_50.0-2ubuntu0.2_amd64.deb ...
Unpacking libibverbs1:amd64 (50.0-2ubuntu0.2) over (50.0-2build2) ...
Preparing to unpack .../11-ibverbs-providers_50.0-2ubuntu0.2_amd64.deb ...
Unpacking ibverbs-providers:amd64 (50.0-2ubuntu0.2) over (50.0-2build2) ...
Preparing to unpack .../12-git-man_1%3a2.43.0-1ubuntu7.3_all.deb ...
Unpacking git-man (1:2.43.0-1ubuntu7.3) over (1:2.43.0-1ubuntu7.
2) ...
Preparing to unpack .../13-git_1%3a2.43.0-1ubuntu7.3_amd64.deb ...
Unpacking git (1:2.43.0-1ubuntu7.3) over (1:2.43.0-1ubuntu7.2) ...
Preparing to unpack .../14-openssh-client_1%3a9.6p1-3ubuntu13.12_amd64.deb ...
Unpacking openssh-client (1:9.6p1-3ubuntu13.12) over (1:9.6p1-3ubuntu13.11) ...
Preparing to unpack .../15-python3-update-manager_1%3a24.04.12_all.deb ...
Unpacking python3-update-manager (1:24.04.12) over (1:24.04.9) .
..
Preparing to unpack .../16-update-manager-core_1%3a24.04.12_all.deb ...
Unpacking update-manager-core (1:24.04.12) over (1:24.04.9) ...
Preparing to unpack .../17-libasound2t64_1.2.11-1ubuntu0.1_amd64.deb ...
Unpacking libasound2t64:amd64 (1.2.11-1ubuntu0.1) over (1.2.11-1build2) ...
Preparing to unpack .../18-libasound2-data_1.2.11-1ubuntu0.1_all.deb ...
Unpacking libasound2-data (1.2.11-1ubuntu0.1) over (1.2.11-1buil
d2) ...
Preparing to unpack .../19-alsa-ucm-conf_1.2.10-1ubuntu5.7_all.deb ...
Unpacking alsa-ucm-conf (1.2.10-1ubuntu5.7) over (1.2.10-1ubuntu5.4) ...
Preparing to unpack .../20-docker-buildx-plugin_0.25.0-1~ubuntu.24.04~noble_amd64.deb ...
Unpacking docker-buildx-plugin (0.25.0-1~ubuntu.24.04~noble) ove
r (0.22.0-1~ubuntu.24.04~noble) ...
Preparing to unpack .../21-docker-ce-rootless-extras_5%3a28.3.1-1~ubuntu.24.04~noble_amd64.deb ...
Unpacking docker-ce-rootless-extras (5:28.3.1-1~ubuntu.24.04~noble) over (5:28.0.4-1~ubuntu.24.04~noble) ...
Preparing to unpack .../22-docker-compose-plugin_2.38.1-1~ubuntu.24.04~noble_amd64.deb ...
Unpacking docker-compose-plugin (2.38.1-1~ubuntu.24.04~noble) ov
er (2.34.0-1~ubuntu.24.04~noble) ...
Preparing to unpack .../23-fdisk_2.39.3-9ubuntu6.3_amd64.deb ...
Unpacking fdisk (2.39.3-9ubuntu6.3) over (2.39.3-9ubuntu6.2) ...
Preparing to unpack .../24-gtk-update-icon-cache_3.24.41-4ubuntu1.3_amd64.deb ...
Unpacking gtk-update-icon-cache (3.24.41-4ubuntu1.3) over (3.24.41-4ubuntu1.2) ...
Preparing to unpack .../25-libgtk-3-common_3.24.41-4ubuntu1.3_all.deb ...
Unpacking libgtk-3-common (3.24.41-4ubuntu1.3) over (3.24.41-4ubuntu1.2) ...
Preparing to unpack .../26-libgtk-3-0t64_3.24.41-4ubuntu1.3_amd64.deb ...
Unpacking libgtk-3-0t64:amd64 (3.24.41-4ubuntu1.3) over (3.24.41-4ubuntu1.2) ...
Preparing to unpack .../27-libgtk-3-bin_3.24.41-4ubuntu1.3_amd64.deb ...
Unpacking libgtk-3-bin (3.24.41-4ubuntu1.3) over (3.24.41-4ubuntu1.2) ...
Preparing to unpack .../28-libpciaccess0_0.17-3ubuntu0.24.04.2_amd64.deb ...
Unpacking libpciaccess0:amd64 (0.17-3ubuntu0.24.04.2) over (0.17
-3build1) ...
Preparing to unpack .../29-cloud-init_25.1.2-0ubuntu0~24.04.1_all.deb ...
Unpacking cloud-init (25.1.2-0ubuntu0~24.04.1) over (24.4.1-0ubuntu0~24.04.2) ...
dpkg: warning: unable to delete old directory '/lib/systemd/system-generators': Directory not empty
dpkg: warning: unable to delete old directory '/lib/systemd/syst
em/sshd-keygen@.service.d': Directory not empty
Setting up gtk-update-icon-cache (3.24.41-4ubuntu1.3) ...
Setting up libpciaccess0:amd64 (0.17-3ubuntu0.24.04.2) ...
Setting up libibverbs1:amd64 (50.0-2ubuntu0.2) ...
Setting up libapparmor1:amd64 (4.0.1really4.0.1-0ubuntu0.24.04.4) ...
Setting up apt-utils (2.8.3) ...
Setting up bsdextrautils (2.39.3-9ubuntu6.3) ...
Setting up ibverbs-providers:amd64 (50.0-2ubuntu0.2) ...
Setting up distro-info-data (0.60ubuntu0.3) ...
Setting up openssh-client (1:9.6p1-3ubuntu13.12) ...
Setting up libnetplan1:amd64 (1.1.2-2~ubuntu24.04.1) ...
Setting up eject (2.39.3-9ubuntu6.3) ...
Setting up libasound2-data (1.2.11-1ubuntu0.1) ...
Setting up apparmor (4.0.1really4.0.1-0ubuntu0.24.04.4) ...
Setting up docker-buildx-plugin (0.25.0-1~ubuntu.24.04~noble) ...
Setting up libasound2t64:amd64 (1.2.11-1ubuntu0.1) ...
Setting up docker-compose-plugin (2.38.1-1~ubuntu.24.04~noble) ...
Setting up libfdisk1:amd64 (2.39.3-9ubuntu6.3) ...
Setting up docker-ce-cli (5:28.3.1-1~ubuntu.24.04~noble) ...
Setting up mount (2.39.3-9ubuntu6.3) ...
Setting up uuid-runtime (2.39.3-9ubuntu6.3) ...
uuidd.service is a disabled or a static unit not running, not starting it.
Setting up python3-update-manager (1:24.04.12) ...
Setting up python3-netplan (1.1.2-2~ubuntu24.04.1) ...
Setting up git-man (1:2.43.0-1ubuntu7.3) ...
Setting up docker-ce-rootless-extras (5:28.3.1-1~ubuntu.24.04~no
ble) ...
Setting up netplan-generator (1.1.2-2~ubuntu24.04.1) ...
Removing 'diversion of /lib/systemd/system-generators/netplan to /lib/systemd/system-generators/netplan.usr-is-merged by netplan
-generator'
Setting up libgtk-3-common (3.24.41-4ubuntu1.3) ...
Setting up alsa-ucm-conf (1.2.10-1ubuntu5.7) ...
Setting up fdisk (2.39.3-9ubuntu6.3) ...
Setting up netplan.io (1.1.2-2~ubuntu24.04.1) ...
Setting up docker-ce (5:28.3.1-1~ubuntu.24.04~noble) ...
Setting up git (1:2.43.0-1ubuntu7.3) ...
Setting up update-manager-core (1:24.04.12) ...
Setting up cloud-init (25.1.2-0ubuntu0~24.04.1) ...
Installing new version of config file /etc/cloud/templates/sources.list.debian.deb822.tmpl ...
Installing new version of config file /etc/cloud/templates/sources.list.ubuntu.deb822.tmpl ...
Processing triggers for install-info (7.1-3build2) ...
Processing triggers for libc-bin (2.39-0ubuntu8.4) ...
Processing triggers for rsyslog (8.2312.0-3ubuntu9.1) ...
Processing triggers for man-db (2.12.0-4build2) ...
Processing triggers for libglib2.0-0t64:amd64 (2.80.0-6ubuntu3.4) ...
Processing triggers for dbus (1.14.10-4ubuntu4.1) ...
Setting up libgtk-3-0t64:amd64 (3.24.41-4ubuntu1.3) ...
Setting up libgtk-3-bin (3.24.41-4ubuntu1.3) ...
Processing triggers for libc-bin (2.39-0ubuntu8.4) ...
vishalkumar_shaw@IN-8KBKXD3:~$
vishalkumar_shaw@IN-8KBKXD3:~$
vishalkumar_shaw@IN-8KBKXD3:~$ # Install essential build tools
sudo apt install -y git cmake build-essential pkg-config
sudo apt install -y libfftw3-dev libmbedtls-dev libsctp-dev
sudo apt install -y libyaml-cpp-dev libpcap-dev ninja-build
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
git is already the newest version (1:2.43.0-1ubuntu7.3).
build-essential is already the newest version (12.10ubuntu1).
build-essential set to manually installed.
The following additional packages will be installed:
  cmake-data libarchive13t64 libjsoncpp25 libpkgconf3
  librhash0 pkgconf pkgconf-bin
Suggested packages:
  cmake-doc cmake-format elpa-cmake-mode ninja-build lrzip
The following NEW packages will be installed:
  cmake cmake-data libarchive13t64 libjsoncpp25 libpkgconf3
  librhash0 pkg-config pkgconf pkgconf-bin
0 upgraded, 9 newly installed, 0 to remove and 2 not upgraded.
Need to get 14.0 MB of archives.
After this operation, 50.3 MB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libarchive13t64 amd64 3.7.2-2ubuntu0.5 [382 kB]
Get:2 http://archive.ubuntu.com/ubuntu noble/main amd64 libjsoncpp25 amd64 1.9.5-6build1 [82.8 kB]
Get:3 http://archive.ubuntu.com/ubuntu noble/main amd64 librhash0 amd64 1.4.3-3build1 [129 kB]
Get:4 http://archive.ubuntu.com/ubuntu noble/main amd64 cmake-data all 3.28.3-1build7 [2155 kB]
Get:5 http://archive.ubuntu.com/ubuntu noble/main amd64 cmake amd64 3.28.3-1build7 [11.2 MB]
Get:6 http://archive.ubuntu.com/ubuntu noble/main amd64 libpkgconf3 amd64 1.8.1-2build1 [30.7 kB]
Get:7 http://archive.ubuntu.com/ubuntu noble/main amd64 pkgconf-bin amd64 1.8.1-2build1 [20.7 kB]
Get:8 http://archive.ubuntu.com/ubuntu noble/main amd64 pkgconf amd64 1.8.1-2build1 [16.8 kB]
Get:9 http://archive.ubuntu.com/ubuntu noble/main amd64 pkg-config amd64 1.8.1-2build1 [7264 B]
Fetched 14.0 MB in 6s (2509 kB/s)
Selecting previously unselected package libarchive13t64:amd64.
(Reading database ... 56821 files and directories currently installed.)
Preparing to unpack .../0-libarchive13t64_3.7.2-2ubuntu0.5_amd64
.deb ...
Unpacking libarchive13t64:amd64 (3.7.2-2ubuntu0.5) ...
Selecting previously unselected package libjsoncpp25:amd64.
Preparing to unpack .../1-libjsoncpp25_1.9.5-6build1_amd64.deb .
..
Unpacking libjsoncpp25:amd64 (1.9.5-6build1) ...
Selecting previously unselected package librhash0:amd64.
Preparing to unpack .../2-librhash0_1.4.3-3build1_amd64.deb ...
Unpacking librhash0:amd64 (1.4.3-3build1) ...
Selecting previously unselected package cmake-data.
Preparing to unpack .../3-cmake-data_3.28.3-1build7_all.deb ...
Unpacking cmake-data (3.28.3-1build7) ...
Selecting previously unselected package cmake.
Preparing to unpack .../4-cmake_3.28.3-1build7_amd64.deb ...
Unpacking cmake (3.28.3-1build7) ...
Selecting previously unselected package libpkgconf3:amd64.
Preparing to unpack .../5-libpkgconf3_1.8.1-2build1_amd64.deb ...
Unpacking libpkgconf3:amd64 (1.8.1-2build1) ...
Selecting previously unselected package pkgconf-bin.
Preparing to unpack .../6-pkgconf-bin_1.8.1-2build1_amd64.deb ..
.
Unpacking pkgconf-bin (1.8.1-2build1) ...
Selecting previously unselected package pkgconf:amd64.
Preparing to unpack .../7-pkgconf_1.8.1-2build1_amd64.deb ...
Unpacking pkgconf:amd64 (1.8.1-2build1) ...
Selecting previously unselected package pkg-config:amd64.
Preparing to unpack .../8-pkg-config_1.8.1-2build1_amd64.deb ...
Unpacking pkg-config:amd64 (1.8.1-2build1) ...
Setting up libpkgconf3:amd64 (1.8.1-2build1) ...
Setting up libjsoncpp25:amd64 (1.9.5-6build1) ...
Setting up pkgconf-bin (1.8.1-2build1) ...
Setting up librhash0:amd64 (1.4.3-3build1) ...
Setting up cmake-data (3.28.3-1build7) ...
Setting up libarchive13t64:amd64 (3.7.2-2ubuntu0.5) ...
Setting up pkgconf:amd64 (1.8.1-2build1) ...
Setting up pkg-config:amd64 (1.8.1-2build1) ...
Setting up cmake (3.28.3-1build7) ...
Processing triggers for man-db (2.12.0-4build2) ...
Processing triggers for libc-bin (2.39-0ubuntu8.4) ...
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libfftw3-bin libfftw3-long3 libfftw3-quad3 libfftw3-single3
  libmbedcrypto7t64 libmbedtls14t64 libmbedx509-1t64 libsctp1
Suggested packages:
  libfftw3-doc libmbedtls-doc lksctp-tools
The following NEW packages will be installed:
  libfftw3-bin libfftw3-dev libfftw3-long3 libfftw3-quad3
  libfftw3-single3 libmbedcrypto7t64 libmbedtls-dev
  libmbedtls14t64 libmbedx509-1t64 libsctp-dev libsctp1
0 upgraded, 11 newly installed, 0 to remove and 2 not upgraded.
Need to get 5361 kB of archives.
After this operation, 27.2 MB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu noble/main amd64 libfftw3-long3 amd64 3.3.10-1ubuntu3 [374 kB]
Get:2 http://archive.ubuntu.com/ubuntu noble/main amd64 libfftw3-quad3 amd64 3.3.10-1ubuntu3 [658 kB]
Get:3 http://archive.ubuntu.com/ubuntu noble/main amd64 libfftw3-single3 amd64 3.3.10-1ubuntu3 [868 kB]
Get:4 http://archive.ubuntu.com/ubuntu noble/main amd64 libfftw3-bin amd64 3.3.10-1ubuntu3 [39.2 kB]
Get:5 http://archive.ubuntu.com/ubuntu noble/main amd64 libfftw3-dev amd64 3.3.10-1ubuntu3 [2372 kB]
Get:6 http://archive.ubuntu.com/ubuntu noble/universe amd64 libmbedcrypto7t64 amd64 2.28.8-1 [209 kB]
Get:7 http://archive.ubuntu.com/ubuntu noble/universe amd64 libmbedx509-1t64 amd64 2.28.8-1 [46.6 kB]
Get:8 http://archive.ubuntu.com/ubuntu noble/universe amd64 libmbedtls14t64 amd64 2.28.8-1 [82.2 kB]
Get:9 http://archive.ubuntu.com/ubuntu noble/universe amd64 libmbedtls-dev amd64 2.28.8-1 [651 kB]
Get:10 http://archive.ubuntu.com/ubuntu noble/main amd64 libsctp1 amd64 1.0.19+dfsg-2build1 [9146 B]
Get:11 http://archive.ubuntu.com/ubuntu noble/main amd64 libsctp-dev amd64 1.0.19+dfsg-2build1 [51.1 kB]
Fetched 5361 kB in 8s (678 kB/s)
Selecting previously unselected package libfftw3-long3:amd64.
(Reading database ... 60269 files and directories currently installed.)
Preparing to unpack .../00-libfftw3-long3_3.3.10-1ubuntu3_amd64.
deb ...
Unpacking libfftw3-long3:amd64 (3.3.10-1ubuntu3) ...
Selecting previously unselected package libfftw3-quad3:amd64.
Preparing to unpack .../01-libfftw3-quad3_3.3.10-1ubuntu3_amd64.
deb ...
Unpacking libfftw3-quad3:amd64 (3.3.10-1ubuntu3) ...
Selecting previously unselected package libfftw3-single3:amd64.
Preparing to unpack .../02-libfftw3-single3_3.3.10-1ubuntu3_amd6
4.deb ...
Unpacking libfftw3-single3:amd64 (3.3.10-1ubuntu3) ...
Selecting previously unselected package libfftw3-bin.
Preparing to unpack .../03-libfftw3-bin_3.3.10-1ubuntu3_amd64.de
b ...
Unpacking libfftw3-bin (3.3.10-1ubuntu3) ...
Selecting previously unselected package libfftw3-dev:amd64.
Preparing to unpack .../04-libfftw3-dev_3.3.10-1ubuntu3_amd64.de
b ...
Unpacking libfftw3-dev:amd64 (3.3.10-1ubuntu3) ...
Selecting previously unselected package libmbedcrypto7t64:amd64.
Preparing to unpack .../05-libmbedcrypto7t64_2.28.8-1_amd64.deb
...
Unpacking libmbedcrypto7t64:amd64 (2.28.8-1) ...
Selecting previously unselected package libmbedx509-1t64:amd64.
Preparing to unpack .../06-libmbedx509-1t64_2.28.8-1_amd64.deb .
..
Unpacking libmbedx509-1t64:amd64 (2.28.8-1) ...
Selecting previously unselected package libmbedtls14t64:amd64.
Preparing to unpack .../07-libmbedtls14t64_2.28.8-1_amd64.deb ..
.
Unpacking libmbedtls14t64:amd64 (2.28.8-1) ...
Selecting previously unselected package libmbedtls-dev:amd64.
Preparing to unpack .../08-libmbedtls-dev_2.28.8-1_amd64.deb ...
Unpacking libmbedtls-dev:amd64 (2.28.8-1) ...
Selecting previously unselected package libsctp1:amd64.
Preparing to unpack .../09-libsctp1_1.0.19+dfsg-2build1_amd64.de
b ...
Unpacking libsctp1:amd64 (1.0.19+dfsg-2build1) ...
Selecting previously unselected package libsctp-dev:amd64.
Preparing to unpack .../10-libsctp-dev_1.0.19+dfsg-2build1_amd64.deb ...
Unpacking libsctp-dev:amd64 (1.0.19+dfsg-2build1) ...
Setting up libfftw3-single3:amd64 (3.3.10-1ubuntu3) ...
Setting up libmbedcrypto7t64:amd64 (2.28.8-1) ...
Setting up libfftw3-long3:amd64 (3.3.10-1ubuntu3) ...
Setting up libfftw3-quad3:amd64 (3.3.10-1ubuntu3) ...
Setting up libsctp1:amd64 (1.0.19+dfsg-2build1) ...
Setting up libmbedx509-1t64:amd64 (2.28.8-1) ...
Setting up libfftw3-bin (3.3.10-1ubuntu3) ...
Setting up libsctp-dev:amd64 (1.0.19+dfsg-2build1) ...
Setting up libmbedtls14t64:amd64 (2.28.8-1) ...
Setting up libfftw3-dev:amd64 (3.3.10-1ubuntu3) ...
Setting up libmbedtls-dev:amd64 (2.28.8-1) ...
Processing triggers for libc-bin (2.39-0ubuntu8.4) ...
Processing triggers for man-db (2.12.0-4build2) ...
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libdbus-1-dev libibverbs-dev libnl-3-dev libnl-route-3-dev
  libpcap0.8-dev libyaml-cpp0.8
The following NEW packages will be installed:
  libdbus-1-dev libibverbs-dev libnl-3-dev libnl-route-3-dev
  libpcap-dev libpcap0.8-dev libyaml-cpp-dev libyaml-cpp0.8
  ninja-build
0 upgraded, 9 newly installed, 0 to remove and 2 not upgraded.
Need to get 1895 kB of archives.
After this operation, 8123 kB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libdbus-1-dev amd64 1.14.10-4ubuntu4.1 [190 kB]
Get:2 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libnl-3-dev amd64 3.7.0-0.3build1.1 [99.5 kB]
Get:3 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libnl-route-3-dev amd64 3.7.0-0.3build1.1 [216 kB]
Get:4 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libibverbs-dev amd64 50.0-2ubuntu0.2 [686 kB]
Get:5 http://archive.ubuntu.com/ubuntu noble/main amd64 libpcap0.8-dev amd64 1.10.4-4.1ubuntu3 [269 kB]
Get:6 http://archive.ubuntu.com/ubuntu noble/main amd64 libpcap-dev amd64 1.10.4-4.1ubuntu3 [3324 B]
Get:7 http://archive.ubuntu.com/ubuntu noble/main amd64 libyaml-cpp0.8 amd64 0.8.0+dfsg-6build1 [115 kB]
Get:8 http://archive.ubuntu.com/ubuntu noble/main amd64 libyaml-cpp-dev amd64 0.8.0+dfsg-6build1 [188 kB]
Get:9 http://archive.ubuntu.com/ubuntu noble/universe amd64 ninja-build amd64 1.11.1-2 [129 kB]
Fetched 1895 kB in 3s (593 kB/s)
Selecting previously unselected package libdbus-1-dev:amd64.
(Reading database ... 60545 files and directories currently installed.)
Preparing to unpack .../0-libdbus-1-dev_1.14.10-4ubuntu4.1_amd64
.deb ...
Unpacking libdbus-1-dev:amd64 (1.14.10-4ubuntu4.1) ...
Selecting previously unselected package libnl-3-dev:amd64.
Preparing to unpack .../1-libnl-3-dev_3.7.0-0.3build1.1_amd64.de
b ...
Unpacking libnl-3-dev:amd64 (3.7.0-0.3build1.1) ...
Selecting previously unselected package libnl-route-3-dev:amd64.
Preparing to unpack .../2-libnl-route-3-dev_3.7.0-0.3build1.1_am
d64.deb ...
Unpacking libnl-route-3-dev:amd64 (3.7.0-0.3build1.1) ...
Selecting previously unselected package libibverbs-dev:amd64.
Preparing to unpack .../3-libibverbs-dev_50.0-2ubuntu0.2_amd64.d
eb ...
Unpacking libibverbs-dev:amd64 (50.0-2ubuntu0.2) ...
Selecting previously unselected package libpcap0.8-dev:amd64.
Preparing to unpack .../4-libpcap0.8-dev_1.10.4-4.1ubuntu3_amd64
.deb ...
Unpacking libpcap0.8-dev:amd64 (1.10.4-4.1ubuntu3) ...
Selecting previously unselected package libpcap-dev:amd64.
Preparing to unpack .../5-libpcap-dev_1.10.4-4.1ubuntu3_amd64.deb ...
Unpacking libpcap-dev:amd64 (1.10.4-4.1ubuntu3) ...
Selecting previously unselected package libyaml-cpp0.8:amd64.
Preparing to unpack .../6-libyaml-cpp0.8_0.8.0+dfsg-6build1_amd64.deb ...
Unpacking libyaml-cpp0.8:amd64 (0.8.0+dfsg-6build1) ...
Selecting previously unselected package libyaml-cpp-dev:amd64.
Preparing to unpack .../7-libyaml-cpp-dev_0.8.0+dfsg-6build1_amd
64.deb ...
Unpacking libyaml-cpp-dev:amd64 (0.8.0+dfsg-6build1) ...
Selecting previously unselected package ninja-build.
Preparing to unpack .../8-ninja-build_1.11.1-2_amd64.deb ...
Unpacking ninja-build (1.11.1-2) ...
Setting up libyaml-cpp0.8:amd64 (0.8.0+dfsg-6build1) ...
Setting up libyaml-cpp-dev:amd64 (0.8.0+dfsg-6build1) ...
Setting up ninja-build (1.11.1-2) ...
Setting up libdbus-1-dev:amd64 (1.14.10-4ubuntu4.1) ...
Setting up libnl-3-dev:amd64 (3.7.0-0.3build1.1) ...
Setting up libnl-route-3-dev:amd64 (3.7.0-0.3build1.1) ...
Setting up libibverbs-dev:amd64 (50.0-2ubuntu0.2) ...
Processing triggers for libc-bin (2.39-0ubuntu8.4) ...
Processing triggers for man-db (2.12.0-4build2) ...
Processing triggers for sgml-base (1.31) ...
Setting up libpcap0.8-dev:amd64 (1.10.4-4.1ubuntu3) ...
Setting up libpcap-dev:amd64 (1.10.4-4.1ubuntu3) ...
vishalkumar_shaw@IN-8KBKXD3:~$
vishalkumar_shaw@IN-8KBKXD3:~$
vishalkumar_shaw@IN-8KBKXD3:~$ # Clone the main OAI repository
git clone https://gitlab.eurecom.fr/oai/openairinterface5g oai
cd oai/
Cloning into 'oai'...
warning: redirecting to https://gitlab.eurecom.fr/oai/openairinterface5g.git/
remote: Enumerating objects: 463536, done.
remote: Counting objects: 100% (6314/6314), done.
remote: Compressing objects: 100% (991/991), done.
remote: Total 463536 (delta 5841), reused 5513 (delta 5297), pack-reused 457222 (from 1)
Receiving objects: 100% (463536/463536), 433.30 MiB | 3.16 MiB/s, done.
Resolving deltas: 100% (384520/384520), done.
vishalkumar_shaw@IN-8KBKXD3:~/oai$
vishalkumar_shaw@IN-8KBKXD3:~/oai$
vishalkumar_shaw@IN-8KBKXD3:~/oai$ # Navigate to build scripts
cd cmake_targets/
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets$
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets$
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets$
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets$ # Execute : Build step by step for debugging
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets$ ./build_oai -I                    # Install dependencies first
Will install external packages
OPENAIR_DIR    = /home/vishalkumar_shaw/oai
Installing packages
Hit:1 https://download.docker.com/linux/ubuntu noble InRelease
Hit:2 http://security.ubuntu.com/ubuntu noble-security InRelease
Hit:3 http://archive.ubuntu.com/ubuntu noble InRelease
Hit:4 http://archive.ubuntu.com/ubuntu noble-updates InRelease
Hit:5 http://archive.ubuntu.com/ubuntu noble-backports InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
software-properties-common is already the newest version (0.99.49.2).
software-properties-common set to manually installed.
0 upgraded, 0 newly installed, 0 to remove and 2 not upgraded.
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
build-essential is already the newest version (12.10ubuntu1).
cmake is already the newest version (3.28.3-1build7).
ninja-build is already the newest version (1.11.1-2).
pkg-config is already the newest version (1.8.1-2build1).
git is already the newest version (1:2.43.0-1ubuntu7.3).
libsctp-dev is already the newest version (1.0.19+dfsg-2build1).
patch is already the newest version (2.7.6-7build3).
patch set to manually installed.
openssl is already the newest version (3.0.13-0ubuntu3.5).
openssl set to manually installed.
zlib1g-dev is already the newest version (1:1.3.dfsg-3.1ubuntu2.1).
zlib1g-dev set to manually installed.
xxd is already the newest version (2:9.1.0016-1ubuntu7.8).
xxd set to manually installed.
libyaml-cpp-dev is already the newest version (0.8.0+dfsg-6build1).
The following additional packages will be installed:
  autoconf autotools-dev libblas3 libconfig-doc libconfig9
  libgfortran5 liblapack3 liblapacke libltdl-dev
  libncurses-dev libncurses6 libtmglib-dev libtmglib3 m4
Suggested packages:
  autoconf-archive gnu-standards autoconf-doc gettext
  liblapack-doc libtool-doc ncurses-doc readline-doc
  libssl-doc gfortran | fortran95-compiler gcj-jdk m4-doc
The following NEW packages will be installed:
  autoconf automake autotools-dev libblas-dev libblas3
  libconfig-dev libconfig-doc libconfig9 libgfortran5
  liblapack-dev liblapack3 liblapacke liblapacke-dev
  libltdl-dev libncurses-dev libncurses6 libreadline-dev
  libssl-dev libtmglib-dev libtmglib3 libtool m4
0 upgraded, 22 newly installed, 0 to remove and 2 not upgraded.
Need to get 15.2 MB of archives.
After this operation, 73.3 MB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu noble/main amd64 libncurses6 amd64 6.4+20240113-1ubuntu2 [112 kB]
Get:2 http://archive.ubuntu.com/ubuntu noble/main amd64 m4 amd64 1.4.19-4build1 [244 kB]
Get:3 http://archive.ubuntu.com/ubuntu noble/main amd64 autoconf all 2.71-3 [339 kB]
Get:4 http://archive.ubuntu.com/ubuntu noble/main amd64 autotools-dev all 20220109.1 [44.9 kB]
Get:5 http://archive.ubuntu.com/ubuntu noble/main amd64 automake all 1:1.16.5-1.3ubuntu1 [558 kB]
Get:6 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libblas3 amd64 3.12.0-3build1.1 [238 kB]
Get:7 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libblas-dev amd64 3.12.0-3build1.1 [170 kB]
Get:8 http://archive.ubuntu.com/ubuntu noble/universe amd64 libconfig9 amd64 1.5-0.4build2 [23.2 kB]
Get:9 http://archive.ubuntu.com/ubuntu noble/universe amd64 libconfig-dev amd64 1.5-0.4build2 [54.1 kB]
Get:10 http://archive.ubuntu.com/ubuntu noble/universe amd64 libconfig-doc all 1.5-0.4build2 [303 kB]
Get:11 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libgfortran5 amd64 14.2.0-4ubuntu2~24.04 [916 kB]
Get:12 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 liblapack3 amd64 3.12.0-3build1.1 [2646 kB]
Get:13 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 liblapack-dev amd64 3.12.0-3build1.1 [5196 kB]
Get:14 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libtmglib3 amd64 3.12.0-3build1.1 [141 kB]
Get:15 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 liblapacke amd64 3.12.0-3build1.1 [412 kB]
Get:16 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libtmglib-dev amd64 3.12.0-3build1.1 [140 kB]
Get:17 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 liblapacke-dev amd64 3.12.0-3build1.1 [344 kB]
Get:18 http://archive.ubuntu.com/ubuntu noble/main amd64 libltdl-dev amd64 2.4.7-7build1 [168 kB]
Get:19 http://archive.ubuntu.com/ubuntu noble/main amd64 libncurses-dev amd64 6.4+20240113-1ubuntu2 [384 kB]
Get:20 http://archive.ubuntu.com/ubuntu noble/main amd64 libreadline-dev amd64 8.2-4build1 [167 kB]
Get:21 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libssl-dev amd64 3.0.13-0ubuntu3.5 [2408 kB]
Get:22 http://archive.ubuntu.com/ubuntu noble/main amd64 libtool all 2.4.7-7build1 [166 kB]
Fetched 15.2 MB in 11s (1321 kB/s)
Selecting previously unselected package libncurses6:amd64.
(Reading database ... 61248 files and directories currently installed.)
Preparing to unpack .../00-libncurses6_6.4+20240113-1ubuntu2_amd64.deb ...
Unpacking libncurses6:amd64 (6.4+20240113-1ubuntu2) ...
Selecting previously unselected package m4.
Preparing to unpack .../01-m4_1.4.19-4build1_amd64.deb ...
Unpacking m4 (1.4.19-4build1) ...
Selecting previously unselected package autoconf.
Preparing to unpack .../02-autoconf_2.71-3_all.deb ...
Unpacking autoconf (2.71-3) ...
Selecting previously unselected package autotools-dev.
Preparing to unpack .../03-autotools-dev_20220109.1_all.deb ...
Unpacking autotools-dev (20220109.1) ...
Selecting previously unselected package automake.
Preparing to unpack .../04-automake_1%3a1.16.5-1.3ubuntu1_all.deb ...
Unpacking automake (1:1.16.5-1.3ubuntu1) ...
Selecting previously unselected package libblas3:amd64.
Preparing to unpack .../05-libblas3_3.12.0-3build1.1_amd64.deb ...
Unpacking libblas3:amd64 (3.12.0-3build1.1) ...
Selecting previously unselected package libblas-dev:amd64.
Preparing to unpack .../06-libblas-dev_3.12.0-3build1.1_amd64.deb ...
Unpacking libblas-dev:amd64 (3.12.0-3build1.1) ...
Selecting previously unselected package libconfig9:amd64.
Preparing to unpack .../07-libconfig9_1.5-0.4build2_amd64.deb ...
Unpacking libconfig9:amd64 (1.5-0.4build2) ...
Selecting previously unselected package libconfig-dev:amd64.
Preparing to unpack .../08-libconfig-dev_1.5-0.4build2_amd64.deb ...
Unpacking libconfig-dev:amd64 (1.5-0.4build2) ...
Selecting previously unselected package libconfig-doc.
Preparing to unpack .../09-libconfig-doc_1.5-0.4build2_all.deb ...
Unpacking libconfig-doc (1.5-0.4build2) ...
Selecting previously unselected package libgfortran5:amd64.
Preparing to unpack .../10-libgfortran5_14.2.0-4ubuntu2~24.04_amd64.deb ...
Unpacking libgfortran5:amd64 (14.2.0-4ubuntu2~24.04) ...
Selecting previously unselected package liblapack3:amd64.
Preparing to unpack .../11-liblapack3_3.12.0-3build1.1_amd64.deb ...
Unpacking liblapack3:amd64 (3.12.0-3build1.1) ...
Selecting previously unselected package liblapack-dev:amd64.
Preparing to unpack .../12-liblapack-dev_3.12.0-3build1.1_amd64.deb ...
Unpacking liblapack-dev:amd64 (3.12.0-3build1.1) ...
Selecting previously unselected package libtmglib3:amd64.
Preparing to unpack .../13-libtmglib3_3.12.0-3build1.1_amd64.deb ...
Unpacking libtmglib3:amd64 (3.12.0-3build1.1) ...
Selecting previously unselected package liblapacke:amd64.
Preparing to unpack .../14-liblapacke_3.12.0-3build1.1_amd64.deb ...
Unpacking liblapacke:amd64 (3.12.0-3build1.1) ...
Selecting previously unselected package libtmglib-dev:amd64.
Preparing to unpack .../15-libtmglib-dev_3.12.0-3build1.1_amd64.deb ...
Unpacking libtmglib-dev:amd64 (3.12.0-3build1.1) ...
Selecting previously unselected package liblapacke-dev:amd64.
Preparing to unpack .../16-liblapacke-dev_3.12.0-3build1.1_amd64.deb ...
Unpacking liblapacke-dev:amd64 (3.12.0-3build1.1) ...
Selecting previously unselected package libltdl-dev:amd64.
Preparing to unpack .../17-libltdl-dev_2.4.7-7build1_amd64.deb ...
Unpacking libltdl-dev:amd64 (2.4.7-7build1) ...
Selecting previously unselected package libncurses-dev:amd64.
Preparing to unpack .../18-libncurses-dev_6.4+20240113-1ubuntu2_amd64.deb ...
Unpacking libncurses-dev:amd64 (6.4+20240113-1ubuntu2) ...
Selecting previously unselected package libreadline-dev:amd64.
Preparing to unpack .../19-libreadline-dev_8.2-4build1_amd64.deb ...
Unpacking libreadline-dev:amd64 (8.2-4build1) ...
Selecting previously unselected package libssl-dev:amd64.
Preparing to unpack .../20-libssl-dev_3.0.13-0ubuntu3.5_amd64.deb ...
Unpacking libssl-dev:amd64 (3.0.13-0ubuntu3.5) ...
Selecting previously unselected package libtool.
Preparing to unpack .../21-libtool_2.4.7-7build1_all.deb ...
Unpacking libtool (2.4.7-7build1) ...
Setting up libconfig9:amd64 (1.5-0.4build2) ...
Setting up libconfig-doc (1.5-0.4build2) ...
Setting up m4 (1.4.19-4build1) ...
Setting up autotools-dev (20220109.1) ...
Setting up libblas3:amd64 (3.12.0-3build1.1) ...
update-alternatives: using /usr/lib/x86_64-linux-gnu/blas/libblas.so.3 to provide /usr/lib/x86_64-linux-gnu/libblas.so.3 (libblas.so.3-x86_64-linux-gnu) in auto mode
Setting up libncurses6:amd64 (6.4+20240113-1ubuntu2) ...
Setting up libssl-dev:amd64 (3.0.13-0ubuntu3.5) ...
Setting up libgfortran5:amd64 (14.2.0-4ubuntu2~24.04) ...
Setting up autoconf (2.71-3) ...
Setting up libconfig-dev:amd64 (1.5-0.4build2) ...
Setting up libblas-dev:amd64 (3.12.0-3build1.1) ...
update-alternatives: using /usr/lib/x86_64-linux-gnu/blas/libblas.so to provide /usr/lib/x86_64-linux-gnu/libblas.so (libblas.so-x86_64-linux-gnu) in auto mode
Setting up automake (1:1.16.5-1.3ubuntu1) ...
update-alternatives: using /usr/bin/automake-1.16 to provide /usr/bin/automake (automake) in auto mode
Setting up liblapack3:amd64 (3.12.0-3build1.1) ...
update-alternatives: using /usr/lib/x86_64-linux-gnu/lapack/liblapack.so.3 to provide /usr/lib/x86_64-linux-gnu/liblapack.so.3 (liblapack.so.3-x86_64-linux-gnu) in auto mode
Setting up libncurses-dev:amd64 (6.4+20240113-1ubuntu2) ...
Setting up libtool (2.4.7-7build1) ...
Setting up libreadline-dev:amd64 (8.2-4build1) ...
Setting up libltdl-dev:amd64 (2.4.7-7build1) ...
Setting up libtmglib3:amd64 (3.12.0-3build1.1) ...
Setting up liblapack-dev:amd64 (3.12.0-3build1.1) ...
update-alternatives: using /usr/lib/x86_64-linux-gnu/lapack/liblapack.so to provide /usr/lib/x86_64-linux-gnu/liblapack.so (liblapack.so-x86_64-linux-gnu) in auto mode
Setting up liblapacke:amd64 (3.12.0-3build1.1) ...
Setting up libtmglib-dev:amd64 (3.12.0-3build1.1) ...
Setting up liblapacke-dev:amd64 (3.12.0-3build1.1) ...
Processing triggers for install-info (7.1-3build2) ...
Processing triggers for libc-bin (2.39-0ubuntu8.4) ...
Processing triggers for man-db (2.12.0-4build2) ...

Installing ASN1. The log file for ASN1 installation is here: /home/vishalkumar_shaw/oai/cmake_targets/log/asn1c_install_log.txt

Installing SIMDE from source without test cases (header files only)
Cloning into '/tmp/simde'...
remote: Enumerating objects: 12105, done.
remote: Counting objects: 100% (174/174), done.
remote: Compressing objects: 100% (98/98), done.
remote: Total 12105 (delta 118), reused 117 (delta 76), pack-reused 11931 (from 2)
Receiving objects: 100% (12105/12105), 5.24 MiB | 2.82 MiB/s, done.
Resolving deltas: 100% (9195/9195), done.
Note: switching to 'c7f26b73ba8e874b95c2cec2b497826ad2188f68'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at c7f26b7 x86 avx for loongarch: use vfcmp_clt to save one instruction in `_mm_cmp_{sd,ss}` and `_mm256_cmp_pd`
commit c7f26b73ba8e874b95c2cec2b497826ad2188f68 (HEAD)
Author: jinbo <jinbo@loongson.cn>
Date:   Fri Feb 14 12:32:16 2025 +0800

    x86 avx for loongarch: use vfcmp_clt to save one instruction in `_mm_cmp_{sd,ss}` and `_mm256_cmp_pd`
'../simde' -> '/usr/include/simde'
'../simde/.git' -> '/usr/include/simde/.git'
'../simde/.git/branches' -> '/usr/include/simde/.git/branches'
'../simde/.git/description' -> '/usr/include/simde/.git/description'
'../simde/.git/hooks' -> '/usr/include/simde/.git/hooks'
'../simde/.git/hooks/fsmonitor-watchman.sample' -> '/usr/include/simde/.git/hooks/fsmonitor-watchman.sample'
'../simde/.git/hooks/push-to-checkout.sample' -> '/usr/include/simde/.git/hooks/push-to-checkout.sample'
'../simde/.git/hooks/update.sample' -> '/usr/include/simde/.git/hooks/update.sample'
'../simde/.git/hooks/pre-applypatch.sample' -> '/usr/include/simde/.git/hooks/pre-applypatch.sample'
'../simde/.git/hooks/pre-push.sample' -> '/usr/include/simde/.git/hooks/pre-push.sample'
'../simde/.git/hooks/pre-receive.sample' -> '/usr/include/simde/.git/hooks/pre-receive.sample'
'../simde/.git/hooks/sendemail-validate.sample' -> '/usr/include/simde/.git/hooks/sendemail-validate.sample'
'../simde/.git/hooks/pre-merge-commit.sample' -> '/usr/include/simde/.git/hooks/pre-merge-commit.sample'
'../simde/.git/hooks/applypatch-msg.sample' -> '/usr/include/simde/.git/hooks/applypatch-msg.sample'
'../simde/.git/hooks/pre-commit.sample' -> '/usr/include/simde/.git/hooks/pre-commit.sample'
'../simde/.git/hooks/prepare-commit-msg.sample' -> '/usr/include/simde/.git/hooks/prepare-commit-msg.sample'
'../simde/.git/hooks/commit-msg.sample' -> '/usr/include/simde/.git/hooks/commit-msg.sample'
'../simde/.git/hooks/post-update.sample' -> '/usr/include/simde/.git/hooks/post-update.sample'
'../simde/.git/hooks/pre-rebase.sample' -> '/usr/include/simde/.git/hooks/pre-rebase.sample'
'../simde/.git/info' -> '/usr/include/simde/.git/info'
'../simde/.git/info/exclude' -> '/usr/include/simde/.git/info/exclude'
'../simde/.git/refs' -> '/usr/include/simde/.git/refs'
'../simde/.git/refs/heads' -> '/usr/include/simde/.git/refs/heads'
'../simde/.git/refs/heads/master' -> '/usr/include/simde/.git/refs/heads/master'
'../simde/.git/refs/tags' -> '/usr/include/simde/.git/refs/tags'
'../simde/.git/refs/remotes' -> '/usr/include/simde/.git/refs/remotes'
'../simde/.git/refs/remotes/origin' -> '/usr/include/simde/.git/refs/remotes/origin'
'../simde/.git/refs/remotes/origin/HEAD' -> '/usr/include/simde/.git/refs/remotes/origin/HEAD'
'../simde/.git/HEAD' -> '/usr/include/simde/.git/HEAD'
'../simde/.git/objects' -> '/usr/include/simde/.git/objects'
'../simde/.git/objects/pack' -> '/usr/include/simde/.git/objects/pack'
'../simde/.git/objects/pack/pack-a9cdbe93b03d6d3d304046427dd57d28eef15d47.idx' -> '/usr/include/simde/.git/objects/pack/pack-a9cdbe93b03d6d3d304046427dd57d28eef15d47.idx'
'../simde/.git/objects/pack/pack-a9cdbe93b03d6d3d304046427dd57d28eef15d47.rev' -> '/usr/include/simde/.git/objects/pack/pack-a9cdbe93b03d6d3d304046427dd57d28eef15d47.rev'
'../simde/.git/objects/pack/pack-a9cdbe93b03d6d3d304046427dd57d28eef15d47.pack' -> '/usr/include/simde/.git/objects/pack/pack-a9cdbe93b03d6d3d304046427dd57d28eef15d47.pack'
'../simde/.git/objects/info' -> '/usr/include/simde/.git/objects/info'
'../simde/.git/config' -> '/usr/include/simde/.git/config'
'../simde/.git/packed-refs' -> '/usr/include/simde/.git/packed-refs'
'../simde/.git/logs' -> '/usr/include/simde/.git/logs'
'../simde/.git/logs/refs' -> '/usr/include/simde/.git/logs/refs'
'../simde/.git/logs/refs/remotes' -> '/usr/include/simde/.git/logs/refs/remotes'
'../simde/.git/logs/refs/remotes/origin' -> '/usr/include/simde/.git/logs/refs/remotes/origin'
'../simde/.git/logs/refs/remotes/origin/HEAD' -> '/usr/include/simde/.git/logs/refs/remotes/origin/HEAD'
'../simde/.git/logs/refs/heads' -> '/usr/include/simde/.git/logs/refs/heads'
'../simde/.git/logs/refs/heads/master' -> '/usr/include/simde/.git/logs/refs/heads/master'
'../simde/.git/logs/HEAD' -> '/usr/include/simde/.git/logs/HEAD'
'../simde/.git/index' -> '/usr/include/simde/.git/index'
'../simde/COPYING' -> '/usr/include/simde/COPYING'
'../simde/README.md' -> '/usr/include/simde/README.md'
'../simde/arm' -> '/usr/include/simde/arm'
'../simde/arm/neon.h' -> '/usr/include/simde/arm/neon.h'
'../simde/arm/neon' -> '/usr/include/simde/arm/neon'
'../simde/arm/neon/aba.h' -> '/usr/include/simde/arm/neon/aba.h'
'../simde/arm/neon/abal.h' -> '/usr/include/simde/arm/neon/abal.h'
'../simde/arm/neon/abal_high.h' -> '/usr/include/simde/arm/neon/abal_high.h'
'../simde/arm/neon/abd.h' -> '/usr/include/simde/arm/neon/abd.h'
'../simde/arm/neon/abdl.h' -> '/usr/include/simde/arm/neon/abdl.h'
'../simde/arm/neon/abdl_high.h' -> '/usr/include/simde/arm/neon/abdl_high.h'
'../simde/arm/neon/abs.h' -> '/usr/include/simde/arm/neon/abs.h'
'../simde/arm/neon/add.h' -> '/usr/include/simde/arm/neon/add.h'
'../simde/arm/neon/addhn.h' -> '/usr/include/simde/arm/neon/addhn.h'
'../simde/arm/neon/addhn_high.h' -> '/usr/include/simde/arm/neon/addhn_high.h'
'../simde/arm/neon/addl.h' -> '/usr/include/simde/arm/neon/addl.h'
'../simde/arm/neon/addl_high.h' -> '/usr/include/simde/arm/neon/addl_high.h'
'../simde/arm/neon/addlv.h' -> '/usr/include/simde/arm/neon/addlv.h'
'../simde/arm/neon/addv.h' -> '/usr/include/simde/arm/neon/addv.h'
'../simde/arm/neon/addw.h' -> '/usr/include/simde/arm/neon/addw.h'
'../simde/arm/neon/addw_high.h' -> '/usr/include/simde/arm/neon/addw_high.h'
'../simde/arm/neon/aes.h' -> '/usr/include/simde/arm/neon/aes.h'
'../simde/arm/neon/and.h' -> '/usr/include/simde/arm/neon/and.h'
'../simde/arm/neon/bcax.h' -> '/usr/include/simde/arm/neon/bcax.h'
'../simde/arm/neon/bic.h' -> '/usr/include/simde/arm/neon/bic.h'
'../simde/arm/neon/bsl.h' -> '/usr/include/simde/arm/neon/bsl.h'
'../simde/arm/neon/cadd_rot270.h' -> '/usr/include/simde/arm/neon/cadd_rot270.h'
'../simde/arm/neon/cadd_rot90.h' -> '/usr/include/simde/arm/neon/cadd_rot90.h'
'../simde/arm/neon/cage.h' -> '/usr/include/simde/arm/neon/cage.h'
'../simde/arm/neon/cagt.h' -> '/usr/include/simde/arm/neon/cagt.h'
'../simde/arm/neon/cale.h' -> '/usr/include/simde/arm/neon/cale.h'
'../simde/arm/neon/calt.h' -> '/usr/include/simde/arm/neon/calt.h'
'../simde/arm/neon/ceq.h' -> '/usr/include/simde/arm/neon/ceq.h'
'../simde/arm/neon/ceqz.h' -> '/usr/include/simde/arm/neon/ceqz.h'
'../simde/arm/neon/cge.h' -> '/usr/include/simde/arm/neon/cge.h'
'../simde/arm/neon/cgez.h' -> '/usr/include/simde/arm/neon/cgez.h'
'../simde/arm/neon/cgt.h' -> '/usr/include/simde/arm/neon/cgt.h'
'../simde/arm/neon/cgtz.h' -> '/usr/include/simde/arm/neon/cgtz.h'
'../simde/arm/neon/cle.h' -> '/usr/include/simde/arm/neon/cle.h'
'../simde/arm/neon/clez.h' -> '/usr/include/simde/arm/neon/clez.h'
'../simde/arm/neon/cls.h' -> '/usr/include/simde/arm/neon/cls.h'
'../simde/arm/neon/clt.h' -> '/usr/include/simde/arm/neon/clt.h'
'../simde/arm/neon/cltz.h' -> '/usr/include/simde/arm/neon/cltz.h'
'../simde/arm/neon/clz.h' -> '/usr/include/simde/arm/neon/clz.h'
'../simde/arm/neon/cmla.h' -> '/usr/include/simde/arm/neon/cmla.h'
'../simde/arm/neon/cmla_lane.h' -> '/usr/include/simde/arm/neon/cmla_lane.h'
'../simde/arm/neon/cmla_rot180.h' -> '/usr/include/simde/arm/neon/cmla_rot180.h'
'../simde/arm/neon/cmla_rot180_lane.h' -> '/usr/include/simde/arm/neon/cmla_rot180_lane.h'
'../simde/arm/neon/cmla_rot270.h' -> '/usr/include/simde/arm/neon/cmla_rot270.h'
'../simde/arm/neon/cmla_rot270_lane.h' -> '/usr/include/simde/arm/neon/cmla_rot270_lane.h'
'../simde/arm/neon/cmla_rot90.h' -> '/usr/include/simde/arm/neon/cmla_rot90.h'
'../simde/arm/neon/cmla_rot90_lane.h' -> '/usr/include/simde/arm/neon/cmla_rot90_lane.h'
'../simde/arm/neon/cnt.h' -> '/usr/include/simde/arm/neon/cnt.h'
'../simde/arm/neon/combine.h' -> '/usr/include/simde/arm/neon/combine.h'
'../simde/arm/neon/copy_lane.h' -> '/usr/include/simde/arm/neon/copy_lane.h'
'../simde/arm/neon/crc32.h' -> '/usr/include/simde/arm/neon/crc32.h'
'../simde/arm/neon/create.h' -> '/usr/include/simde/arm/neon/create.h'
'../simde/arm/neon/cvt.h' -> '/usr/include/simde/arm/neon/cvt.h'
'../simde/arm/neon/cvt_n.h' -> '/usr/include/simde/arm/neon/cvt_n.h'
'../simde/arm/neon/cvtm.h' -> '/usr/include/simde/arm/neon/cvtm.h'
'../simde/arm/neon/cvtn.h' -> '/usr/include/simde/arm/neon/cvtn.h'
'../simde/arm/neon/cvtp.h' -> '/usr/include/simde/arm/neon/cvtp.h'
'../simde/arm/neon/div.h' -> '/usr/include/simde/arm/neon/div.h'
'../simde/arm/neon/dot.h' -> '/usr/include/simde/arm/neon/dot.h'
'../simde/arm/neon/dot_lane.h' -> '/usr/include/simde/arm/neon/dot_lane.h'
'../simde/arm/neon/dup_lane.h' -> '/usr/include/simde/arm/neon/dup_lane.h'
'../simde/arm/neon/dup_n.h' -> '/usr/include/simde/arm/neon/dup_n.h'
'../simde/arm/neon/eor.h' -> '/usr/include/simde/arm/neon/eor.h'
'../simde/arm/neon/ext.h' -> '/usr/include/simde/arm/neon/ext.h'
'../simde/arm/neon/fma.h' -> '/usr/include/simde/arm/neon/fma.h'
'../simde/arm/neon/fma_lane.h' -> '/usr/include/simde/arm/neon/fma_lane.h'
'../simde/arm/neon/fma_n.h' -> '/usr/include/simde/arm/neon/fma_n.h'
'../simde/arm/neon/fmlal.h' -> '/usr/include/simde/arm/neon/fmlal.h'
'../simde/arm/neon/fmlsl.h' -> '/usr/include/simde/arm/neon/fmlsl.h'
'../simde/arm/neon/fms.h' -> '/usr/include/simde/arm/neon/fms.h'
'../simde/arm/neon/fms_lane.h' -> '/usr/include/simde/arm/neon/fms_lane.h'
'../simde/arm/neon/fms_n.h' -> '/usr/include/simde/arm/neon/fms_n.h'
'../simde/arm/neon/get_high.h' -> '/usr/include/simde/arm/neon/get_high.h'
'../simde/arm/neon/get_lane.h' -> '/usr/include/simde/arm/neon/get_lane.h'
'../simde/arm/neon/get_low.h' -> '/usr/include/simde/arm/neon/get_low.h'
'../simde/arm/neon/hadd.h' -> '/usr/include/simde/arm/neon/hadd.h'
'../simde/arm/neon/hsub.h' -> '/usr/include/simde/arm/neon/hsub.h'
'../simde/arm/neon/ld1.h' -> '/usr/include/simde/arm/neon/ld1.h'
'../simde/arm/neon/ld1_dup.h' -> '/usr/include/simde/arm/neon/ld1_dup.h'
'../simde/arm/neon/ld1_lane.h' -> '/usr/include/simde/arm/neon/ld1_lane.h'
'../simde/arm/neon/ld1_x2.h' -> '/usr/include/simde/arm/neon/ld1_x2.h'
'../simde/arm/neon/ld1_x3.h' -> '/usr/include/simde/arm/neon/ld1_x3.h'
'../simde/arm/neon/ld1_x4.h' -> '/usr/include/simde/arm/neon/ld1_x4.h'
'../simde/arm/neon/ld1q_x2.h' -> '/usr/include/simde/arm/neon/ld1q_x2.h'
'../simde/arm/neon/ld1q_x3.h' -> '/usr/include/simde/arm/neon/ld1q_x3.h'
'../simde/arm/neon/ld1q_x4.h' -> '/usr/include/simde/arm/neon/ld1q_x4.h'
'../simde/arm/neon/ld2.h' -> '/usr/include/simde/arm/neon/ld2.h'
'../simde/arm/neon/ld2_dup.h' -> '/usr/include/simde/arm/neon/ld2_dup.h'
'../simde/arm/neon/ld2_lane.h' -> '/usr/include/simde/arm/neon/ld2_lane.h'
'../simde/arm/neon/ld3.h' -> '/usr/include/simde/arm/neon/ld3.h'
'../simde/arm/neon/ld3_dup.h' -> '/usr/include/simde/arm/neon/ld3_dup.h'
'../simde/arm/neon/ld3_lane.h' -> '/usr/include/simde/arm/neon/ld3_lane.h'
'../simde/arm/neon/ld4.h' -> '/usr/include/simde/arm/neon/ld4.h'
'../simde/arm/neon/ld4_dup.h' -> '/usr/include/simde/arm/neon/ld4_dup.h'
'../simde/arm/neon/ld4_lane.h' -> '/usr/include/simde/arm/neon/ld4_lane.h'
'../simde/arm/neon/max.h' -> '/usr/include/simde/arm/neon/max.h'
'../simde/arm/neon/maxnm.h' -> '/usr/include/simde/arm/neon/maxnm.h'
'../simde/arm/neon/maxnmv.h' -> '/usr/include/simde/arm/neon/maxnmv.h'
'../simde/arm/neon/maxv.h' -> '/usr/include/simde/arm/neon/maxv.h'
'../simde/arm/neon/min.h' -> '/usr/include/simde/arm/neon/min.h'
'../simde/arm/neon/minnm.h' -> '/usr/include/simde/arm/neon/minnm.h'
'../simde/arm/neon/minnmv.h' -> '/usr/include/simde/arm/neon/minnmv.h'
'../simde/arm/neon/minv.h' -> '/usr/include/simde/arm/neon/minv.h'
'../simde/arm/neon/mla.h' -> '/usr/include/simde/arm/neon/mla.h'
'../simde/arm/neon/mla_lane.h' -> '/usr/include/simde/arm/neon/mla_lane.h'
'../simde/arm/neon/mla_n.h' -> '/usr/include/simde/arm/neon/mla_n.h'
'../simde/arm/neon/mlal.h' -> '/usr/include/simde/arm/neon/mlal.h'
'../simde/arm/neon/mlal_high.h' -> '/usr/include/simde/arm/neon/mlal_high.h'
'../simde/arm/neon/mlal_high_lane.h' -> '/usr/include/simde/arm/neon/mlal_high_lane.h'
'../simde/arm/neon/mlal_high_n.h' -> '/usr/include/simde/arm/neon/mlal_high_n.h'
'../simde/arm/neon/mlal_lane.h' -> '/usr/include/simde/arm/neon/mlal_lane.h'
'../simde/arm/neon/mlal_n.h' -> '/usr/include/simde/arm/neon/mlal_n.h'
'../simde/arm/neon/mls.h' -> '/usr/include/simde/arm/neon/mls.h'
'../simde/arm/neon/mls_lane.h' -> '/usr/include/simde/arm/neon/mls_lane.h'
'../simde/arm/neon/mls_n.h' -> '/usr/include/simde/arm/neon/mls_n.h'
'../simde/arm/neon/mlsl.h' -> '/usr/include/simde/arm/neon/mlsl.h'
'../simde/arm/neon/mlsl_high.h' -> '/usr/include/simde/arm/neon/mlsl_high.h'
'../simde/arm/neon/mlsl_high_lane.h' -> '/usr/include/simde/arm/neon/mlsl_high_lane.h'
'../simde/arm/neon/mlsl_high_n.h' -> '/usr/include/simde/arm/neon/mlsl_high_n.h'
'../simde/arm/neon/mlsl_lane.h' -> '/usr/include/simde/arm/neon/mlsl_lane.h'
'../simde/arm/neon/mlsl_n.h' -> '/usr/include/simde/arm/neon/mlsl_n.h'
'../simde/arm/neon/mmlaq.h' -> '/usr/include/simde/arm/neon/mmlaq.h'
'../simde/arm/neon/movl.h' -> '/usr/include/simde/arm/neon/movl.h'
'../simde/arm/neon/movl_high.h' -> '/usr/include/simde/arm/neon/movl_high.h'
'../simde/arm/neon/movn.h' -> '/usr/include/simde/arm/neon/movn.h'
'../simde/arm/neon/movn_high.h' -> '/usr/include/simde/arm/neon/movn_high.h'
'../simde/arm/neon/mul.h' -> '/usr/include/simde/arm/neon/mul.h'
'../simde/arm/neon/mul_lane.h' -> '/usr/include/simde/arm/neon/mul_lane.h'
'../simde/arm/neon/mul_n.h' -> '/usr/include/simde/arm/neon/mul_n.h'
'../simde/arm/neon/mull.h' -> '/usr/include/simde/arm/neon/mull.h'
'../simde/arm/neon/mull_high.h' -> '/usr/include/simde/arm/neon/mull_high.h'
'../simde/arm/neon/mull_high_lane.h' -> '/usr/include/simde/arm/neon/mull_high_lane.h'
'../simde/arm/neon/mull_high_n.h' -> '/usr/include/simde/arm/neon/mull_high_n.h'
'../simde/arm/neon/mull_lane.h' -> '/usr/include/simde/arm/neon/mull_lane.h'
'../simde/arm/neon/mull_n.h' -> '/usr/include/simde/arm/neon/mull_n.h'
'../simde/arm/neon/mulx.h' -> '/usr/include/simde/arm/neon/mulx.h'
'../simde/arm/neon/mulx_lane.h' -> '/usr/include/simde/arm/neon/mulx_lane.h'
'../simde/arm/neon/mulx_n.h' -> '/usr/include/simde/arm/neon/mulx_n.h'
'../simde/arm/neon/mvn.h' -> '/usr/include/simde/arm/neon/mvn.h'
'../simde/arm/neon/neg.h' -> '/usr/include/simde/arm/neon/neg.h'
'../simde/arm/neon/orn.h' -> '/usr/include/simde/arm/neon/orn.h'
'../simde/arm/neon/orr.h' -> '/usr/include/simde/arm/neon/orr.h'
'../simde/arm/neon/padal.h' -> '/usr/include/simde/arm/neon/padal.h'
'../simde/arm/neon/padd.h' -> '/usr/include/simde/arm/neon/padd.h'
'../simde/arm/neon/paddl.h' -> '/usr/include/simde/arm/neon/paddl.h'
'../simde/arm/neon/pmax.h' -> '/usr/include/simde/arm/neon/pmax.h'
'../simde/arm/neon/pmaxnm.h' -> '/usr/include/simde/arm/neon/pmaxnm.h'
'../simde/arm/neon/pmin.h' -> '/usr/include/simde/arm/neon/pmin.h'
'../simde/arm/neon/pminnm.h' -> '/usr/include/simde/arm/neon/pminnm.h'
'../simde/arm/neon/qabs.h' -> '/usr/include/simde/arm/neon/qabs.h'
'../simde/arm/neon/qadd.h' -> '/usr/include/simde/arm/neon/qadd.h'
'../simde/arm/neon/qdmlal.h' -> '/usr/include/simde/arm/neon/qdmlal.h'
'../simde/arm/neon/qdmlal_high.h' -> '/usr/include/simde/arm/neon/qdmlal_high.h'
'../simde/arm/neon/qdmlal_high_lane.h' -> '/usr/include/simde/arm/neon/qdmlal_high_lane.h'
'../simde/arm/neon/qdmlal_high_n.h' -> '/usr/include/simde/arm/neon/qdmlal_high_n.h'
'../simde/arm/neon/qdmlal_lane.h' -> '/usr/include/simde/arm/neon/qdmlal_lane.h'
'../simde/arm/neon/qdmlal_n.h' -> '/usr/include/simde/arm/neon/qdmlal_n.h'
'../simde/arm/neon/qdmlsl.h' -> '/usr/include/simde/arm/neon/qdmlsl.h'
'../simde/arm/neon/qdmlsl_high.h' -> '/usr/include/simde/arm/neon/qdmlsl_high.h'
'../simde/arm/neon/qdmlsl_high_lane.h' -> '/usr/include/simde/arm/neon/qdmlsl_high_lane.h'
'../simde/arm/neon/qdmlsl_high_n.h' -> '/usr/include/simde/arm/neon/qdmlsl_high_n.h'
'../simde/arm/neon/qdmlsl_lane.h' -> '/usr/include/simde/arm/neon/qdmlsl_lane.h'
'../simde/arm/neon/qdmlsl_n.h' -> '/usr/include/simde/arm/neon/qdmlsl_n.h'
'../simde/arm/neon/qdmulh.h' -> '/usr/include/simde/arm/neon/qdmulh.h'
'../simde/arm/neon/qdmulh_lane.h' -> '/usr/include/simde/arm/neon/qdmulh_lane.h'
'../simde/arm/neon/qdmulh_n.h' -> '/usr/include/simde/arm/neon/qdmulh_n.h'
'../simde/arm/neon/qdmull.h' -> '/usr/include/simde/arm/neon/qdmull.h'
'../simde/arm/neon/qdmull_high.h' -> '/usr/include/simde/arm/neon/qdmull_high.h'
'../simde/arm/neon/qdmull_high_lane.h' -> '/usr/include/simde/arm/neon/qdmull_high_lane.h'
'../simde/arm/neon/qdmull_high_n.h' -> '/usr/include/simde/arm/neon/qdmull_high_n.h'
'../simde/arm/neon/qdmull_lane.h' -> '/usr/include/simde/arm/neon/qdmull_lane.h'
'../simde/arm/neon/qdmull_n.h' -> '/usr/include/simde/arm/neon/qdmull_n.h'
'../simde/arm/neon/qmovn.h' -> '/usr/include/simde/arm/neon/qmovn.h'
'../simde/arm/neon/qmovn_high.h' -> '/usr/include/simde/arm/neon/qmovn_high.h'
'../simde/arm/neon/qmovun.h' -> '/usr/include/simde/arm/neon/qmovun.h'
'../simde/arm/neon/qmovun_high.h' -> '/usr/include/simde/arm/neon/qmovun_high.h'
'../simde/arm/neon/qneg.h' -> '/usr/include/simde/arm/neon/qneg.h'
'../simde/arm/neon/qrdmlah.h' -> '/usr/include/simde/arm/neon/qrdmlah.h'
'../simde/arm/neon/qrdmlah_lane.h' -> '/usr/include/simde/arm/neon/qrdmlah_lane.h'
'../simde/arm/neon/qrdmlsh.h' -> '/usr/include/simde/arm/neon/qrdmlsh.h'
'../simde/arm/neon/qrdmlsh_lane.h' -> '/usr/include/simde/arm/neon/qrdmlsh_lane.h'
'../simde/arm/neon/qrdmulh.h' -> '/usr/include/simde/arm/neon/qrdmulh.h'
'../simde/arm/neon/qrdmulh_lane.h' -> '/usr/include/simde/arm/neon/qrdmulh_lane.h'
'../simde/arm/neon/qrdmulh_n.h' -> '/usr/include/simde/arm/neon/qrdmulh_n.h'
'../simde/arm/neon/qrshl.h' -> '/usr/include/simde/arm/neon/qrshl.h'
'../simde/arm/neon/qrshrn_high_n.h' -> '/usr/include/simde/arm/neon/qrshrn_high_n.h'
'../simde/arm/neon/qrshrn_n.h' -> '/usr/include/simde/arm/neon/qrshrn_n.h'
'../simde/arm/neon/qrshrun_high_n.h' -> '/usr/include/simde/arm/neon/qrshrun_high_n.h'
'../simde/arm/neon/qrshrun_n.h' -> '/usr/include/simde/arm/neon/qrshrun_n.h'
'../simde/arm/neon/qshl.h' -> '/usr/include/simde/arm/neon/qshl.h'
'../simde/arm/neon/qshl_n.h' -> '/usr/include/simde/arm/neon/qshl_n.h'
'../simde/arm/neon/qshlu_n.h' -> '/usr/include/simde/arm/neon/qshlu_n.h'
'../simde/arm/neon/qshrn_high_n.h' -> '/usr/include/simde/arm/neon/qshrn_high_n.h'
'../simde/arm/neon/qshrn_n.h' -> '/usr/include/simde/arm/neon/qshrn_n.h'
'../simde/arm/neon/qshrun_high_n.h' -> '/usr/include/simde/arm/neon/qshrun_high_n.h'
'../simde/arm/neon/qshrun_n.h' -> '/usr/include/simde/arm/neon/qshrun_n.h'
'../simde/arm/neon/qsub.h' -> '/usr/include/simde/arm/neon/qsub.h'
'../simde/arm/neon/qtbl.h' -> '/usr/include/simde/arm/neon/qtbl.h'
'../simde/arm/neon/qtbx.h' -> '/usr/include/simde/arm/neon/qtbx.h'
'../simde/arm/neon/raddhn.h' -> '/usr/include/simde/arm/neon/raddhn.h'
'../simde/arm/neon/raddhn_high.h' -> '/usr/include/simde/arm/neon/raddhn_high.h'
'../simde/arm/neon/rax.h' -> '/usr/include/simde/arm/neon/rax.h'
'../simde/arm/neon/rbit.h' -> '/usr/include/simde/arm/neon/rbit.h'
'../simde/arm/neon/recpe.h' -> '/usr/include/simde/arm/neon/recpe.h'
'../simde/arm/neon/recps.h' -> '/usr/include/simde/arm/neon/recps.h'
'../simde/arm/neon/recpx.h' -> '/usr/include/simde/arm/neon/recpx.h'
'../simde/arm/neon/reinterpret.h' -> '/usr/include/simde/arm/neon/reinterpret.h'
'../simde/arm/neon/rev16.h' -> '/usr/include/simde/arm/neon/rev16.h'
'../simde/arm/neon/rev32.h' -> '/usr/include/simde/arm/neon/rev32.h'
'../simde/arm/neon/rev64.h' -> '/usr/include/simde/arm/neon/rev64.h'
'../simde/arm/neon/rhadd.h' -> '/usr/include/simde/arm/neon/rhadd.h'
'../simde/arm/neon/rnd.h' -> '/usr/include/simde/arm/neon/rnd.h'
'../simde/arm/neon/rnd32x.h' -> '/usr/include/simde/arm/neon/rnd32x.h'
'../simde/arm/neon/rnd32z.h' -> '/usr/include/simde/arm/neon/rnd32z.h'
'../simde/arm/neon/rnd64x.h' -> '/usr/include/simde/arm/neon/rnd64x.h'
'../simde/arm/neon/rnd64z.h' -> '/usr/include/simde/arm/neon/rnd64z.h'
'../simde/arm/neon/rnda.h' -> '/usr/include/simde/arm/neon/rnda.h'
'../simde/arm/neon/rndi.h' -> '/usr/include/simde/arm/neon/rndi.h'
'../simde/arm/neon/rndm.h' -> '/usr/include/simde/arm/neon/rndm.h'
'../simde/arm/neon/rndn.h' -> '/usr/include/simde/arm/neon/rndn.h'
'../simde/arm/neon/rndp.h' -> '/usr/include/simde/arm/neon/rndp.h'
'../simde/arm/neon/rndx.h' -> '/usr/include/simde/arm/neon/rndx.h'
'../simde/arm/neon/rshl.h' -> '/usr/include/simde/arm/neon/rshl.h'
'../simde/arm/neon/rshr_n.h' -> '/usr/include/simde/arm/neon/rshr_n.h'
'../simde/arm/neon/rshrn_high_n.h' -> '/usr/include/simde/arm/neon/rshrn_high_n.h'
'../simde/arm/neon/rshrn_n.h' -> '/usr/include/simde/arm/neon/rshrn_n.h'
'../simde/arm/neon/rsqrte.h' -> '/usr/include/simde/arm/neon/rsqrte.h'
'../simde/arm/neon/rsqrts.h' -> '/usr/include/simde/arm/neon/rsqrts.h'
'../simde/arm/neon/rsra_n.h' -> '/usr/include/simde/arm/neon/rsra_n.h'
'../simde/arm/neon/rsubhn.h' -> '/usr/include/simde/arm/neon/rsubhn.h'
'../simde/arm/neon/rsubhn_high.h' -> '/usr/include/simde/arm/neon/rsubhn_high.h'
'../simde/arm/neon/set_lane.h' -> '/usr/include/simde/arm/neon/set_lane.h'
'../simde/arm/neon/sha1.h' -> '/usr/include/simde/arm/neon/sha1.h'
'../simde/arm/neon/sha256.h' -> '/usr/include/simde/arm/neon/sha256.h'
'../simde/arm/neon/sha512.h' -> '/usr/include/simde/arm/neon/sha512.h'
'../simde/arm/neon/shl.h' -> '/usr/include/simde/arm/neon/shl.h'
'../simde/arm/neon/shl_n.h' -> '/usr/include/simde/arm/neon/shl_n.h'
'../simde/arm/neon/shll_high_n.h' -> '/usr/include/simde/arm/neon/shll_high_n.h'
'../simde/arm/neon/shll_n.h' -> '/usr/include/simde/arm/neon/shll_n.h'
'../simde/arm/neon/shr_n.h' -> '/usr/include/simde/arm/neon/shr_n.h'
'../simde/arm/neon/shrn_high_n.h' -> '/usr/include/simde/arm/neon/shrn_high_n.h'
'../simde/arm/neon/shrn_n.h' -> '/usr/include/simde/arm/neon/shrn_n.h'
'../simde/arm/neon/sli_n.h' -> '/usr/include/simde/arm/neon/sli_n.h'
'../simde/arm/neon/sm3.h' -> '/usr/include/simde/arm/neon/sm3.h'
'../simde/arm/neon/sm4.h' -> '/usr/include/simde/arm/neon/sm4.h'
'../simde/arm/neon/sqadd.h' -> '/usr/include/simde/arm/neon/sqadd.h'
'../simde/arm/neon/sqrt.h' -> '/usr/include/simde/arm/neon/sqrt.h'
'../simde/arm/neon/sra_n.h' -> '/usr/include/simde/arm/neon/sra_n.h'
'../simde/arm/neon/sri_n.h' -> '/usr/include/simde/arm/neon/sri_n.h'
'../simde/arm/neon/st1.h' -> '/usr/include/simde/arm/neon/st1.h'
'../simde/arm/neon/st1_lane.h' -> '/usr/include/simde/arm/neon/st1_lane.h'
'../simde/arm/neon/st1_x2.h' -> '/usr/include/simde/arm/neon/st1_x2.h'
'../simde/arm/neon/st1_x3.h' -> '/usr/include/simde/arm/neon/st1_x3.h'
'../simde/arm/neon/st1_x4.h' -> '/usr/include/simde/arm/neon/st1_x4.h'
'../simde/arm/neon/st1q_x2.h' -> '/usr/include/simde/arm/neon/st1q_x2.h'
'../simde/arm/neon/st1q_x3.h' -> '/usr/include/simde/arm/neon/st1q_x3.h'
'../simde/arm/neon/st1q_x4.h' -> '/usr/include/simde/arm/neon/st1q_x4.h'
'../simde/arm/neon/st2.h' -> '/usr/include/simde/arm/neon/st2.h'
'../simde/arm/neon/st2_lane.h' -> '/usr/include/simde/arm/neon/st2_lane.h'
'../simde/arm/neon/st3.h' -> '/usr/include/simde/arm/neon/st3.h'
'../simde/arm/neon/st3_lane.h' -> '/usr/include/simde/arm/neon/st3_lane.h'
'../simde/arm/neon/st4.h' -> '/usr/include/simde/arm/neon/st4.h'
'../simde/arm/neon/st4_lane.h' -> '/usr/include/simde/arm/neon/st4_lane.h'
'../simde/arm/neon/sub.h' -> '/usr/include/simde/arm/neon/sub.h'
'../simde/arm/neon/subhn.h' -> '/usr/include/simde/arm/neon/subhn.h'
'../simde/arm/neon/subhn_high.h' -> '/usr/include/simde/arm/neon/subhn_high.h'
'../simde/arm/neon/subl.h' -> '/usr/include/simde/arm/neon/subl.h'
'../simde/arm/neon/subl_high.h' -> '/usr/include/simde/arm/neon/subl_high.h'
'../simde/arm/neon/subw.h' -> '/usr/include/simde/arm/neon/subw.h'
'../simde/arm/neon/subw_high.h' -> '/usr/include/simde/arm/neon/subw_high.h'
'../simde/arm/neon/sudot_lane.h' -> '/usr/include/simde/arm/neon/sudot_lane.h'
'../simde/arm/neon/tbl.h' -> '/usr/include/simde/arm/neon/tbl.h'
'../simde/arm/neon/tbx.h' -> '/usr/include/simde/arm/neon/tbx.h'
'../simde/arm/neon/trn.h' -> '/usr/include/simde/arm/neon/trn.h'
'../simde/arm/neon/trn1.h' -> '/usr/include/simde/arm/neon/trn1.h'
'../simde/arm/neon/trn2.h' -> '/usr/include/simde/arm/neon/trn2.h'
'../simde/arm/neon/tst.h' -> '/usr/include/simde/arm/neon/tst.h'
'../simde/arm/neon/types.h' -> '/usr/include/simde/arm/neon/types.h'
'../simde/arm/neon/uqadd.h' -> '/usr/include/simde/arm/neon/uqadd.h'
'../simde/arm/neon/usdot.h' -> '/usr/include/simde/arm/neon/usdot.h'
'../simde/arm/neon/usdot_lane.h' -> '/usr/include/simde/arm/neon/usdot_lane.h'
'../simde/arm/neon/uzp.h' -> '/usr/include/simde/arm/neon/uzp.h'
'../simde/arm/neon/uzp1.h' -> '/usr/include/simde/arm/neon/uzp1.h'
'../simde/arm/neon/uzp2.h' -> '/usr/include/simde/arm/neon/uzp2.h'
'../simde/arm/neon/xar.h' -> '/usr/include/simde/arm/neon/xar.h'
'../simde/arm/neon/zip.h' -> '/usr/include/simde/arm/neon/zip.h'
'../simde/arm/neon/zip1.h' -> '/usr/include/simde/arm/neon/zip1.h'
'../simde/arm/neon/zip2.h' -> '/usr/include/simde/arm/neon/zip2.h'
'../simde/arm/sve.h' -> '/usr/include/simde/arm/sve.h'
'../simde/arm/sve' -> '/usr/include/simde/arm/sve'
'../simde/arm/sve/add.h' -> '/usr/include/simde/arm/sve/add.h'
'../simde/arm/sve/and.h' -> '/usr/include/simde/arm/sve/and.h'
'../simde/arm/sve/cmplt.h' -> '/usr/include/simde/arm/sve/cmplt.h'
'../simde/arm/sve/cnt.h' -> '/usr/include/simde/arm/sve/cnt.h'
'../simde/arm/sve/dup.h' -> '/usr/include/simde/arm/sve/dup.h'
'../simde/arm/sve/ld1.h' -> '/usr/include/simde/arm/sve/ld1.h'
'../simde/arm/sve/ptest.h' -> '/usr/include/simde/arm/sve/ptest.h'
'../simde/arm/sve/ptrue.h' -> '/usr/include/simde/arm/sve/ptrue.h'
'../simde/arm/sve/qadd.h' -> '/usr/include/simde/arm/sve/qadd.h'
'../simde/arm/sve/reinterpret.h' -> '/usr/include/simde/arm/sve/reinterpret.h'
'../simde/arm/sve/sel.h' -> '/usr/include/simde/arm/sve/sel.h'
'../simde/arm/sve/st1.h' -> '/usr/include/simde/arm/sve/st1.h'
'../simde/arm/sve/sub.h' -> '/usr/include/simde/arm/sve/sub.h'
'../simde/arm/sve/types.h' -> '/usr/include/simde/arm/sve/types.h'
'../simde/arm/sve/whilelt.h' -> '/usr/include/simde/arm/sve/whilelt.h'
'../simde/check.h' -> '/usr/include/simde/check.h'
'../simde/debug-trap.h' -> '/usr/include/simde/debug-trap.h'
'../simde/hedley.h' -> '/usr/include/simde/hedley.h'
'../simde/mips' -> '/usr/include/simde/mips'
'../simde/mips/msa.h' -> '/usr/include/simde/mips/msa.h'
'../simde/mips/msa' -> '/usr/include/simde/mips/msa'
'../simde/mips/msa/add_a.h' -> '/usr/include/simde/mips/msa/add_a.h'
'../simde/mips/msa/adds.h' -> '/usr/include/simde/mips/msa/adds.h'
'../simde/mips/msa/adds_a.h' -> '/usr/include/simde/mips/msa/adds_a.h'
'../simde/mips/msa/addv.h' -> '/usr/include/simde/mips/msa/addv.h'
'../simde/mips/msa/addvi.h' -> '/usr/include/simde/mips/msa/addvi.h'
'../simde/mips/msa/and.h' -> '/usr/include/simde/mips/msa/and.h'
'../simde/mips/msa/andi.h' -> '/usr/include/simde/mips/msa/andi.h'
'../simde/mips/msa/ld.h' -> '/usr/include/simde/mips/msa/ld.h'
'../simde/mips/msa/madd.h' -> '/usr/include/simde/mips/msa/madd.h'
'../simde/mips/msa/st.h' -> '/usr/include/simde/mips/msa/st.h'
'../simde/mips/msa/subv.h' -> '/usr/include/simde/mips/msa/subv.h'
'../simde/mips/msa/types.h' -> '/usr/include/simde/mips/msa/types.h'
'../simde/simde-aes.h' -> '/usr/include/simde/simde-aes.h'
'../simde/simde-align.h' -> '/usr/include/simde/simde-align.h'
'../simde/simde-arch.h' -> '/usr/include/simde/simde-arch.h'
'../simde/simde-bf16.h' -> '/usr/include/simde/simde-bf16.h'
'../simde/simde-common.h' -> '/usr/include/simde/simde-common.h'
'../simde/simde-complex.h' -> '/usr/include/simde/simde-complex.h'
'../simde/simde-constify.h' -> '/usr/include/simde/simde-constify.h'
'../simde/simde-detect-clang.h' -> '/usr/include/simde/simde-detect-clang.h'
'../simde/simde-diagnostic.h' -> '/usr/include/simde/simde-diagnostic.h'
'../simde/simde-f16.h' -> '/usr/include/simde/simde-f16.h'
'../simde/simde-features.h' -> '/usr/include/simde/simde-features.h'
'../simde/simde-math.h' -> '/usr/include/simde/simde-math.h'
'../simde/wasm' -> '/usr/include/simde/wasm'
'../simde/wasm/relaxed-simd.h' -> '/usr/include/simde/wasm/relaxed-simd.h'
'../simde/wasm/simd128.h' -> '/usr/include/simde/wasm/simd128.h'
'../simde/x86' -> '/usr/include/simde/x86'
'../simde/x86/aes.h' -> '/usr/include/simde/x86/aes.h'
'../simde/x86/avx.h' -> '/usr/include/simde/x86/avx.h'
'../simde/x86/avx2.h' -> '/usr/include/simde/x86/avx2.h'
'../simde/x86/avx512.h' -> '/usr/include/simde/x86/avx512.h'
'../simde/x86/avx512' -> '/usr/include/simde/x86/avx512'
'../simde/x86/avx512/2intersect.h' -> '/usr/include/simde/x86/avx512/2intersect.h'
'../simde/x86/avx512/4dpwssd.h' -> '/usr/include/simde/x86/avx512/4dpwssd.h'
'../simde/x86/avx512/4dpwssds.h' -> '/usr/include/simde/x86/avx512/4dpwssds.h'
'../simde/x86/avx512/abs.h' -> '/usr/include/simde/x86/avx512/abs.h'
'../simde/x86/avx512/add.h' -> '/usr/include/simde/x86/avx512/add.h'
'../simde/x86/avx512/adds.h' -> '/usr/include/simde/x86/avx512/adds.h'
'../simde/x86/avx512/and.h' -> '/usr/include/simde/x86/avx512/and.h'
'../simde/x86/avx512/andnot.h' -> '/usr/include/simde/x86/avx512/andnot.h'
'../simde/x86/avx512/avg.h' -> '/usr/include/simde/x86/avx512/avg.h'
'../simde/x86/avx512/bitshuffle.h' -> '/usr/include/simde/x86/avx512/bitshuffle.h'
'../simde/x86/avx512/blend.h' -> '/usr/include/simde/x86/avx512/blend.h'
'../simde/x86/avx512/broadcast.h' -> '/usr/include/simde/x86/avx512/broadcast.h'
'../simde/x86/avx512/cast.h' -> '/usr/include/simde/x86/avx512/cast.h'
'../simde/x86/avx512/cmp.h' -> '/usr/include/simde/x86/avx512/cmp.h'
'../simde/x86/avx512/cmpeq.h' -> '/usr/include/simde/x86/avx512/cmpeq.h'
'../simde/x86/avx512/cmpge.h' -> '/usr/include/simde/x86/avx512/cmpge.h'
'../simde/x86/avx512/cmpgt.h' -> '/usr/include/simde/x86/avx512/cmpgt.h'
'../simde/x86/avx512/cmple.h' -> '/usr/include/simde/x86/avx512/cmple.h'
'../simde/x86/avx512/cmplt.h' -> '/usr/include/simde/x86/avx512/cmplt.h'
'../simde/x86/avx512/cmpneq.h' -> '/usr/include/simde/x86/avx512/cmpneq.h'
'../simde/x86/avx512/compress.h' -> '/usr/include/simde/x86/avx512/compress.h'
'../simde/x86/avx512/conflict.h' -> '/usr/include/simde/x86/avx512/conflict.h'
'../simde/x86/avx512/copysign.h' -> '/usr/include/simde/x86/avx512/copysign.h'
'../simde/x86/avx512/cvt.h' -> '/usr/include/simde/x86/avx512/cvt.h'
'../simde/x86/avx512/cvts.h' -> '/usr/include/simde/x86/avx512/cvts.h'
'../simde/x86/avx512/cvtt.h' -> '/usr/include/simde/x86/avx512/cvtt.h'
'../simde/x86/avx512/cvtus.h' -> '/usr/include/simde/x86/avx512/cvtus.h'
'../simde/x86/avx512/dbsad.h' -> '/usr/include/simde/x86/avx512/dbsad.h'
'../simde/x86/avx512/div.h' -> '/usr/include/simde/x86/avx512/div.h'
'../simde/x86/avx512/dpbf16.h' -> '/usr/include/simde/x86/avx512/dpbf16.h'
'../simde/x86/avx512/dpbusd.h' -> '/usr/include/simde/x86/avx512/dpbusd.h'
'../simde/x86/avx512/dpbusds.h' -> '/usr/include/simde/x86/avx512/dpbusds.h'
'../simde/x86/avx512/dpwssd.h' -> '/usr/include/simde/x86/avx512/dpwssd.h'
'../simde/x86/avx512/dpwssds.h' -> '/usr/include/simde/x86/avx512/dpwssds.h'
'../simde/x86/avx512/expand.h' -> '/usr/include/simde/x86/avx512/expand.h'
'../simde/x86/avx512/extract.h' -> '/usr/include/simde/x86/avx512/extract.h'
'../simde/x86/avx512/fixupimm.h' -> '/usr/include/simde/x86/avx512/fixupimm.h'
'../simde/x86/avx512/fixupimm_round.h' -> '/usr/include/simde/x86/avx512/fixupimm_round.h'
'../simde/x86/avx512/flushsubnormal.h' -> '/usr/include/simde/x86/avx512/flushsubnormal.h'
'../simde/x86/avx512/fmadd.h' -> '/usr/include/simde/x86/avx512/fmadd.h'
'../simde/x86/avx512/fmaddsub.h' -> '/usr/include/simde/x86/avx512/fmaddsub.h'
'../simde/x86/avx512/fmsub.h' -> '/usr/include/simde/x86/avx512/fmsub.h'
'../simde/x86/avx512/fnmadd.h' -> '/usr/include/simde/x86/avx512/fnmadd.h'
'../simde/x86/avx512/fnmsub.h' -> '/usr/include/simde/x86/avx512/fnmsub.h'
'../simde/x86/avx512/fpclass.h' -> '/usr/include/simde/x86/avx512/fpclass.h'
'../simde/x86/avx512/gather.h' -> '/usr/include/simde/x86/avx512/gather.h'
'../simde/x86/avx512/insert.h' -> '/usr/include/simde/x86/avx512/insert.h'
'../simde/x86/avx512/kand.h' -> '/usr/include/simde/x86/avx512/kand.h'
'../simde/x86/avx512/knot.h' -> '/usr/include/simde/x86/avx512/knot.h'
'../simde/x86/avx512/kshift.h' -> '/usr/include/simde/x86/avx512/kshift.h'
'../simde/x86/avx512/kxor.h' -> '/usr/include/simde/x86/avx512/kxor.h'
'../simde/x86/avx512/load.h' -> '/usr/include/simde/x86/avx512/load.h'
'../simde/x86/avx512/loadu.h' -> '/usr/include/simde/x86/avx512/loadu.h'
'../simde/x86/avx512/lzcnt.h' -> '/usr/include/simde/x86/avx512/lzcnt.h'
'../simde/x86/avx512/madd.h' -> '/usr/include/simde/x86/avx512/madd.h'
'../simde/x86/avx512/maddubs.h' -> '/usr/include/simde/x86/avx512/maddubs.h'
'../simde/x86/avx512/max.h' -> '/usr/include/simde/x86/avx512/max.h'
'../simde/x86/avx512/min.h' -> '/usr/include/simde/x86/avx512/min.h'
'../simde/x86/avx512/mov.h' -> '/usr/include/simde/x86/avx512/mov.h'
'../simde/x86/avx512/mov_mask.h' -> '/usr/include/simde/x86/avx512/mov_mask.h'
'../simde/x86/avx512/movm.h' -> '/usr/include/simde/x86/avx512/movm.h'
'../simde/x86/avx512/mul.h' -> '/usr/include/simde/x86/avx512/mul.h'
'../simde/x86/avx512/mulhi.h' -> '/usr/include/simde/x86/avx512/mulhi.h'
'../simde/x86/avx512/mulhrs.h' -> '/usr/include/simde/x86/avx512/mulhrs.h'
'../simde/x86/avx512/mullo.h' -> '/usr/include/simde/x86/avx512/mullo.h'
'../simde/x86/avx512/multishift.h' -> '/usr/include/simde/x86/avx512/multishift.h'
'../simde/x86/avx512/negate.h' -> '/usr/include/simde/x86/avx512/negate.h'
'../simde/x86/avx512/or.h' -> '/usr/include/simde/x86/avx512/or.h'
'../simde/x86/avx512/packs.h' -> '/usr/include/simde/x86/avx512/packs.h'
'../simde/x86/avx512/packus.h' -> '/usr/include/simde/x86/avx512/packus.h'
'../simde/x86/avx512/permutex.h' -> '/usr/include/simde/x86/avx512/permutex.h'
'../simde/x86/avx512/permutex2var.h' -> '/usr/include/simde/x86/avx512/permutex2var.h'
'../simde/x86/avx512/permutexvar.h' -> '/usr/include/simde/x86/avx512/permutexvar.h'
'../simde/x86/avx512/popcnt.h' -> '/usr/include/simde/x86/avx512/popcnt.h'
'../simde/x86/avx512/range.h' -> '/usr/include/simde/x86/avx512/range.h'
'../simde/x86/avx512/range_round.h' -> '/usr/include/simde/x86/avx512/range_round.h'
'../simde/x86/avx512/rcp.h' -> '/usr/include/simde/x86/avx512/rcp.h'
'../simde/x86/avx512/reduce.h' -> '/usr/include/simde/x86/avx512/reduce.h'
'../simde/x86/avx512/rol.h' -> '/usr/include/simde/x86/avx512/rol.h'
'../simde/x86/avx512/rolv.h' -> '/usr/include/simde/x86/avx512/rolv.h'
'../simde/x86/avx512/ror.h' -> '/usr/include/simde/x86/avx512/ror.h'
'../simde/x86/avx512/rorv.h' -> '/usr/include/simde/x86/avx512/rorv.h'
'../simde/x86/avx512/round.h' -> '/usr/include/simde/x86/avx512/round.h'
'../simde/x86/avx512/roundscale.h' -> '/usr/include/simde/x86/avx512/roundscale.h'
'../simde/x86/avx512/roundscale_round.h' -> '/usr/include/simde/x86/avx512/roundscale_round.h'
'../simde/x86/avx512/sad.h' -> '/usr/include/simde/x86/avx512/sad.h'
'../simde/x86/avx512/scalef.h' -> '/usr/include/simde/x86/avx512/scalef.h'
'../simde/x86/avx512/set.h' -> '/usr/include/simde/x86/avx512/set.h'
'../simde/x86/avx512/set1.h' -> '/usr/include/simde/x86/avx512/set1.h'
'../simde/x86/avx512/set4.h' -> '/usr/include/simde/x86/avx512/set4.h'
'../simde/x86/avx512/setone.h' -> '/usr/include/simde/x86/avx512/setone.h'
'../simde/x86/avx512/setr.h' -> '/usr/include/simde/x86/avx512/setr.h'
'../simde/x86/avx512/setr4.h' -> '/usr/include/simde/x86/avx512/setr4.h'
'../simde/x86/avx512/setzero.h' -> '/usr/include/simde/x86/avx512/setzero.h'
'../simde/x86/avx512/shldv.h' -> '/usr/include/simde/x86/avx512/shldv.h'
'../simde/x86/avx512/shuffle.h' -> '/usr/include/simde/x86/avx512/shuffle.h'
'../simde/x86/avx512/sll.h' -> '/usr/include/simde/x86/avx512/sll.h'
'../simde/x86/avx512/slli.h' -> '/usr/include/simde/x86/avx512/slli.h'
'../simde/x86/avx512/sllv.h' -> '/usr/include/simde/x86/avx512/sllv.h'
'../simde/x86/avx512/sqrt.h' -> '/usr/include/simde/x86/avx512/sqrt.h'
'../simde/x86/avx512/sra.h' -> '/usr/include/simde/x86/avx512/sra.h'
'../simde/x86/avx512/srai.h' -> '/usr/include/simde/x86/avx512/srai.h'
'../simde/x86/avx512/srav.h' -> '/usr/include/simde/x86/avx512/srav.h'
'../simde/x86/avx512/srl.h' -> '/usr/include/simde/x86/avx512/srl.h'
'../simde/x86/avx512/srli.h' -> '/usr/include/simde/x86/avx512/srli.h'
'../simde/x86/avx512/srlv.h' -> '/usr/include/simde/x86/avx512/srlv.h'
'../simde/x86/avx512/store.h' -> '/usr/include/simde/x86/avx512/store.h'
'../simde/x86/avx512/storeu.h' -> '/usr/include/simde/x86/avx512/storeu.h'
'../simde/x86/avx512/sub.h' -> '/usr/include/simde/x86/avx512/sub.h'
'../simde/x86/avx512/subs.h' -> '/usr/include/simde/x86/avx512/subs.h'
'../simde/x86/avx512/ternarylogic.h' -> '/usr/include/simde/x86/avx512/ternarylogic.h'
'../simde/x86/avx512/test.h' -> '/usr/include/simde/x86/avx512/test.h'
'../simde/x86/avx512/testn.h' -> '/usr/include/simde/x86/avx512/testn.h'
'../simde/x86/avx512/types.h' -> '/usr/include/simde/x86/avx512/types.h'
'../simde/x86/avx512/unpackhi.h' -> '/usr/include/simde/x86/avx512/unpackhi.h'
'../simde/x86/avx512/unpacklo.h' -> '/usr/include/simde/x86/avx512/unpacklo.h'
'../simde/x86/avx512/xor.h' -> '/usr/include/simde/x86/avx512/xor.h'
'../simde/x86/avx512/xorsign.h' -> '/usr/include/simde/x86/avx512/xorsign.h'
'../simde/x86/clmul.h' -> '/usr/include/simde/x86/clmul.h'
'../simde/x86/f16c.h' -> '/usr/include/simde/x86/f16c.h'
'../simde/x86/fma.h' -> '/usr/include/simde/x86/fma.h'
'../simde/x86/gfni.h' -> '/usr/include/simde/x86/gfni.h'
'../simde/x86/mmx.h' -> '/usr/include/simde/x86/mmx.h'
'../simde/x86/sse.h' -> '/usr/include/simde/x86/sse.h'
'../simde/x86/sse2.h' -> '/usr/include/simde/x86/sse2.h'
'../simde/x86/sse3.h' -> '/usr/include/simde/x86/sse3.h'
'../simde/x86/sse4.1.h' -> '/usr/include/simde/x86/sse4.1.h'
'../simde/x86/sse4.2.h' -> '/usr/include/simde/x86/sse4.2.h'
'../simde/x86/ssse3.h' -> '/usr/include/simde/x86/ssse3.h'
'../simde/x86/svml.h' -> '/usr/include/simde/x86/svml.h'
'../simde/x86/xop.h' -> '/usr/include/simde/x86/xop.h'
BUILD SHOULD BE SUCCESSFUL
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets$
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets$
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets$
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets$ ./build_oai -w SIMU --gNB --ninja # Build gNB
Will compile gNB
OPENAIR_DIR    = /home/vishalkumar_shaw/oai
Running "cmake -DOAI_SIMU=ON -GNinja ../../.."
-- The C compiler identification is GNU 13.3.0
-- The CXX compiler identification is GNU 13.3.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Ccache not found. Consider installing it for faster compilation. Command: sudo apt/dnf install ccache
-- Found PkgConfig: /usr/bin/pkg-config (found version "1.8.1")
-- Check if /opt/asn1c/bin/asn1c supports -gen-APER
-- Check if /opt/asn1c/bin/asn1c supports -no-gen-UPER
-- Check if /opt/asn1c/bin/asn1c supports -no-gen-JER
-- Check if /opt/asn1c/bin/asn1c supports -no-gen-BER
-- Check if /opt/asn1c/bin/asn1c supports -no-gen-OER
-- CMAKE_BUILD_TYPE is RelWithDebInfo
-- CPU architecture is x86_64
-- AVX512 intrinsics are ON
-- AVX2 intrinsics are ON
-- Found Git: /usr/bin/git (found version "2.43.0")
-- Checking for module 'libconfig'
--   Found libconfig, version 1.5
-- LTTNG support disabled
-- Checking for module 'libcap'
--   Package 'libcap', required by 'virtual:world', not found
-- Checking for module 'openssl'
--   Found openssl, version 3.0.13
-- Checking for module 'blas'
--   Found blas, version 3.12.0
-- Checking for module 'lapacke'
--   Found lapacke, version 3.12.0
-- Checking for module 'cblas'
--   Package 'cblas', required by 'virtual:world', not found
-- No Doxygen documentation requested
-- No Support for Aerial
-- Configuring done (4.8s)
-- Generating done (1.7s)
-- Build files have been written to: /home/vishalkumar_shaw/oai/cmake_targets/ran_build/build
cd /home/vishalkumar_shaw/oai/cmake_targets/ran_build/build
Running "cmake --build .  --target  rfsimulator nr-softmodem nr-cuup params_libconfig coding rfsimulator dfts params_yaml vrtsim -- -j8"
Log file for compilation is being written to: /home/vishalkumar_shaw/oai/cmake_targets/log/all.txt
^C
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets$
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets$
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets$
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets$ sudo apt install -y libcap-dev libcblas-dev
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Package libcblas-dev is not available, but is referred to by another package.
This may mean that the package is missing, has been obsoleted, or
is only available from another source
However the following packages replace it:
  libatlas-base-dev

E: Package 'libcblas-dev' has no installation candidate
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets$ sudo apt install -y libatlas-base-dev
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages were automatically installed and are no longer required:
  libtmglib-dev libtmglib3
Use 'sudo apt autoremove' to remove them.
The following additional packages will be installed:
  libatlas3-base
Suggested packages:
  libatlas-doc liblapack-doc
The following packages will be REMOVED:
  liblapacke liblapacke-dev
The following NEW packages will be installed:
  libatlas-base-dev libatlas3-base
0 upgraded, 2 newly installed, 2 to remove and 2 not upgraded.
Need to get 7371 kB of archives.
After this operation, 25.9 MB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu noble/universe amd64 libatlas3-base amd64 3.10.3-13ubuntu1 [3544 kB]
Get:2 http://archive.ubuntu.com/ubuntu noble/universe amd64 libatlas-base-dev amd64 3.10.3-13ubuntu1 [3827 kB]
Fetched 7371 kB in 4s (2088 kB/s)
(Reading database ... 62149 files and directories currently installed.)
Removing liblapacke-dev:amd64 (3.12.0-3build1.1) ...
Removing liblapacke:amd64 (3.12.0-3build1.1) ...
Selecting previously unselected package libatlas3-base:amd64.
(Reading database ... 62132 files and directories currently installed.)
Preparing to unpack .../libatlas3-base_3.10.3-13ubuntu1_amd64.de
b ...
Unpacking libatlas3-base:amd64 (3.10.3-13ubuntu1) ...
Selecting previously unselected package libatlas-base-dev:amd64.
Preparing to unpack .../libatlas-base-dev_3.10.3-13ubuntu1_amd64
.deb ...
Unpacking libatlas-base-dev:amd64 (3.10.3-13ubuntu1) ...
Setting up libatlas3-base:amd64 (3.10.3-13ubuntu1) ...
update-alternatives: using /usr/lib/x86_64-linux-gnu/atlas/libblas.so.3 to provide /usr/lib/x86_64-linux-gnu/libblas.so.3 (libbl
as.so.3-x86_64-linux-gnu) in auto mode
update-alternatives: using /usr/lib/x86_64-linux-gnu/atlas/libla
pack.so.3 to provide /usr/lib/x86_64-linux-gnu/liblapack.so.3 (l
iblapack.so.3-x86_64-linux-gnu) in auto mode
Setting up libatlas-base-dev:amd64 (3.10.3-13ubuntu1) ...
update-alternatives: using /usr/lib/x86_64-linux-gnu/atlas/libbl
as.so to provide /usr/lib/x86_64-linux-gnu/libblas.so (libblas.s
o-x86_64-linux-gnu) in auto mode
update-alternatives: using /usr/lib/x86_64-linux-gnu/atlas/libla
pack.so to provide /usr/lib/x86_64-linux-gnu/liblapack.so (libla
pack.so-x86_64-linux-gnu) in auto mode
Processing triggers for libc-bin (2.39-0ubuntu8.4) ...
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets$ sudo apt install -y libcap-dev
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages were automatically installed and are no longer required:
  libtmglib-dev libtmglib3
Use 'sudo apt autoremove' to remove them.
The following NEW packages will be installed:
  libcap-dev
0 upgraded, 1 newly installed, 0 to remove and 2 not upgraded.
Need to get 595 kB of archives.
After this operation, 2929 kB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 libcap-dev amd64 1:2.66-5ubuntu2.2 [595 kB]
Fetched 595 kB in 2s (388 kB/s)
Selecting previously unselected package libcap-dev:amd64.
(Reading database ... 62328 files and directories currently installed.)
Preparing to unpack .../libcap-dev_1%3a2.66-5ubuntu2.2_amd64.deb
 ...
Unpacking libcap-dev:amd64 (1:2.66-5ubuntu2.2) ...
Setting up libcap-dev:amd64 (1:2.66-5ubuntu2.2) ...
Processing triggers for man-db (2.12.0-4build2) ...
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets$
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets$
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets$ ./build_oai -w SIMU --gNB --ninja # Build gNB
Will compile gNB
OPENAIR_DIR    = /home/vishalkumar_shaw/oai
Running "cmake -DOAI_SIMU=ON -GNinja ../../.."
-- Ccache not found. Consider installing it for faster compilation. Command: sudo apt/dnf install ccache
-- Check if /opt/asn1c/bin/asn1c supports -gen-APER
-- Check if /opt/asn1c/bin/asn1c supports -no-gen-UPER
-- Check if /opt/asn1c/bin/asn1c supports -no-gen-JER
-- Check if /opt/asn1c/bin/asn1c supports -no-gen-BER
-- Check if /opt/asn1c/bin/asn1c supports -no-gen-OER
-- CMAKE_BUILD_TYPE is RelWithDebInfo
-- CPU architecture is x86_64
-- AVX512 intrinsics are ON
-- AVX2 intrinsics are ON
-- LTTNG support disabled
-- Checking for module 'libcap'
--   Found libcap, version 2.66
-- Checking for module 'cblas'
--   Package 'cblas', required by 'virtual:world', not found
-- No Doxygen documentation requested
-- No Support for Aerial
-- Configuring done (0.5s)
-- Generating done (1.4s)
-- Build files have been written to: /home/vishalkumar_shaw/oai/cmake_targets/ran_build/build
cd /home/vishalkumar_shaw/oai/cmake_targets/ran_build/build
Running "cmake --build .  --target  rfsimulator nr-softmodem nr-cuup params_libconfig coding rfsimulator dfts params_yaml vrtsim -- -j8"
Log file for compilation is being written to: /home/vishalkumar_shaw/oai/cmake_targets/log/all.txt
 rfsimulator nr-softmodem nr-cuup params_libconfig coding rfsimulator dfts params_yaml vrtsim compiled
BUILD SHOULD BE SUCCESSFUL
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets$
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets$
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets$ ./build_oai -w SIMU --nrUE --ninja # Build nrUE
Will compile NR UE
OPENAIR_DIR    = /home/vishalkumar_shaw/oai
Running "cmake -DOAI_SIMU=ON -GNinja ../../.."
-- Ccache not found. Consider installing it for faster compilation. Command: sudo apt/dnf install ccache
-- Check if /opt/asn1c/bin/asn1c supports -gen-APER
-- Check if /opt/asn1c/bin/asn1c supports -no-gen-UPER
-- Check if /opt/asn1c/bin/asn1c supports -no-gen-JER
-- Check if /opt/asn1c/bin/asn1c supports -no-gen-BER
-- Check if /opt/asn1c/bin/asn1c supports -no-gen-OER
-- CMAKE_BUILD_TYPE is RelWithDebInfo
-- CPU architecture is x86_64
-- AVX512 intrinsics are ON
-- AVX2 intrinsics are ON
-- LTTNG support disabled
-- Checking for module 'cblas'
--   Package 'cblas', required by 'virtual:world', not found
-- No Doxygen documentation requested
-- No Support for Aerial
-- Configuring done (0.3s)
-- Generating done (1.5s)
-- Build files have been written to: /home/vishalkumar_shaw/oai/cmake_targets/ran_build/build
cd /home/vishalkumar_shaw/oai/cmake_targets/ran_build/build
Running "cmake --build .  --target  rfsimulator nr-uesoftmodem params_libconfig coding rfsimulator dfts params_yaml vrtsim -- -j8"
Log file for compilation is being written to: /home/vishalkumar_shaw/oai/cmake_targets/log/all.txt
 rfsimulator nr-uesoftmodem params_libconfig coding rfsimulator dfts params_yaml vrtsim compiled
BUILD SHOULD BE SUCCESSFUL
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets$
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets$
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets$ ./build_oai --build-e2 --ninja    # Build E2 components
OPENAIR_DIR    = /home/vishalkumar_shaw/oai
BUILD SHOULD BE SUCCESSFUL
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets$
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets$
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets$
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets$
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets$

vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets$ # Navigate to configuration directory
cd ~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF/
vishalkumar_shaw@IN-8KBKXD3:~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF$ ls
channelmod_rfsimu_LEO_satellite.conf
cu_gnb.conf
du_gnb.conf
gnb-cu.sa.f1.conf
gnb-du.sa.band66.25prb.rfsim.pci0.conf
gnb-du.sa.band66.25prb.rfsim.pci1.conf
gnb-du.sa.band77.273prb.fhi72.4x4-benetel650.conf
gnb-du.sa.band77.273prb.fhi72.8x8-benetel650_650-mplane.conf
gnb-du.sa.band77.273prb.fhi72.8x8-benetel650_650.conf
gnb-du.sa.band78.106prb.rfsim.pci0.conf
gnb-du.sa.band78.106prb.rfsim.pci1.conf
gnb-pnf.band78.rfsim.2x2.conf
gnb-pnf.band78.rfsim.conf
gnb-vnf.sa.band78.106prb.nfapi.conf
gnb-vnf.sa.band78.273prb.aerial.conf
gnb-vnf.sa.band78.273prb.nfapi.conf
gnb-vnf.sa.cbrs.aerial.conf
gnb.band78.sa.fr1.106PRB.2x2.usrpn310.conf
gnb.band78.sa.fr1.162PRB.2x2.usrpn310.conf
gnb.sa.band1.u0.52PRB.usrpb210.conf
gnb.sa.band254.ntn.mu1.24prb.rfsim.conf
gnb.sa.band255.ntn.mu0.25prb.rfsim.conf
gnb.sa.band256.ntn.mu0.25prb.rfsim.conf
gnb.sa.band257.u3.32prb.usrpx410.conf
gnb.sa.band41.fr1.106PRB.usrpb210.conf
gnb.sa.band41.fr1.52PRB.usrpb210.conf
gnb.sa.band66.fr1.106PRB.usrpn300.conf
gnb.sa.band66.fr1.106PRB.usrpx300.conf
gnb.sa.band66.fr1.24PRB.usrpx300.conf
gnb.sa.band66.fr1.25PRB.usrpx300.conf
gnb.sa.band66.u0.25prb.rfsim.conf
gnb.sa.band77.106prb.fhi72.4x4-vvdn.conf
gnb.sa.band77.162prb.usrpn310.4x4.conf
gnb.sa.band77.273prb.fhi72.2x2-vvdn.conf
gnb.sa.band77.273prb.fhi72.4x4-vvdn.conf
gnb.sa.band77.fr1.273PRB.2x2.usrpn300.conf
gnb.sa.band77.fr1.273PRB.usrpx300.conf
gnb.sa.band78.106PRB.usrpb210.RA_2-Step.conf
gnb.sa.band78.273prb.fhi72.4x2-benetel550.conf
gnb.sa.band78.273prb.fhi72.4x4-benetel550-mplane.conf
gnb.sa.band78.273prb.fhi72.4x4-benetel550.conf
gnb.sa.band78.273prb.fhi72.4x4-foxconn.conf
gnb.sa.band78.273prb.fhi72.4x4-liteon.conf
gnb.sa.band78.273prb.fhi72.4x4-metanoia.conf
gnb.sa.band78.fr1.106PRB.2x2.usrpn300.conf
gnb.sa.band78.fr1.106PRB.usrpb210.2pattern.conf
gnb.sa.band78.fr1.106PRB.usrpb210.4layer.conf
gnb.sa.band78.fr1.106PRB.usrpb210.conf
gnb.sa.band78.fr1.106PRB.usrpb210.redcap.yaml
gnb.sa.band78.fr1.106PRB.usrpb210.sabox.conf
gnb.sa.band78.fr1.106PRB.usrpb210.yaml
gnb.sa.band78.fr1.162PRB.2x2.usrpn300.conf
gnb.sa.band78.fr1.189PRB.rfsim.conf
gnb.sa.band78.fr1.217PRB.2x2.usrpn300.conf
gnb.sa.band78.fr1.24PRB.usrpb210.conf
gnb0.prs.band261.fr2.64PRB.usrpx310.conf
gnb0.prs.band78.fr1.106PRB.usrpx310.conf
gnb1.prs.band261.fr2.64PRB.usrpx310.conf
gnb1.prs.band78.fr1.106PRB.usrpx310.conf
ue.conf
ue.nr.prs.fr1.106prb.conf
ue.nr.prs.fr2.64prb.conf
uecap_ports1.xml
uecap_ports2.xml
uecap_ports4.xml
vishalkumar_shaw@IN-8KBKXD3:~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF$ # Copy and modify gNB configuration
cp gnb.sa.band78.fr1.106PRB.usrpb210.conf my_gnb.conf
vishalkumar_shaw@IN-8KBKXD3:~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF$ nano my_gnb.conf
vishalkumar_shaw@IN-8KBKXD3:~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF$ # Copy UE configuration
cp ue.conf my_ue.conf
vishalkumar_shaw@IN-8KBKXD3:~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF$
vishalkumar_shaw@IN-8KBKXD3:~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF$
vishalkumar_shaw@IN-8KBKXD3:~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF$
vishalkumar_shaw@IN-8KBKXD3:~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF$ nano my_gnb.conf
vishalkumar_shaw@IN-8KBKXD3:~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF$ rm my_gnb.conf
vishalkumar_shaw@IN-8KBKXD3:~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF$ cp gnb.sa.band78.fr1.106PRB.usrpb210.conf my_gnb.conf
vishalkumar_shaw@IN-8KBKXD3:~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF$
vishalkumar_shaw@IN-8KBKXD3:~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF$
vishalkumar_shaw@IN-8KBKXD3:~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF$
vishalkumar_shaw@IN-8KBKXD3:~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF$ # Check if gNodeB is running
ps aux | grep nr-softmodem
root      157786  0.0  0.0  14156  6912 pts/2    S+   07:20   0:00 sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/my_gnb.conf --rfsim --sa
root      157787  0.0  0.0  14156  2484 pts/3    Ss   07:20   0:00 sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/my_gnb.conf --rfsim --sa
root      157788 28.1 11.0 2119772 876884 pts/3  SLl+ 07:20   0:43 ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/my_gnb.conf --rfsim --sa
root      157805  0.0  0.0  70056  7164 pts/3    S+   07:20   0:00 ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/my_gnb.conf --rfsim --sa
vishalk+  160620  0.0  0.0   4092  2048 pts/0    S+   07:23   0:00 grep --color=auto nr-softmodem
vishalkumar_shaw@IN-8KBKXD3:~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF$
vishalkumar_shaw@IN-8KBKXD3:~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF$
vishalkumar_shaw@IN-8KBKXD3:~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF$
vishalkumar_shaw@IN-8KBKXD3:~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF$ # Check network interfaces
ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet 10.255.255.254/32 brd 10.255.255.254 scope global lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:75:e6:a5 brd ff:ff:ff:ff:ff:ff
    inet 172.28.251.212/20 brd 172.28.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::215:5dff:fe75:e6a5/64 scope link
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 86:d3:55:73:f8:07 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 brd 172.18.255.255 scope global docker0
       valid_lft forever preferred_lft forever
4: br-49c47fbd63f4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 3a:46:a7:42:32:09 brd ff:ff:ff:ff:ff:ff
    inet 172.19.0.1/16 brd 172.19.255.255 scope global br-49c47fbd63f4
       valid_lft forever preferred_lft forever
    inet6 fc00:f853:ccd:e793::1/64 scope global nodad
       valid_lft forever preferred_lft forever
    inet6 fe80::3846:a7ff:fe42:3209/64 scope link
       valid_lft forever preferred_lft forever
5: br-98da745fb3ef: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 5e:d8:d3:33:d6:b0 brd ff:ff:ff:ff:ff:ff
    inet 192.168.49.1/24 brd 192.168.49.255 scope global br-98da745fb3ef
       valid_lft forever preferred_lft forever
6: br-c08dd3386fe5: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 7e:4f:95:2e:c2:74 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global br-c08dd3386fe5
       valid_lft forever preferred_lft forever
9: tunl0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/ipip 0.0.0.0 brd 0.0.0.0
12: veth1bfba09@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-49c47fbd63f4 state UP group default
    link/ether 22:59:ca:30:10:12 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::2059:caff:fe30:1012/64 scope link
       valid_lft forever preferred_lft forever
13: veth2484af1@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-49c47fbd63f4 state UP group default
    link/ether 2e:fc:0d:74:ae:a0 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::2cfc:dff:fe74:aea0/64 scope link
       valid_lft forever preferred_lft forever
vishalkumar_shaw@IN-8KBKXD3:~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF$
vishalkumar_shaw@IN-8KBKXD3:~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF$
vishalkumar_shaw@IN-8KBKXD3:~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF$
vishalkumar_shaw@IN-8KBKXD3:~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF$ # Check UE status
ps aux | grep nr-uesoftmodem
vishalk+  161440  0.0  0.0   4092  2048 pts/0    S+   07:24   0:00 grep --color=auto nr-uesoftmodem
vishalkumar_shaw@IN-8KBKXD3:~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF$
vishalkumar_shaw@IN-8KBKXD3:~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF$
vishalkumar_shaw@IN-8KBKXD3:~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF$
vishalkumar_shaw@IN-8KBKXD3:~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF$ # Check if UE interface is created
ip addr show oaitun_ue1
Device "oaitun_ue1" does not exist.
vishalkumar_shaw@IN-8KBKXD3:~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF$
vishalkumar_shaw@IN-8KBKXD3:~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF$
vishalkumar_shaw@IN-8KBKXD3:~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF$ # Ping test from UE
ping -I oaitun_ue1 8.8.8.8
ping: SO_BINDTODEVICE oaitun_ue1: No such device
vishalkumar_shaw@IN-8KBKXD3:~/oai/targets/PROJECTS/GENERIC-NR-5GC/CONF$
```

### gNB

```bash
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets/ran_build/build$ sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/my_gnb.conf --rfsim --sa
CMDLINE: "./nr-softmodem" "-O" "../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/my_gnb.conf" "--rfsim" "--sa"
[CONFIG] function config_libconfig_init returned 0
[UTIL]   running in SA mode (no --phy-test, --do-ra, --nsa option present)
[OPT]   OPT disabled
[HW]   Version: Branch: develop Abrev. Hash: d71e333329 Date: Tue Jul 8 15:05:31 2025 +0000
[GNB_APP]   Initialized RAN Context: RC.nb_nr_inst = 1, RC.nb_nr_macrlc_inst = 1, RC.nb_nr_L1_inst = 1, RC.nb_RU = 1, RC.nb_nr_CC[0] = 1
[NR_PHY]   Initializing gNB RAN context: RC.nb_nr_L1_inst = 1
[NR_PHY]   Registered with MAC interface module (0x64924f94e7e0)
[NR_PHY]   Initializing NR L1: RC.nb_nr_L1_inst = 1
[NR_PHY]   L1_RX_THREAD_CORE -1 (15)
[NR_PHY]   TX_AMP = 519 (-36 dBFS)
[PHY]   No prs_config configuration found..!!
[GNB_APP]   pdsch_AntennaPorts N1 1 N2 1 XP 1 pusch_AntennaPorts 1
[GNB_APP]   minTXRXTIME 2
[GNB_APP]   SIB1 TDA 1
[GNB_APP]   CSI-RS 1, SRS 1, SINR:0, 256 QAM may be on, delta_MCS off, maxMIMO_Layers -1, HARQ feedback enabled, num DLHARQ:16, num ULHARQ:16
[NR_MAC]   No RedCap configuration found
[GNB_APP]   sr_ProhibitTimer 0, sr_TransMax 64, sr_ProhibitTimer_v1700 0, t300 400, t301 400, t310 2000, n310 10, t311 3000, n311 1, t319 400
[NR_MAC]   Candidates per PDCCH aggregation level on UESS: L1: 0, L2: 2, L4: 0, L8: 0, L16: 0
[RRC]   Read in ServingCellConfigCommon (PhysCellId 0, ABSFREQSSB 641280, DLBand 78, ABSFREQPOINTA 640008, DLBW 106,RACH_TargetReceivedPower -96
[RRC]   absoluteFrequencySSB 641280 corresponds to 3619200000 Hz
[NR_MAC]   TDD period index = 6, based on the sum of dl_UL_TransmissionPeriodicity from Pattern1 (5.000000 ms) and Pattern2 (0.000000 ms): Total = 5.000000 ms
[UTIL]   threadCreate() for MAC_STATS: creating thread with affinity ffffffff, priority 2
[NR_MAC]   PUSCH Target 150, PUCCH Target 200, PUCCH Failure 10, PUSCH Failure 10
[NR_PHY]   Copying 0 blacklisted PRB to L1 context
[NR_MAC]   Set TX antenna number to 1, Set RX antenna number to 1 (num ssb 1: 80000000,0)
[NR_MAC]   TDD period index = 6, based on the sum of dl_UL_TransmissionPeriodicity from Pattern1 (5.000000 ms) and Pattern2 (0.000000 ms): Total = 5.000000 ms
[NR_MAC]   Set TDD configuration period to: 8 DL slots, 3 UL slots, 10 slots per period (NR_TDD_UL_DL_Pattern is 7 DL slots, 2 UL slots, 6 DL symbols, 4 UL symbols)
[NR_MAC]   Configured 1 TDD patterns (total slots: pattern1 = 10, pattern2 = 0)
[NR_PHY]   Set TDD Period Configuration: 2 periods per frame, 20 slots to be configured (8 DL, 3 UL)
[NR_PHY]   TDD period configuration: slot 0 is DOWNLINK
[NR_PHY]   TDD period configuration: slot 1 is DOWNLINK
[NR_PHY]   TDD period configuration: slot 2 is DOWNLINK
[NR_PHY]   TDD period configuration: slot 3 is DOWNLINK
[NR_PHY]   TDD period configuration: slot 4 is DOWNLINK
[NR_PHY]   TDD period configuration: slot 5 is DOWNLINK
[NR_PHY]   TDD period configuration: slot 6 is DOWNLINK
[NR_PHY]   TDD period configuration: slot 7 is FLEXIBLE: DDDDDDFFFFUUUU
[NR_PHY]   TDD period configuration: slot 8 is UPLINK
[NR_PHY]   TDD period configuration: slot 9 is UPLINK
DL frequency 3619200000: band 48, UL frequency 3619200000
[PHY]   DL frequency 3619200000 Hz, UL frequency 3619200000 Hz: band 48, uldl offset 0 Hz
[PHY]   Initializing frame parms for mu 1, N_RB 106, Ncp 0
[PHY]   Init: N_RB_DL 106, first_carrier_offset 1412, nb_prefix_samples 144,nb_prefix_samples0 176, ofdm_symbol_size 2048
[NR_RRC]   SIB1 freq: offsetToPointA 86
[GNB_APP]   F1AP: gNB idx 0 gNB_DU_id 3584, gNB_DU_name gNB-OAI, TAC 1 MCC/MNC/length 1/1/2 cellID 12345678
[GNB_APP]   ngran_DU: Configuring Cell 0 for TDD
[GNB_APP]   SDAP layer is disabled
[GNB_APP]   Data Radio Bearer count 1
[GNB_APP]   Parsed IPv4 address for NG AMF: 192.168.70.129
[UTIL]   threadCreate() for TASK_SCTP: creating thread with affinity ffffffff, priority 50
[X2AP]   X2AP is disabled.
[UTIL]   threadCreate() for TASK_NGAP: creating thread with affinity ffffffff, priority 50
[NGAP]   Starting NGAP layer
[UTIL]   threadCreate() for TASK_RRC_GNB: creating thread with affinity ffffffff, priority 50
[NGAP]   Registered new gNB[0] and macro gNB id 3584
[NGAP]   [gNB 0] check the amf registration state
[GTPU]   Configuring GTPu
[GTPU]   SA mode
[GTPU]   Configuring GTPu address : 192.168.70.129, port : 2152
[GTPU]   Initializing UDP for local address 192.168.70.129 with port 2152
[NR_RRC]   Entering main loop of NR_RRC message task
[GTPU]   bind: Cannot assign requested address
[GTPU]   failed to bind socket: 192.168.70.129 2152
[GTPU]   can't create GTP-U instance
[GTPU]   Created gtpu instance id: -1
[E1AP]   Failed to create CUUP N3 UDP listener
[UTIL]   threadCreate() for TASK_GNB_APP: creating thread with affinity ffffffff, priority 50
[NR_RRC]   Accepting new CU-UP ID 3584 name gNB-OAI (assoc_id -1)
[UTIL]   threadCreate() for TASK_GTPV1_U: creating thread with affinity ffffffff, priority 50
[NR_RRC]   Received F1 Setup Request from gNB_DU 3584 (gNB-OAI) on assoc_id -1
[NR_RRC]   Accepting DU 3584 (gNB-OAI), sending F1 Setup Response
[NR_RRC]   DU uses RRC version 17.3.0
[MAC]   received F1 Setup Response from CU (null)
[MAC]   CU uses RRC version 17.3.0
[MAC]   Clearing the DU's UE states before, if any.
[NR_RRC]   cell PLMN 001.01 Cell ID 12345678 is in service
[MAC]   received gNB-DU configuration update acknowledge
[UTIL]   threadCreate() for time source iq samples: creating thread with affinity ffffffff, priority 2
[UTIL]   time manager configuration: [time source: iq_samples] [mode: standalone] [server IP: 127.0.0.1} [server port: 7374] (server IP/port not used)
[SCTP]   sctp_bindx() SCTP_BINDX_ADD_ADDR failed: errno 99 Cannot assign requested address
[SCTP]   could not open socket, no SCTP connection established
[PHY]   RU clock source set as internal
[PHY]   number of L1 instances 1, number of RU 1, number of CPU cores 8
[PHY]   Initialized RU proc 0 (,synch_to_ext_device),
[PHY]   RU thread-pool core string -1,-1 (size 2)
[UTIL]   threadCreate() for Tpool0_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool1_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for ru_thread: creating thread with affinity ffffffff, priority 97
[PHY]   Starting RU 0 (,synch_to_ext_device) on cpu 5
[PHY]   Initializing frame parms for mu 1, N_RB 106, Ncp 0
[PHY]   Init: N_RB_DL 106, first_carrier_offset 1412, nb_prefix_samples 144,nb_prefix_samples0 176, ofdm_symbol_size 2048
[PHY]   fp->scs=30000
[PHY]   fp->ofdm_symbol_size=2048
[PHY]   fp->nb_prefix_samples0=176
[PHY]   fp->nb_prefix_samples=144
[PHY]   fp->slots_per_subframe=2
[PHY]   fp->samples_per_subframe_wCP=57344
[PHY]   fp->samples_per_frame_wCP=573440
[PHY]   fp->samples_per_subframe=61440
[PHY]   fp->samples_per_frame=614400
[PHY]   fp->dl_CarrierFreq=3619200000
[PHY]   fp->ul_CarrierFreq=3619200000
[PHY]   fp->Nid_cell=0
[PHY]   fp->first_carrier_offset=1412
[PHY]   fp->ssb_start_subcarrier=0
[PHY]   fp->Ncp=0
[PHY]   fp->N_RB_DL=106
[PHY]   fp->numerology_index=1
[PHY]   fp->nr_band=48
[PHY]   fp->ofdm_offset_divisor=8
[PHY]   fp->threequarter_fs=0
[PHY]   fp->sl_CarrierFreq=0
[PHY]   fp->N_RB_SL=0
[NR_PHY]   nb_tx_streams 1, nb_rx_streams 1, num_Beams_period 1
[PHY]   Setting RF config for N_RB 106, NB_RX 1, NB_TX 1
[PHY]   tune_offset 0 Hz, sample_rate 61440000 Hz
[PHY]   Channel 0: setting tx_gain offset 12, tx_freq 3619200000 Hz
[PHY]   Channel 0: setting rx_gain offset 102, rx_freq 3619200000 Hz
[HW]   Running as server waiting opposite rfsimulators to connect
Initializing random number generator, seed 6458763154511634255
[HW]   [RAU] has loaded RFSIMULATOR device.
[PHY]   RU 0 Setting N_TA_offset to 800 samples (UL Freq 3600120, N_RB 106, mu 1)
[PHY]   Signaling main thread that RU 0 is ready, sl_ahead 6
[PHY]   L1 configured without analog beamforming
[PHY]   Attaching RU 0 antenna 0 to gNB antenna 0
[UTIL]   threadCreate() for Tpool0_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool1_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool2_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool3_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool4_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool5_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool6_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool7_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for L1_rx_thread: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for L1_tx_thread: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for L1_stats: creating thread with affinity ffffffff, priority 1
TYPE <CTRL-C> TO TERMINATE
[PHY]   got sync (L1_stats_thread)
[PHY]   got sync (ru_thread)
[PHY]   RU 0 rf device ready
[PHY]   RU 0 RF started cpu_meas_enabled 0
[HW]   No connected device, generating void samples...
[PHY]   Command line parameters for OAI UE: -C 3619200000 -r 106 --numerology 1 --ssb 516
[NR_MAC]   Frame.Slot 128.0

[NR_MAC]   Frame.Slot 256.0

[NR_MAC]   Frame.Slot 384.0

[NR_MAC]   Frame.Slot 512.0

[NR_MAC]   Frame.Slot 640.0

[NR_MAC]   Frame.Slot 768.0

[NR_MAC]   Frame.Slot 896.0

[NR_MAC]   Frame.Slot 0.0

[NR_MAC]   Frame.Slot 128.0

[NR_MAC]   Frame.Slot 256.0

[NR_MAC]   Frame.Slot 384.0

[NR_MAC]   Frame.Slot 512.0

[NR_MAC]   Frame.Slot 640.0

[NR_MAC]   Frame.Slot 768.0

[NR_MAC]   Frame.Slot 896.0

[NR_MAC]   Frame.Slot 0.0

[NR_MAC]   Frame.Slot 128.0

[NR_MAC]   Frame.Slot 256.0

[NR_MAC]   Frame.Slot 384.0

[NR_MAC]   Frame.Slot 512.0

[NR_MAC]   Frame.Slot 640.0

[NR_MAC]   Frame.Slot 768.0

[NR_MAC]   Frame.Slot 896.0

[NR_MAC]   Frame.Slot 0.0

[NR_MAC]   Frame.Slot 128.0

[NR_MAC]   Frame.Slot 256.0

[NR_MAC]   Frame.Slot 384.0

[NR_MAC]   Frame.Slot 512.0

[NR_MAC]   Frame.Slot 640.0

[NR_MAC]   Frame.Slot 768.0

[NR_MAC]   Frame.Slot 896.0

[NR_MAC]   Frame.Slot 0.0

[NR_MAC]   Frame.Slot 128.0

[NR_MAC]   Frame.Slot 256.0

[NR_MAC]   Frame.Slot 384.0

[NR_MAC]   Frame.Slot 512.0

[NR_MAC]   Frame.Slot 640.0

[HW]   A client connects, sending the current time
[HW]   Not supported to send Tx out of order 2956721024, 2956721023
[NR_MAC]   Frame.Slot 768.0

[NR_MAC]   Frame.Slot 896.0

[NR_PHY]   [RAPROC] 903.19 Initiating RA procedure with preamble 30, energy 43.0 dB (I0 0, thres 120), delay 0 start symbol 0 freq index 0
[NR_MAC]   903.19 UE RA-RNTI 010b TC-RNTI affe: initiating RA procedure
[NR_MAC]   UE affe: Msg3 scheduled at 904.17 (904.7 TDA 3) start 0 RBs 8
[NR_MAC]   UE affe: 904.7 Generating RA-Msg2 DCI, RA RNTI 0x10b, state 1, preamble_index(RAPID) 30, timing_offset = 0 (estimated distance 0.0 [m])
[NR_MAC]   904.7 Send RAR to RA-RNTI 010b
[NR_MAC]    904.17 PUSCH with TC_RNTI 0xaffe received correctly
[MAC]   [RAPROC] Received SDU for CCCH length 6 for UE affe
[RLC]   Activated srb0 for UE 45054
[RLC]   Added srb 1 to UE 45054
[NR_MAC]   Activating scheduling Msg4 for TC_RNTI 0xaffe (state WAIT_Msg3)
[NR_RRC]   Decoding CCCH: RNTI affe, payload_size 6
[NR_RRC]   [--] (cellID 0, UE ID 1 RNTI affe) Create UE context: CU UE ID 1 DU UE ID 45054 (rnti: affe, random ue id bccbc453e5000000)
[RRC]   activate SRB 1 of UE 1
[NR_RRC]   [DL] (cellID bc614e, UE ID 1 RNTI affe) Send RRC Setup
[NR_MAC]   UE affe Generate Msg4: feedback at  905.17, payload 225 bytes, next state nrRA_WAIT_Msg4_MsgB_ACK
[NR_MAC]    905.17 UE affe: Received Ack of Msg4. CBRA procedure succeeded (UE Connected)
[NR_MAC]   Adding new UE context with RNTI 0xaffe
[HW]   Lost socket
[HW]   write() failed, errno(104)
[HW]   write() failed, errno(9)
[HW]   write() failed, errno(9)
[NR_MAC]   UE affe: Detected UL Failure on PUSCH after 10 PUSCH DTX, stopping scheduling
[NR_MAC]   Frame.Slot 0.0
UE RNTI affe CU-UE-ID 1 out-of-sync PH 0 dB PCMAX 0 dBm, average RSRP 0 (0 meas)
UE affe: dlsch_rounds 1/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE affe: ulsch_rounds 3/3/2/2, ulsch_errors 2, ulsch_DTX 10, BLER 0.27100 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 0  SNR 50.5 dB
UE affe: MAC:    TX              0 RX              0 bytes
UE affe: LCID 1: TX              0 RX              0 bytes

[NR_MAC]   Frame.Slot 128.0
UE RNTI affe CU-UE-ID 1 out-of-sync PH 0 dB PCMAX 0 dBm, average RSRP 0 (0 meas)
UE affe: dlsch_rounds 1/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE affe: ulsch_rounds 3/3/2/2, ulsch_errors 2, ulsch_DTX 10, BLER 0.27100 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 0  SNR 50.5 dB
UE affe: MAC:    TX              0 RX              0 bytes
UE affe: LCID 1: TX              0 RX              0 bytes

[NR_MAC]   Frame.Slot 256.0
UE RNTI affe CU-UE-ID 1 out-of-sync PH 0 dB PCMAX 0 dBm, average RSRP 0 (0 meas)
UE affe: dlsch_rounds 1/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE affe: ulsch_rounds 3/3/2/2, ulsch_errors 2, ulsch_DTX 10, BLER 0.27100 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 0  SNR 50.5 dB
UE affe: MAC:    TX              0 RX              0 bytes
UE affe: LCID 1: TX              0 RX              0 bytes

[NR_MAC]   Frame.Slot 384.0
UE RNTI affe CU-UE-ID 1 out-of-sync PH 0 dB PCMAX 0 dBm, average RSRP 0 (0 meas)
UE affe: dlsch_rounds 1/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE affe: ulsch_rounds 3/3/2/2, ulsch_errors 2, ulsch_DTX 10, BLER 0.27100 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 0  SNR 50.5 dB
UE affe: MAC:    TX              0 RX              0 bytes
UE affe: LCID 1: TX              0 RX              0 bytes

[NR_MAC]   Frame.Slot 512.0
UE RNTI affe CU-UE-ID 1 out-of-sync PH 0 dB PCMAX 0 dBm, average RSRP 0 (0 meas)
UE affe: dlsch_rounds 1/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE affe: ulsch_rounds 3/3/2/2, ulsch_errors 2, ulsch_DTX 10, BLER 0.27100 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 0  SNR 50.5 dB
UE affe: MAC:    TX              0 RX              0 bytes
UE affe: LCID 1: TX              0 RX              0 bytes

[NR_MAC]   Frame.Slot 640.0
UE RNTI affe CU-UE-ID 1 out-of-sync PH 0 dB PCMAX 0 dBm, average RSRP 0 (0 meas)
UE affe: dlsch_rounds 1/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE affe: ulsch_rounds 3/3/2/2, ulsch_errors 2, ulsch_DTX 10, BLER 0.27100 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 0  SNR 50.5 dB
UE affe: MAC:    TX              0 RX              0 bytes
UE affe: LCID 1: TX              0 RX              0 bytes

[NR_MAC]   Frame.Slot 768.0
UE RNTI affe CU-UE-ID 1 out-of-sync PH 0 dB PCMAX 0 dBm, average RSRP 0 (0 meas)
UE affe: dlsch_rounds 1/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE affe: ulsch_rounds 3/3/2/2, ulsch_errors 2, ulsch_DTX 10, BLER 0.27100 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 0  SNR 50.5 dB
UE affe: MAC:    TX              0 RX              0 bytes
UE affe: LCID 1: TX              0 RX              0 bytes

[NR_MAC]   Frame.Slot 896.0
UE RNTI affe CU-UE-ID 1 out-of-sync PH 0 dB PCMAX 0 dBm, average RSRP 0 (0 meas)
UE affe: dlsch_rounds 1/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE affe: ulsch_rounds 3/3/2/2, ulsch_errors 2, ulsch_DTX 10, BLER 0.27100 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 0  SNR 50.5 dB
UE affe: MAC:    TX              0 RX              0 bytes
UE affe: LCID 1: TX              0 RX              0 bytes

[NR_MAC]   Frame.Slot 0.0
UE RNTI affe CU-UE-ID 1 out-of-sync PH 0 dB PCMAX 0 dBm, average RSRP 0 (0 meas)
UE affe: dlsch_rounds 1/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE affe: ulsch_rounds 3/3/2/2, ulsch_errors 2, ulsch_DTX 10, BLER 0.27100 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 0  SNR 50.5 dB
UE affe: MAC:    TX              0 RX              0 bytes
UE affe: LCID 1: TX              0 RX              0 bytes

[NR_MAC]   Frame.Slot 128.0
UE RNTI affe CU-UE-ID 1 out-of-sync PH 0 dB PCMAX 0 dBm, average RSRP 0 (0 meas)
UE affe: dlsch_rounds 1/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE affe: ulsch_rounds 3/3/2/2, ulsch_errors 2, ulsch_DTX 10, BLER 0.27100 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 0  SNR 50.5 dB
UE affe: MAC:    TX              0 RX              0 bytes
UE affe: LCID 1: TX              0 RX              0 bytes

[NR_MAC]   Frame.Slot 256.0
UE RNTI affe CU-UE-ID 1 out-of-sync PH 0 dB PCMAX 0 dBm, average RSRP 0 (0 meas)
UE affe: dlsch_rounds 1/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE affe: ulsch_rounds 3/3/2/2, ulsch_errors 2, ulsch_DTX 10, BLER 0.27100 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 0  SNR 50.5 dB
UE affe: MAC:    TX              0 RX              0 bytes
UE affe: LCID 1: TX              0 RX              0 bytes

[NR_MAC]   Frame.Slot 384.0
UE RNTI affe CU-UE-ID 1 out-of-sync PH 0 dB PCMAX 0 dBm, average RSRP 0 (0 meas)
UE affe: dlsch_rounds 1/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE affe: ulsch_rounds 3/3/2/2, ulsch_errors 2, ulsch_DTX 10, BLER 0.27100 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 0  SNR 50.5 dB
UE affe: MAC:    TX              0 RX              0 bytes
UE affe: LCID 1: TX              0 RX              0 bytes

[NR_MAC]   Frame.Slot 512.0
UE RNTI affe CU-UE-ID 1 out-of-sync PH 0 dB PCMAX 0 dBm, average RSRP 0 (0 meas)
UE affe: dlsch_rounds 1/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE affe: ulsch_rounds 3/3/2/2, ulsch_errors 2, ulsch_DTX 10, BLER 0.27100 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 0  SNR 50.5 dB
UE affe: MAC:    TX              0 RX              0 bytes
UE affe: LCID 1: TX              0 RX              0 bytes

[NR_MAC]   Frame.Slot 640.0
UE RNTI affe CU-UE-ID 1 out-of-sync PH 0 dB PCMAX 0 dBm, average RSRP 0 (0 meas)
UE affe: dlsch_rounds 1/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE affe: ulsch_rounds 3/3/2/2, ulsch_errors 2, ulsch_DTX 10, BLER 0.27100 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 0  SNR 50.5 dB
UE affe: MAC:    TX              0 RX              0 bytes
UE affe: LCID 1: TX              0 RX              0 bytes

[NR_MAC]   Frame.Slot 768.0
UE RNTI affe CU-UE-ID 1 out-of-sync PH 0 dB PCMAX 0 dBm, average RSRP 0 (0 meas)
UE affe: dlsch_rounds 1/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE affe: ulsch_rounds 3/3/2/2, ulsch_errors 2, ulsch_DTX 10, BLER 0.27100 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 0  SNR 50.5 dB
UE affe: MAC:    TX              0 RX              0 bytes
UE affe: LCID 1: TX              0 RX              0 bytes

[NR_MAC]   Frame.Slot 896.0
UE RNTI affe CU-UE-ID 1 out-of-sync PH 0 dB PCMAX 0 dBm, average RSRP 0 (0 meas)
UE affe: dlsch_rounds 1/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE affe: ulsch_rounds 3/3/2/2, ulsch_errors 2, ulsch_DTX 10, BLER 0.27100 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 0  SNR 50.5 dB
UE affe: MAC:    TX              0 RX              0 bytes
UE affe: LCID 1: TX              0 RX              0 bytes

[NR_MAC]   Frame.Slot 0.0
UE RNTI affe CU-UE-ID 1 out-of-sync PH 0 dB PCMAX 0 dBm, average RSRP 0 (0 meas)
UE affe: dlsch_rounds 1/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE affe: ulsch_rounds 3/3/2/2, ulsch_errors 2, ulsch_DTX 10, BLER 0.27100 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 0  SNR 50.5 dB
UE affe: MAC:    TX              0 RX              0 bytes
UE affe: LCID 1: TX              0 RX              0 bytes

[NR_MAC]   Frame.Slot 128.0
UE RNTI affe CU-UE-ID 1 out-of-sync PH 0 dB PCMAX 0 dBm, average RSRP 0 (0 meas)
UE affe: dlsch_rounds 1/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE affe: ulsch_rounds 3/3/2/2, ulsch_errors 2, ulsch_DTX 10, BLER 0.27100 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 0  SNR 50.5 dB
UE affe: MAC:    TX              0 RX              0 bytes
UE affe: LCID 1: TX              0 RX              0 bytes

[NR_MAC]   Frame.Slot 256.0
UE RNTI affe CU-UE-ID 1 out-of-sync PH 0 dB PCMAX 0 dBm, average RSRP 0 (0 meas)
UE affe: dlsch_rounds 1/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE affe: ulsch_rounds 3/3/2/2, ulsch_errors 2, ulsch_DTX 10, BLER 0.27100 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 0  SNR 50.5 dB
UE affe: MAC:    TX              0 RX              0 bytes
UE affe: LCID 1: TX              0 RX              0 bytes

[NR_MAC]   Frame.Slot 384.0
UE RNTI affe CU-UE-ID 1 out-of-sync PH 0 dB PCMAX 0 dBm, average RSRP 0 (0 meas)
UE affe: dlsch_rounds 1/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE affe: ulsch_rounds 3/3/2/2, ulsch_errors 2, ulsch_DTX 10, BLER 0.27100 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 0  SNR 50.5 dB
UE affe: MAC:    TX              0 RX              0 bytes
UE affe: LCID 1: TX              0 RX              0 bytes

[NR_MAC]   Frame.Slot 512.0
UE RNTI affe CU-UE-ID 1 out-of-sync PH 0 dB PCMAX 0 dBm, average RSRP 0 (0 meas)
UE affe: dlsch_rounds 1/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE affe: ulsch_rounds 3/3/2/2, ulsch_errors 2, ulsch_DTX 10, BLER 0.27100 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 0  SNR 50.5 dB
UE affe: MAC:    TX              0 RX              0 bytes
UE affe: LCID 1: TX              0 RX              0 bytes

[NR_MAC]   Frame.Slot 640.0
UE RNTI affe CU-UE-ID 1 out-of-sync PH 0 dB PCMAX 0 dBm, average RSRP 0 (0 meas)
UE affe: dlsch_rounds 1/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE affe: ulsch_rounds 3/3/2/2, ulsch_errors 2, ulsch_DTX 10, BLER 0.27100 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 0  SNR 50.5 dB
UE affe: MAC:    TX              0 RX              0 bytes
UE affe: LCID 1: TX              0 RX              0 bytes

[NR_MAC]   Frame.Slot 768.0
UE RNTI affe CU-UE-ID 1 out-of-sync PH 0 dB PCMAX 0 dBm, average RSRP 0 (0 meas)
UE affe: dlsch_rounds 1/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE affe: ulsch_rounds 3/3/2/2, ulsch_errors 2, ulsch_DTX 10, BLER 0.27100 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 0  SNR 50.5 dB
UE affe: MAC:    TX              0 RX              0 bytes
UE affe: LCID 1: TX              0 RX              0 bytes

[NR_MAC]   Frame.Slot 896.0
UE RNTI affe CU-UE-ID 1 out-of-sync PH 0 dB PCMAX 0 dBm, average RSRP 0 (0 meas)
UE affe: dlsch_rounds 1/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE affe: ulsch_rounds 3/3/2/2, ulsch_errors 2, ulsch_DTX 10, BLER 0.27100 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 0  SNR 50.5 dB
UE affe: MAC:    TX              0 RX              0 bytes
UE affe: LCID 1: TX              0 RX              0 bytes

[NR_MAC]   Frame.Slot 0.0
UE RNTI affe CU-UE-ID 1 out-of-sync PH 0 dB PCMAX 0 dBm, average RSRP 0 (0 meas)
UE affe: dlsch_rounds 1/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE affe: ulsch_rounds 3/3/2/2, ulsch_errors 2, ulsch_DTX 10, BLER 0.27100 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 0  SNR 50.5 dB
UE affe: MAC:    TX              0 RX              0 bytes
UE affe: LCID 1: TX              0 RX              0 bytes

[NR_MAC]   Frame.Slot 128.0
UE RNTI affe CU-UE-ID 1 out-of-sync PH 0 dB PCMAX 0 dBm, average RSRP 0 (0 meas)
UE affe: dlsch_rounds 1/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE affe: ulsch_rounds 3/3/2/2, ulsch_errors 2, ulsch_DTX 10, BLER 0.27100 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 0  SNR 50.5 dB
UE affe: MAC:    TX              0 RX              0 bytes
UE affe: LCID 1: TX              0 RX              0 bytes

[NR_MAC]   Frame.Slot 256.0
UE RNTI affe CU-UE-ID 1 out-of-sync PH 0 dB PCMAX 0 dBm, average RSRP 0 (0 meas)
UE affe: dlsch_rounds 1/0/0/0, dlsch_errors 0, pucch0_DTX 0, BLER 0.10000 MCS (0) 0
UE affe: ulsch_rounds 3/3/2/2, ulsch_errors 2, ulsch_DTX 10, BLER 0.27100 MCS (0) 0 (Qm 2 deltaMCS 0 dB) NPRB 0  SNR 50.5 dB
UE affe: MAC:    TX              0 RX              0 bytes
UE affe: LCID 1: TX              0 RX              0 bytes

[MAC]   UE affe: request release after UL failure timer expiry
[RRC]   no AMF for CU UE ID 1: auto-generate release command
[RLC]   Remove UE 45054
[NR_MAC]   Remove NR rnti 0xaffe
[NR_MAC]   Frame.Slot 384.0

[NR_MAC]   Frame.Slot 512.0

[NR_MAC]   Frame.Slot 640.0

[NR_MAC]   Frame.Slot 768.0

[NR_MAC]   Frame.Slot 896.0

[NR_MAC]   Frame.Slot 0.0

[NR_MAC]   Frame.Slot 128.0

[NR_MAC]   Frame.Slot 256.0

[NR_MAC]   Frame.Slot 384.0

[NR_MAC]   Frame.Slot 512.0

[NR_MAC]   Frame.Slot 640.0

[NR_MAC]   Frame.Slot 768.0

[NR_MAC]   Frame.Slot 896.0

[NR_MAC]   Frame.Slot 0.0

[NR_MAC]   Frame.Slot 128.0

[NR_MAC]   Frame.Slot 256.0

[NR_MAC]   Frame.Slot 384.0

[NR_MAC]   Frame.Slot 512.0

[NR_MAC]   Frame.Slot 640.0

[NR_MAC]   Frame.Slot 768.0

[NR_MAC]   Frame.Slot 896.0

[NR_MAC]   Frame.Slot 0.0

[NR_MAC]   Frame.Slot 128.0

[NR_MAC]   Frame.Slot 256.0

[NR_MAC]   Frame.Slot 384.0

[NR_MAC]   Frame.Slot 512.0

[NR_MAC]   Frame.Slot 640.0

[NR_MAC]   Frame.Slot 768.0

[NR_MAC]   Frame.Slot 896.0

[NR_MAC]   Frame.Slot 0.0

[NR_MAC]   Frame.Slot 128.0

[NR_MAC]   Frame.Slot 256.0

[NR_MAC]   Frame.Slot 384.0

[NR_MAC]   Frame.Slot 512.0

[NR_MAC]   Frame.Slot 640.0

[NR_MAC]   Frame.Slot 768.0

[NR_MAC]   Frame.Slot 896.0

[NR_MAC]   Frame.Slot 0.0

[NR_MAC]   Frame.Slot 128.0

[NR_MAC]   Frame.Slot 256.0

[HW]   A client connects, sending the current time
[HW]   Not supported to send Tx out of order 8368356224, 8368356223
[NR_MAC]   Frame.Slot 384.0

[NR_PHY]   [RAPROC] 499.19 Initiating RA procedure with preamble 34, energy 42.9 dB (I0 0, thres 120), delay 0 start symbol 8 freq index 0
[NR_MAC]   499.19 UE RA-RNTI 0113 TC-RNTI e08a: initiating RA procedure
[NR_MAC]   UE e08a: Msg3 scheduled at 500.17 (500.7 TDA 3) start 0 RBs 8
[NR_MAC]   UE e08a: 500.7 Generating RA-Msg2 DCI, RA RNTI 0x113, state 1, preamble_index(RAPID) 34, timing_offset = 0 (estimated distance 0.0 [m])
[NR_MAC]   500.7 Send RAR to RA-RNTI 0113
[NR_MAC]    500.17 PUSCH with TC_RNTI 0xe08a received correctly
[MAC]   [RAPROC] Received SDU for CCCH length 6 for UE e08a
[RLC]   Activated srb0 for UE 57482
[RLC]   Added srb 1 to UE 57482
[NR_MAC]   Activating scheduling Msg4 for TC_RNTI 0xe08a (state WAIT_Msg3)
[NR_RRC]   Decoding CCCH: RNTI e08a, payload_size 6
[NR_RRC]   [--] (cellID 0, UE ID 2 RNTI e08a) Create UE context: CU UE ID 2 DU UE ID 57482 (rnti: e08a, random ue id dcbb7527a2000000)
[RRC]   activate SRB 1 of UE 2
[NR_RRC]   [DL] (cellID bc614e, UE ID 2 RNTI e08a) Send RRC Setup
[NR_MAC]   UE e08a Generate Msg4: feedback at  501.17, payload 225 bytes, next state nrRA_WAIT_Msg4_MsgB_ACK
[HW]   Lost socket
[NR_MAC]   (UE e08a) Received Nack in Msg4, preparing retransmission!
[NR_MAC]   UE e08a Generate Msg4: feedback at  502. 7, payload 225 bytes, next state nrRA_WAIT_Msg4_MsgB_ACK
[NR_MAC]   (UE e08a) Received Nack in Msg4, preparing retransmission!
[NR_MAC]   UE e08a Generate Msg4: feedback at  502.17, payload 225 bytes, next state nrRA_WAIT_Msg4_MsgB_ACK
[NR_MAC]   (UE e08a) Received Nack in Msg4, preparing retransmission!
[NR_MAC]   UE e08a Generate Msg4: feedback at  503. 7, payload 225 bytes, next state nrRA_WAIT_Msg4_MsgB_ACK
[NR_MAC]    503. 7 UE e08a: RA Procedure failed at Msg4!
[NR_MAC]   (508.8) RA Contention Resolution timer expired for UE 0xe08a, RA procedure failed...
[RRC]   no AMF for CU UE ID 2: auto-generate release command
[RLC]   Remove UE 57482
[NR_MAC]   Remove NR rnti 0xe08a
[NR_MAC]   Frame.Slot 512.0

[NR_MAC]   Frame.Slot 640.0

[NR_MAC]   Frame.Slot 768.0

[NR_MAC]   Frame.Slot 896.0

^C
** Caught SIGTERM, shutting down
Returned from ITTI signal handler
[GNB_APP]   stopping nr-softmodem
[PHY]   Killing gNB 0 processing threads
^C
** Caught SIGTERM, shutting down
[PHY]   Stopping RU 0 processing threads
[PHY]   RU 0 RF device stopped
[GNB_APP]   turned off RU rfdevice
Bye.
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets/ran_build/build$
```

### nr-ue
```bash
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets/ran_build/build$ sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --rfsim --sa --nokrnmod 1
CMDLINE: "./nr-uesoftmodem" "-r" "106" "--numerology" "1" "--band" "78" "-C" "3619200000" "--rfsim" "--sa" "--nokrnmod" "1"
[UTIL]   running in SA mode (no --phy-test, --do-ra, --nsa option present)
[UTIL]   threadCreate() for Tpool0_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool1_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool2_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool3_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool4_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool5_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool6_-1: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for Tpool7_-1: creating thread with affinity ffffffff, priority 97
[OPT]   OPT disabled
[HW]   Version: Branch: develop Abrev. Hash: d71e333329 Date: Tue Jul 8 15:05:31 2025 +0000
[NR_RRC]   create TASK_RRC_NRUE
[UTIL]   threadCreate() for TASK_RRC_NRUE: creating thread with affinity ffffffff, priority 50
[UTIL]   threadCreate() for TASK_NAS_NRUE: creating thread with affinity ffffffff, priority 50
[SIM]   UICC simulation: IMSI=2089900007487, IMEISV=6754567890123413, Ki=fec86ba6eb707ed08905757b1bb44b8f, OPc=c42449363bbad02b66d16bc975d77cc1, DNN=oai, SST=0x01, SD=0xffffff
[NR_MAC]   [UE0] Initializing MAC
[NR_MAC]   Initializing dl and ul config_request. num_slots = 20
[RLC]   Activated srb0 for UE 0
[UTIL]   threadCreate() for time source iq samples: creating thread with affinity ffffffff, priority 2
[UTIL]   time manager configuration: [time source: iq_samples] [mode: standalone] [server IP: 127.0.0.1} [server port: 7374] (server IP/port not used)
[PHY]   Set UE_fo_compensation 0, UE_scan_carrier 0, UE_no_timing_correction 0
, chest-freq 0, chest-time 0
[PHY]   Set UE nb_rx_antenna 1, nb_tx_antenna 1, threequarter_fs 0, ssb_start_subcarrier 516
[PHY]   SA init parameters. DL freq 3619200000 UL offset 0 SSB numerology 1 N_RB_DL 106
[PHY]   Init: N_RB_DL 106, first_carrier_offset 1412, nb_prefix_samples 144,nb_prefix_samples0 176, ofdm_symbol_size 2048
[PHY]   samples_per_subframe 61440/per second 61440000, wCP 57344
[UTIL]   threadCreate() for SYNC__actor: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for DL__actor: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for DL__actor: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for DL__actor: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for DL__actor: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for UL__actor: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for UL__actor: creating thread with affinity ffffffff, priority 97
[PHY]   Initializing UE vars for gNB TXant 1, UE RXant 1
[PHY]   prs_config configuration NOT found..!! Skipped configuring UE for the PRS reception
[PHY]   HW: Configuring card 0, sample_rate 61440000.000000, tx/rx num_channels 1/1, duplex_mode TDD
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_freq 3619200000 Hz, rx_freq 3619200000 Hz, tune_offset 0
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_gain 0, rx_gain 110
[PHY]   HW: Configuring card 1, sample_rate 61440000.000000, tx/rx num_channels 1/1, duplex_mode TDD
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_freq 3619200000 Hz, rx_freq 3619200000 Hz, tune_offset 0
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_gain 0, rx_gain 110
[PHY]   HW: Configuring card 2, sample_rate 61440000.000000, tx/rx num_channels 1/1, duplex_mode TDD
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_freq 3619200000 Hz, rx_freq 3619200000 Hz, tune_offset 0
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_gain 0, rx_gain 110
[PHY]   HW: Configuring card 3, sample_rate 61440000.000000, tx/rx num_channels 1/1, duplex_mode TDD
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_freq 3619200000 Hz, rx_freq 3619200000 Hz, tune_offset 0
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_gain 0, rx_gain 110
[PHY]   HW: Configuring card 4, sample_rate 61440000.000000, tx/rx num_channels 1/1, duplex_mode TDD
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_freq 3619200000 Hz, rx_freq 3619200000 Hz, tune_offset 0
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_gain 0, rx_gain 110
[PHY]   HW: Configuring card 5, sample_rate 61440000.000000, tx/rx num_channels 1/1, duplex_mode TDD
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_freq 3619200000 Hz, rx_freq 3619200000 Hz, tune_offset 0
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_gain 0, rx_gain 110
[PHY]   HW: Configuring card 6, sample_rate 61440000.000000, tx/rx num_channels 1/1, duplex_mode TDD
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_freq 3619200000 Hz, rx_freq 3619200000 Hz, tune_offset 0
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_gain 0, rx_gain 110
[PHY]   HW: Configuring card 7, sample_rate 61440000.000000, tx/rx num_channels 1/1, duplex_mode TDD
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_freq 3619200000 Hz, rx_freq 3619200000 Hz, tune_offset 0
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_gain 0, rx_gain 110
[PHY]   Intializing UE Threads for instance 0 ...
[UTIL]   threadCreate() for UEthread_0: creating thread with affinity ffffffff, priority 97
[UTIL]   threadCreate() for L1_UE_stats_0: creating thread with affinity ffffffff, priority 1
UE threads created by 163636
TYPE <CTRL-C> TO TERMINATE
[HW]   Running as client: will connect to a rfsimulator server side
Initializing random number generator, seed 36345311390660236
[HW]   [RRU] has loaded RFSIMULATOR device.
[HW]   Trying to connect to 127.0.0.1:4043
[HW]   Connection to 127.0.0.1:4043 established
[PHY]   SSB position provided
[NR_PHY]   Starting sync detection
[PHY]   [UE thread Synch] Running Initial Synch
[NR_PHY]   Starting cell search with center freq: 3619200000, bandwidth: 106. Scanning for 1 number of GSCN.
[NR_PHY]   Scanning GSCN: 0, with SSB offset: 516, SSB Freq: 0.000000
[PHY]   Initial sync: pbch decoded sucessfully, ssb index 0
[PHY]   pbch rx ok. rsrp:51 dB/RE, adjust_rxgain:-1 dB
[NR_PHY]   Cell Detected with GSCN: 0, SSB SC offset: 516, SSB Ref: 0.000000, PSS Corr peak: 99 dB, PSS Corr Average: 61
[PHY]   [UE0] In synch, rx_offset 416896 samples
[PHY]   [UE 0] Measured Carrier Frequency offset 6 Hz
[PHY]   Initial sync successful, PCI: 0
[PHY]   HW: Configuring channel 0 (rf_chain 0): setting tx_freq 3619200006 Hz, rx_freq 3619200006 Hz, tune_offset 0
[PHY]   Got synch: hw_slot_offset 27, carrier off 6 Hz, rxgain 0.000000 (DL 3619200006.000000 Hz, UL 3619200006.000000 Hz)
[PHY]   UE synchronized! decoded_frame_rx=410 UE->init_sync_frame=0 trashed_frames=86
[PHY]   Resynchronizing RX by 416896 samples
[HW]   received write reorder clear context
[NR_RRC]   SIB1 decoded
[NR_MAC]   TDD period index = 6, based on the sum of dl_UL_TransmissionPeriodicity from Pattern1 (5.000000 ms) and Pattern2 (0.000000 ms): Total = 5.000000 ms
[NR_MAC]   Set TDD configuration period to: 8 DL slots, 3 UL slots, 10 slots per period (NR_TDD_UL_DL_Pattern is 7 DL slots, 2 UL slots, 6 DL symbols, 4 UL symbols)
[NR_MAC]   Configured 1 TDD patterns (total slots: pattern1 = 10, pattern2 = 0)
[PHY]   N_TA_offset changed from 0 to 800
[MAC]   Initialization of 4-Step CBRA procedure
[NR_MAC]   PRACH scheduler: Selected RO Frame 499, Slot 19, Symbol 8, Fdm 0
[PHY]   PRACH [UE 0] in frame.slot 499.19, placing PRACH in position 2828, Msg1/MsgA-Preamble frequency start 0 (k1 0), preamble_offset 8, first_nonzero_root_idx 0, preambleIndex = 34
[PHY]   RAR-Msg2 decoded
[NR_MAC]   [UE 0][RAPROC][RA-RNTI 0113] Got BI RAR subPDU 5 ms
[NR_MAC]   [UE 0][RAPROC][RA-RNTI 0113] Got RAPID RAR subPDU
[NR_MAC]   [UE 0][RAPROC][500.7] Found RAR with the intended RAPID 34
[MAC]   received TA command 31
[NR_MAC]   [RAPROC][500.17] RA-Msg3 transmitted

Assertion (feedback_ti >= GET_DURATION_RX_TO_TX(&mac->ntn_ta, dlsch_pdu->SubcarrierSpacing)) failed!
In nr_ue_process_dci_dl_10() /home/vishalkumar_shaw/oai/openair2/LAYER2/NR_MAC_UE/nr_ue_procedures.c:950
PDSCH to HARQ feedback time (2) needs to be higher than DURATION_RX_TO_TX (3).

Exiting execution
/home/vishalkumar_shaw/oai/openair2/LAYER2/NR_MAC_UE/nr_ue_procedures.c:950 nr_ue_process_dci_dl_10() Exiting OAI softmodem: _Assert_Exit_
^C^CAborted
vishalkumar_shaw@IN-8KBKXD3:~/oai/cmake_targets/ran_build/build$  
```