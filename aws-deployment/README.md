# AWS Deployment - Security Onion

## Overview
This document provides a **step-by-step checklist** for deploying Security Onion on AWS. The goal is to logically walk through the AWS setup process, ensuring that all required configurations are documented and can be replicated. As decisions are made, they will be added to this checklist with checkboxes indicating completion.

---

## [1] Determine Required EC2 Instance Size
- [X] Research Security Onion system requirements
- [X] Choose an appropriate EC2 instance type
  - [X] Initial selection: `m5.large` (2 vCPUs, 8GB RAM) ✅
  - [X] Consider upgrade options: `m5.xlarge` (4 vCPUs, 16GB RAM) or higher if needed
  - [X] Document reasoning for instance choice

---

## [2] Set Up VPC & Networking
- [X] Create a new VPC or use an existing one
- [X] Configure subnets (public or private based on need)
- [ ] Configure Security Groups:
  - [X] Allow SSH (Port 22) from specific IPs
  - [X] Allow Web UI access (Port 443) for Security Onion
  - [-] Allow Syslog (Port 514) and other required ingestion ports
- [X] Set up appropriate routing and internet gateway if needed
- [X] Document networking choices and rationale

---

## [3] Create IAM Role for EC2 Instance
- [ ] Create a new IAM role with required permissions:
  - [ ] Attach `AmazonS3ReadOnlyAccess` (if storing logs in S3)
  - [ ] Attach `AWSCloudTrailReadOnlyAccess` (for CloudTrail logs)
  - [ ] Attach `AmazonEC2FullAccess` (for EC2 management, optional)
- [ ] Document IAM role configuration

---

## [4] Provision EC2 Instance
- [ ] Launch EC2 instance with Security Onion-compatible OS:
  - [ ] Select Ubuntu 20.04 or CentOS 7 (recommended for Security Onion)
  - [ ] Assign IAM role created above
  - [ ] Attach a static Elastic IP (optional for external access)
- [ ] Document EC2 provisioning process

---

## [5] Attach Additional Storage for Logs
- [ ] Decide on storage requirements for logs:
  - [ ] Use default root volume (if small-scale setup)
  - [ ] Attach additional **EBS Volume** (recommended for large log storage)
- [ ] Format and mount the storage:
  - [ ] Create filesystem (`mkfs -t ext4 /dev/xvdf`)
  - [ ] Mount to `/var/log/securityonion/`
  - [ ] Add to `/etc/fstab` for persistence
- [ ] Document storage choices and setup process

---

## [6] Security Onion Installation on AWS
- [ ] Download and install Security Onion:
  - [ ] Update system (`sudo apt update && sudo apt upgrade -y`)
  - [ ] Download Security Onion (`wget https://github.com/Security-Onion-Solutions/securityonion2/releases/latest/download/securityonion.iso`)
  - [ ] Install required dependencies (`sudo apt install -y docker.io python3`)
  - [ ] Run Security Onion setup (`sudo so-setup`)
- [ ] Configure Security Onion settings
- [ ] Document installation steps and configurations

---

## [7] Validate Deployment & Troubleshooting
- [ ] Verify Security Onion services are running (`sudo so-status`)
- [ ] Test log ingestion (CloudTrail, VPC Flow Logs, syslog sources)
- [ ] Check Security Onion Web UI for alerts
- [ ] Troubleshoot any failed components and document resolutions

---

## **Next Steps**
- Once AWS deployment is complete, move to **Security Onion configuration** in the `security-onion/` directory.
- Further documentation updates will be added as deployment progresses.

---

### **Additional Notes**
This document will be updated dynamically as new decisions are made. Every completed step will have a checkbox checked off to track progress. More detailed explanations will be included in the `docs/` directory for future reference.



