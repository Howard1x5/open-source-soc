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

### **Security Onion System Requirements & Justification**
#### **Official Security Onion System Requirements:**
- **CPU:** Minimum of 4 CPU cores.
- **RAM:** Minimum of 8 GB, recommended 12 GB or more.
- **Storage:** Depends on log retention needs; more storage allows for longer retention.
- **Network Interface Cards (NICs):** Two NICs recommended—one for management, one for monitoring.

**Reference:** [Security Onion Docs](https://docs.securityonion.net/en/2.4/getting-started.html?utm_source=chatgpt.com)

#### **Decision Rationale:**
Given that this deployment is intended for testing and light use, the selected **m5.large** instance (2 vCPUs, 8 GB RAM) aligns with the minimum requirements. However, it's important to note that while this configuration is suitable for initial testing, future production environments or increased log ingestion may necessitate scaling up to instances with higher specifications.

---

### **VPC & Networking Setup (Public vs. Private Decision)**

#### **Summary of Public vs. Private VPC Consideration**
- **Considered both public and private deployment options.**
- **Public VPC chosen** to minimize costs and simplify access for testing purposes.
- A **private subnet with a NAT gateway** would add **$30–$50/month**, which is excessive for a test environment.
- Security risks of **public VPC & SSH exposure** were noted, leading to additional hardening measures.

#### **Security Considerations for SSH & Public Exposure**
- **Using SSH key authentication only (no passwords).**
- **Restricting SSH access to a specific IP** (not `0.0.0.0/0` to prevent constant scanning).
- **Fail2Ban considered** to block brute-force attempts.
- **Alternative solution for changing IPs:** If a home IP changes frequently, consider using a **dynamic DNS (DDNS) service** to maintain a fixed hostname.

---

### **Key Pair Security & Best Practices**
#### **Why Securing Your AWS Key Pair is Critical**
- **Never store `.pem` files in GitHub repositories** (even private ones), as attackers can use them to access and modify your AWS instances.
- If an AWS key is exposed publicly, **it can be used to hijack your EC2 instance**, change settings, or even mine cryptocurrency on your account.

#### **Where & How I Stored My Key Securely**
1. **Stored the key outside of my GitHub projects**:
   - `~/aws-keys/security-onion-key.pem` (Linux/macOS)
   - `C:\Users\YourName\aws-keys\security-onion-key.pem` (Windows)
2. **Restricted key file permissions**:
   ```bash
   chmod 400 ~/aws-keys/security-onion-key.pem
   ```
3. **Alternative secure storage options**:
   - **Use a password manager (Bitwarden, 1Password, LastPass)**
   - **Store it on an encrypted USB drive or a secure vault (e.g., VeraCrypt, BitLocker)**

 **Key pair security is set up correctly!**

---

### **EC2 Instance Launch Troubleshooting**
#### **Issue: Availability Zone Does Not Support m5.large**
- **Error:** "The zone US East 1E does not support this instance type."
- **Fix:**
  1. Created a **new subnet** in `us-east-1a`, called `security-onion-public-subnet-2`.
  2. Changed the **EC2 instance configuration to use the new subnet.**
  3. Ensured **auto-assign public IP was enabled**.

 **Issue resolved!**

#### **Issue: Security Group Conflict**
- **Error:** "The security group `security-onion-sg` already exists for VPC `security-onion-vpc`."
- **Cause:** A duplicate security group was created, causing a naming conflict.
- **Fix:**
  1. **Deleted the duplicate security group** from AWS VPC Security Groups.
  2. **Re-selected the correct security group (`security-onion-sg`)** in EC2 instance configuration.
  3. **Ensured subnet and VPC settings matched correctly.**

 **Issue resolved!**

---

### **Final EC2 Instance Configuration & Launch**
- **Instance Name:** `security-onion-instance`
- **Subnet:** `security-onion-public-subnet-2` (`us-east-1a`)
- **Security Group:** `security-onion-sg`
- **Storage:**
  - **Root Volume:** 8GB, gp3, `/dev/sda1`, Delete on Termination -
  - **Log Storage Volume:** 50GB, gp3, `/dev/sdb`, Delete on Termination 
- **Advanced Settings:** Left mostly default, no snapshot used.

🚀 **Instance successfully launched! Next step: SSH access and Security Onion installation.**

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
