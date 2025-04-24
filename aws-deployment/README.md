# Security Onion AMI-Based Deployment Checklist

## Phase 1: AWS Cleanup (Completed)
- [x] Terminate existing EC2 instance from manual setup
- [x] Delete 50GB EBS volume
- [x] Release Elastic IP from previous deployment
- [x] Remove custom security group (if unused)
- [x] Remove IAM role created for ISO setup
- [x] Delete custom route tables, subnets, and VPC

---

## Phase 2: Launch Official Security Onion AMI
- [x] Locate and select Security Onion AMI from AWS Marketplace
- [x] Choose instance type (`t3a.xlarge` as per documentation)
- [x] Accept default **256GB gp3** storage requirement
- [x] Enable Auto-assign Public IP
- [x] Allocate and associate Elastic IP for persistent access
- [x] Configure security group:
  - [x] TCP 22 (SSH) from admin IP
  - [ ] TCP 443 (SOC web interface)
  - [ ] UDP 514 (Syslog ingestion)

---

## Phase 3: Configure Security Onion Instance
- [ ] Start EC2 instance
- [ ] SSH into instance using Elastic IP
- [ ] Run `sudo so-setup`
- [ ] Select deployment mode: **Standalone**
- [ ] Assign network interfaces
- [ ] Enable Elastic Stack
- [ ] Set SOC credentials
- [ ] Complete installation process

---

## Phase 4: Validation & Post-Setup
- [ ] Access SOC Web Interface via `https://<Elastic-IP>`
- [ ] Verify services (Zeek, Suricata, Wazuh, Elastic Stack)
- [ ] Test syslog forwarding from internal VM to UDP 514
- [ ] Take snapshot or create AMI backup after successful setup
- [ ] Finalize documentation of setup details
