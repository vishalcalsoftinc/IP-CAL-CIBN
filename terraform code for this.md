### **Deploying a 5G Testbed on GCP with Terraform: The Definitive Guide**

This guide is structured in two main phases:
1.  **Infrastructure Provisioning:** Using Terraform to automatically create all the necessary GCP resources (VPCs, VMs, NAT, Firewall).
2.  **Software Installation:** Manually installing and configuring Open5GS and OAI on the provisioned VMs.

### **Part 1: Setting Up Your Local Environment for Terraform**

Before you can run the Terraform code, you need to configure your local machine to securely communicate with your GCP account.

#### **Step 1: Install the Terraform CLI**
Terraform is a single command-line tool. Download the appropriate package for your operating system from the [official Terraform website](https://www.terraform.io/downloads.html) and add it to your system's `PATH`.

#### **Step 2: Install the Google Cloud CLI (`gcloud`)**
The `gcloud` CLI is the easiest way to handle authentication with GCP. Follow the instructions on the [Google Cloud SDK installation page](https://cloud.google.com/sdk/docs/install).

#### **Step 3: Authenticate and Configure Your Project**
1.  **Login to your GCP account:** This command will open a browser window for you to log in and grant permissions to the CLI.
    ```bash
    gcloud auth application-default login
    ```
2.  **Set your project:** Tell `gcloud` which project you want to work on. Replace `YOUR_PROJECT_ID` with the ID of the GCP project you created.
    ```bash
    gcloud config set project YOUR_PROJECT_ID
    ```
3.  **Enable Required APIs:** Terraform needs certain GCP APIs to be enabled to create resources. Run this command to enable them for your project:
    ```bash
    gcloud services enable compute.googleapis.com \
                            cloudnat.googleapis.com \
                            servicenetworking.googleapis.com
    ```

Your local environment is now ready to provision resources on GCP.

### **Part 2: The Terraform Configuration**

Create a new directory on your local machine called `gcp-5g-testbed`. Inside this directory, create the following four files.

#### **`provider.tf`**
This file tells Terraform you'll be using the Google Cloud provider.

```terraform
# provider.tf
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = ">= 4.0"
    }
  }
}

provider "google" {
  project = var.gcp_project_id
  region  = var.gcp_region
}
```

#### **`variables.tf`**
This file defines the input variables for our infrastructure, making it easy to change things like the project ID or region later.

```terraform
# variables.tf
variable "gcp_project_id" {
  description = "The GCP Project ID to deploy resources into."
  type        = string
}

variable "gcp_region" {
  description = "The GCP region to deploy resources into."
  type        = string
  default     = "us-central1"
}

variable "vm_machine_type" {
  description = "The machine type for the OAI and Open5GS VMs."
  type        = string
  default     = "e2-small"
}

variable "vms" {
  description = "A map of VM names to their static IP address suffixes."
  type        = map(string)
  default = {
    "open5gs-vm"  = "95"
    "oai-cucp-vm" = "93"
    "oai-cuup-vm" = "94"
    "oai-du-vm"   = "92"
    "oai-nr-ue-vm" = "91"
  }
}
```

#### **`main.tf`**
This is the core file that defines all the GCP resources to be created.

```terraform
# main.tf

# 1. VPC Network and Subnet
resource "google_compute_network" "vpc" {
  name                    = "oai-5g-vpc"
  auto_create_subnetworks = false
}

resource "google_compute_subnetwork" "subnet" {
  name          = "oai-subnet"
  ip_cidr_range = "172.17.0.0/24"
  network       = google_compute_network.vpc.id
  region        = var.gcp_region
}

# 2. Firewall Rules
resource "google_compute_firewall" "allow_ssh" {
  name    = "allow-ssh"
  network = google_compute_network.vpc.name
  allow {
    protocol = "tcp"
    ports    = ["22"]
  }
  source_ranges = ["0.0.0.0/0"]
}

resource "google_compute_firewall" "allow_internal" {
  name    = "allow-internal-all"
  network = google_compute_network.vpc.name
  allow {
    protocol = "all"
  }
  source_ranges = [google_compute_subnetwork.subnet.ip_cidr_range]
}

# 3. Cloud NAT for Internet Access
resource "google_compute_router" "router" {
  name    = "oai-5g-router"
  network = google_compute_network.vpc.id
  region  = var.gcp_region
}

resource "google_compute_router_nat" "nat" {
  name                               = "oai-5g-nat"
  router                             = google_compute_router.router.name
  region                             = google_compute_router.router.region
  source_subnetwork_ip_ranges_to_nat = "ALL_SUBNETWORKS_ALL_IP_RANGES"
  log_config {
    enable = true
    filter = "ERRORS_ONLY"
  }
}

# 4. Virtual Machines
resource "google_compute_instance" "vm_instances" {
  for_each = var.vms

  name         = each.key
  machine_type = var.vm_machine_type
  zone         = "${var.gcp_region}-a"

  boot_disk {
    initialize_params {
      image = "ubuntu-os-cloud/ubuntu-2004-lts"
      size  = 30
    }
  }

  network_interface {
    subnetwork = google_compute_subnetwork.subnet.id
    # Assign the static internal IP based on the map value
    network_ip = "172.17.0.${each.value}"
  }

  // Ensures NAT is ready before VMs try to access the internet for package installation
  depends_on = [google_compute_router_nat.nat]
}
```

#### **`outputs.tf`**
This file will print useful information after the deployment is complete.

```terraform
# outputs.tf
output "vm_details" {
  description = "Details of the created VMs"
  value = {
    for vm in google_compute_instance.vm_instances :
    vm.name => {
      private_ip = vm.network_interface.0.network_ip
      gcp_zone   = vm.zone
    }
  }
}
```

### **Part 3: Deploying the Infrastructure**

Now, open your terminal, navigate into the `gcp-5g-testbed` directory, and run the following commands.

1.  **Initialize Terraform:** This downloads the Google provider plugin.
    ```bash
    terraform init
    ```
2.  **Plan the Deployment:** This command shows you a "dry run" of all the resources Terraform will create. It's a critical safety check. You will need to provide your GCP project ID here.
    ```bash
    terraform plan -var="gcp_project_id=YOUR_PROJECT_ID"
    ```
    Review the output. It should tell you it will add ~10 resources (5 VMs, 1 VPC, 1 subnet, 2 firewall rules, 1 router, 1 NAT).

3.  **Apply the Configuration:** This command will build the infrastructure.
    ```bash
    terraform apply -var="gcp_project_id=YOUR_PROJECT_ID"
    ```
    Terraform will show you the plan again and ask for confirmation. Type `yes` and press Enter. The process will take a few minutes. When it's done, it will print the `vm_details` output.

**Your GCP infrastructure is now ready!** You have five VMs running in a private VPC, all with the correct static internal IPs and the ability to reach the internet.

### **Part 4: Software Installation (Manual Steps)**

Your infrastructure is automated, but the software inside the VMs still needs to be installed. Follow the same software installation steps from the previous guide.

1.  **SSH into each VM** using the GCP console or the `gcloud compute ssh` command.
2.  Follow the instructions in **Part 3: Component Installation & Configuration** and **Part 4: Starting and Testing the Network** from the previous response to:
    *   Install and configure **Open5GS** and the **WebUI** on `open5gs-vm`.
    *   Build the **OAI RAN** binaries on all four OAI VMs.
    *   Create and edit the `.conf` files on each OAI VM, using the static IPs (`172.17.0.x`) that Terraform has already configured.
    *   Start the components in the correct order.
    *   Test connectivity with `ping`.

### **Part 5: Cleaning Up with Terraform (CRITICAL STEP)**

When you are finished with your testbed, you can destroy **all** the created resources with a single command. This is a major advantage of using Terraform.

From inside your `gcp-5g-testbed` directory, run:
```bash
terraform destroy -var="gcp_project_id=YOUR_PROJECT_ID"
```
