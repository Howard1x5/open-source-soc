# AWS Deployment Log - Security Onion

## **Date & Session Log**
### **Initial AWS EC2 Instance Selection**

#### **Summary of Decision-Making Process**
- **Objective:** Choose an appropriate EC2 instance for Security Onion.
- **Options Considered:**
  - **m5.large** (2 vCPUs, 8GB RAM) → Minimum requirement.
  - **m5.xlarge** (4 vCPUs, 16GB RAM) → Recommended for heavier workloads.
- **Expected Log Volume:** Light testing use, no heavy log ingestion expected.
- **Final Decision:** Selected `m5.large` for initial deployment.

#### **Step-by-Step Process & Observations**
1. **Logged into AWS Console.**
2. **Navigated to EC2 Dashboard.**
3. **Reviewed instance types:** Compared CPU, RAM, and expected log volume needs.
4. **Chose `m5.large` instance** based on testing/light use case.
5. **Confirmed decision** based on AWS recommendations and best practices.

---

### **Syslog Port 514 Opened for Ingestion**

#### **Summary of Decision-Making Process**
- **Objective:** Enable Security Onion to receive syslog data from internal VMs.
- **CIDR Range Used:** `10.0.0.0/16` — aligned with the active VPC route table.
- **Port/Protocol:** UDP 514
- **Rationale:**
  - Restrict access to internal sources only.
  - Avoid global exposure (`0.0.0.0/0`).
  - Allow future VM log forwarding within VPC.

#### **Step-by-Step Process & Observations**
1. **Opened EC2 instance’s Security Group.**
2. **Edited Inbound Rules:**
   - Type: Custom UDP
   - Protocol: UDP
   - Port Range: 514
   - Source: `10.0.0.0/16`
   - Description: Syslog ingestion from internal VMs
3. **Saved rule changes** without needing to start the EC2 instance.

#### **Result**
- **Port 514 (UDP)** is now open for internal log forwarding.
- EC2 instance is ready to receive syslog from internal VM sources.

---

### **IAM Role Creation and Attachment**

#### **Summary of Decision-Making Process**
- **Objective:** Provide Security Onion EC2 instance with permission to interact with AWS services.
- **Use Case:** Pull logs from AWS services like CloudTrail and S3 during testing.

#### **Step-by-Step Process & Observations**
1. **Opened IAM Console.**
2. **Created new IAM role with the following settings:**
   - Trusted entity: AWS Service → EC2
   - Attached policies:
     - `AmazonS3ReadOnlyAccess`
     - `AWSCloudTrailReadOnlyAccess`
     - `AmazonEC2ReadOnlyAccess`
   - Role name: `SecurityOnion-EC2-Role`
   - Tags added: `Project=open-source-soc`, `Environment=testing`
3. **Navigated to EC2 Console → Instances.**
4. **Selected the Security Onion instance.**
5. **Attached IAM role via:** Actions → Security → Modify IAM Role.

#### **Result**
- **IAM role successfully attached** to the Security Onion EC2 instance.
- Instance now has **read-only access** to AWS services required for logging and diagnostics.

---

### **EC2 Instance Provisioning and EBS Volume Attachment**

#### **Summary of Decision-Making Process**
- **Objective:** Finalize instance provisioning and attach an additional volume for log storage.
- **Rationale:**
  - Root volume alone is insufficient for long-term log retention.
  - EBS cost (~$4/month for 50GB) is acceptable for learning/testing purposes.

#### **Step-by-Step Process & Observations**
1. **Verified EC2 instance provisioning was completed previously.**
2. **Confirmed OS selection:** Ubuntu 20.04 LTS.
3. **Started the EC2 instance (if stopped).**
4. **SSH'd into instance using private key.**
5. **Confirmed EBS volumes already exist:**
   - Root volume: 8GB
   - Log volume: 50GB, gp3, created on March 15, 2025
6. **Attached 50GB volume to EC2 instance.**
7. **SSH'd into instance and confirmed the volume appeared as `/dev/nvme1n1`** via `lsblk`.
8. **Formatted the new volume using ext4:**
   ```bash
   sudo mkfs -t ext4 /dev/nvme1n1
   ```
9. **Created a mount directory for Security Onion logs:**
   ```bash
   sudo mkdir -p /var/log/securityonion
   ```
10. **Mounted the volume to the log directory:**
    ```bash
    sudo mount /dev/nvme1n1 /var/log/securityonion
    ```
11. **Added entry to `/etc/fstab` to make the mount persistent across reboots:**
    ```bash
    /dev/nvme1n1 /var/log/securityonion ext4 defaults,nofail 0 2
    ```
12. **Verified mount persistence:**
    ```bash
    sudo umount /var/log/securityonion
    sudo mount -a
    df -h
    ```

#### **Result**
- EC2 instance successfully provisioned.
- Additional 50GB EBS volume (created earlier) is now correctly formatted, mounted, and persistently attached to `/var/log/securityonion` using device path `/dev/nvme1n1`.

---

### **Elastic IP Allocation Decision**

#### **Summary of Decision-Making Process**
- **Objective:** Decide whether to use an Elastic IP for consistent external access.
- **Consideration:** Security Onion includes a web interface (SOC) accessed via port 443.
- **Issue:** Without Elastic IP, public IP changes on every reboot.

#### **Decision Rationale**
- Persistent access to Security Onion’s web interface (SOC) is important.
- Avoids manually updating firewall, bookmarks, or scripts every session.
- Elastic IP cost (~$3.60/month) is low and justified by the convenience.

#### **Outcome**
- Chose to allocate and associate an Elastic IP to the Security Onion EC2 instance.
- Allows seamless, consistent SSH and web access during setup and use.

---

### **Network Connectivity Troubleshooting (SSH Timeout Issue)**

#### **Symptoms:**
- SSH timeout despite Elastic IP, correct key file, and security group rules

#### **Root Cause:**
- EC2 instance was launched in a subnet associated with the **main route table**, which lacked a route to the **Internet Gateway (IGW)**
- Subnet had **auto-assign public IP disabled**, and was not initially associated with the route table that had outbound internet access (`security-onion-rt`)

#### **Resolution Steps:**
1. Identified that the instance's subnet was `security-onion-public-subnet-2` (`subnet-08a56e8c...`)
2. Verified this subnet was incorrectly using `rtb-0272d50d...`, which had no `0.0.0.0/0` → IGW route
3. Confirmed that the correct route table (`security-onion-rt`) did have a route to the internet
4. Edited **route table associations** to associate `subnet-08a56e8c...` with `security-onion-rt`
5. Waited ~15 seconds for propagation
6. SSH access immediately worked via:
   ```bash
   ssh -i ~/aws-keys/security-onion-key.pem ubuntu@<elastic-ip>
   ```

#### **Outcome:**
- Successfully connected to Security Onion Ubuntu EC2 instance
- Routing and network configuration confirmed functional

---

## **Next Steps**
- Begin Security Onion installation
- Document SOC web interface setup and port access (port 443)
- Verify data ingestion paths from other internal VMs

Security Onion ISO Download & Verification (Superseded by AMI-Based Workflow)

Summary of Decision-Making Process

Objective: Download and verify the official Security Onion 2.4.141 ISO image before installation.

Challenge: Default root volume (8GB) was too small for handling the 13GB ISO.

Solution: Repurposed 50GB EBS volume, mounted at /mnt/data, to house installation files in a clean, dedicated directory structure.

Step-by-Step Process & Observations

# Unmount previous log mount
sudo umount /var/log/securityonion

# Mount the EBS volume to a clean directory
sudo mkdir -p /mnt/data
sudo mount /dev/nvme1n1 /mnt/data

# Create installer directory
mkdir -p /mnt/data/installer
cd /mnt/data/installer

# Import GPG key
wget https://raw.githubusercontent.com/Security-Onion-Solutions/securityonion/2.4/main/KEYS -O - | gpg --import -

# Download ISO signature
wget https://github.com/Security-Onion-Solutions/securityonion/raw/2.4/main/sigs/securityonion-2.4.141-20250331.iso.sig

# Download ISO file
wget https://download.securityonion.net/file/securityonion/securityonion-2.4.141-20250331.iso

# Verify signature
gpg --verify securityonion-2.4.141-20250331.iso.sig securityonion-2.4.141-20250331.iso

Verification Result

Good signature from: "Security Onion Solutions, LLC <info@securityonionsolutions.com>"
Primary key fingerprint: C804 A93D 36BE 0C73 3EA1 9644 7C10 60B7 FE50 7013

Outcome

ISO image integrity and authenticity successfully verified.

Attempted to mount the ISO and run ./install, but the ISO is not designed for execution on a running EC2 instance.

Pivoted to correct method: Security Onion must be installed using the prebuilt AMI available for AWS deployments.

Attempted curl -sSfL https://securityonion.net/so | sudo bash but the URL returned a 404 error.

Decision made to completely decommission manually built AWS environment (instances, VPCs, subnets, EBS, Elastic IPs) in favor of a clean restart using the Security Onion AMI deployment method.

Next Steps (Updated for AMI Workflow)

Discard ISO-based install strategy.

Tear down manually created AWS resources to avoid configuration drift.

Begin fresh using the official Security Onion AMI as outlined here:
https://docs.securityonion.net/en/2.4/cloud-amazon.html

Deploy Security Onion using AMI:

Launch AMI from AWS Marketplace or public image list.

Assign appropriate security group and IAM role.

Complete setup using so-setup script on the AMI instance.

Validate access to SOC web interface.

Document new checklist steps in README for AMI-based deployment.