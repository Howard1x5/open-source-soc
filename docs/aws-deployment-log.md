# AWS Deployment Log - Security Onion (AMI-Based Deployment)

## Overview
Following Security Onion's official documentation for AWS deployment using prebuilt AMIs. This approach replaces the previous manual configuration and ISO-based installation.

Reference: [Security Onion AWS Deployment Guide](https://docs.securityonion.net/en/2.4/cloud-amazon.html)

---

## Date & Session Log

### Elastic IP Assignment
- **Allocated** a new Elastic IP for persistent public access.
- **Associated** the Elastic IP with the Security Onion EC2 instance.
- Ensures stable SSH and SOC web interface connectivity regardless of instance restarts.

### Initial Setup Decision
- **Objective:** Deploy Security Onion in AWS using the official AMI for streamlined setup and best practices.
- **Decision:** Abandoned manual VPC, EC2, and ISO installation in favor of Security Onion's supported AMI deployment method.
- **Cleanup Completed:** All previous AWS resources (EC2, VPCs, subnets, EBS volumes, Elastic IPs, IAM roles) from manual setup have been removed to ensure a clean environment.

---

## Storage Provisioning
- **Attempted:** Reduce gp3 volume size to 100GB to lower monthly costs.
- **Result:** Launch failed due to AMI snapshot requirement (`expect size >= 256GB`).
- **Final Decision:** Proceeding with default **256GB gp3** volume as required by Security Onion AMI.
- **Estimated Cost:** ~$20/month for storage.

---

## Next Steps
- Launch Security Onion AMI from AWS Marketplace or official AMI ID.
- Select instance type based on Security Onion's recommendation (`t3a.xlarge` or `t3a.2xlarge`).
- Configure security group:
  - Allow TCP 22 (SSH) from admin IP.
  - Allow TCP 443 for SOC web interface.
  - Allow UDP 514 for syslog ingestion.
- SSH into instance and run `sudo so-setup`.
- Complete setup and verify SOC accessibility.
- Document configuration and validation steps.
