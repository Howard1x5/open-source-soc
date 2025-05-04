# AWS Deployment Log - Security Onion

## Environment Setup

### Initial AWS EC2 Instance Selection

**Objective:** Choose an appropriate EC2 instance for Security Onion.

**Options Considered:**
- m5.large (2 vCPUs, 8GB RAM) → Minimum requirement
- m5.xlarge (4 vCPUs, 16GB RAM) → Recommended for heavier workloads

**Expected Log Volume:** Light testing use, no heavy log ingestion expected

**Final Decision:** Selected t3a.xlarge for initial deployment

---

### VPC and Networking Setup

**Public VPC chosen** for simplicity and cost savings.

Security concerns mitigated by:
- SSH access limited to home IP
- Using SSH keys (no passwords)
- Elastic IP assigned for management

---

### Key Pair Setup

Key file saved at `~/aws-keys/sec-onion.pem`

Permissions set:

```bash
chmod 400 ~/aws-keys/sec-onion.pem

Key pair security confirmed.
EC2 Instance Launch and Security Group

Launched instance with:

    Instance Type: t3a.xlarge

    Storage: 256GB gp3

    Security Group Rules:

        SSH (22) from home IP

        HTTPS (443) from home IP

        Syslog UDP (514) from anywhere (for log ingestion)

Elastic IP associated.

Security group confirmed.
Deployment Issues and Fixes

    SSH connection initially failed with old key → Created new key pair

    SSH connection denied (username) → Correct username is onion

    Wizard exited due to missing second NIC → Added sniffing NIC to instance

SSH now connects successfully using:

ssh -i ~/aws-keys/sec-onion2.pem onion@<elastic-ip>

Re-entered setup wizard after adding NIC.
Current Status

Reached Security Onion setup wizard.

Selected:

    Install Type: Standalone

    Admin Email: [my_email@example.com]

    IP Mode: Manual

Waiting for installation packages to download (encountering internet connectivity issues at this step)
Next Steps

    Verify internet connectivity inside instance

    Check route tables, IGW, and outbound rules on instance subnet

    Retry installation after resolving connectivity