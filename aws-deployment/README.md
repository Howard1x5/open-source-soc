### Security Onion AMI-Based Deployment Checklist

#### Phase 1: AWS Cleanup (if starting fresh)
- [ ] Terminate existing EC2 instance
- [ ] Delete 50GB EBS volume
- [ ] Release Elastic IP (if allocated)
- [ ] Remove custom security group (if unused)
- [ ] Remove IAM role created for ISO setup
- [ ] Delete custom route tables, subnets, and VPC (if no longer needed)

---

#### Phase 2: Launch Official Security Onion AMI
- [ ] Review official AMI instructions: https://docs.securityonion.net/en/2.4/cloud-amazon.html
- [ ] Locate the AMI ID for your AWS region
- [ ] Launch EC2 instance using the AMI
  - Instance type: `m5.large` or larger
  - Storage: At least 50 GB
  - Enable public IP assignment
  - Use existing or new key pair for SSH
- [ ] Create or assign a security group:
  - [ ] TCP 22 (SSH) – your IP only
  - [ ] TCP 443 (SOC web interface)
  - [ ] UDP 514 (Syslog ingestion)
- [ ] Allocate and associate Elastic IP (optional but recommended)

---

#### Phase 3: Configure Security Onion Instance
- [ ] SSH into the new instance
- [ ] Run `sudo so-setup`
- [ ] Choose deployment mode: **Standalone**
- [ ] Assign network interfaces:
  - Management: default interface (e.g., `ens5`)
  - Monitoring: same (optional for testing)
- [ ] Enable Elastic Stack (Kibana, Elasticsearch, etc.)
- [ ] Set up credentials for SOC access
- [ ] Complete installation (may take 30–60 minutes)

---

#### Phase 4: Validation & Access
- [ ] Visit `https://<your-elastic-ip>` in browser
- [ ] Log in to the SOC web interface
- [ ] Verify services (Zeek, Suricata, Wazuh, etc.) are running
- [ ] Test forwarding logs from internal VMs to UDP 514
- [ ] Take a snapshot or AMI backup of the working install

