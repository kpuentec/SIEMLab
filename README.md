﻿# Security Monitoring Lab with Wazuh SIEM and AWS CloudTrail Integration

---

## 1. Overview
This document outlines the deployment and configuration of a security monitoring lab using the open-source Wazuh SIEM platform. The purpose of this lab is to demonstrate the integration of host-based intrusion detection, file integrity monitoring, brute-force attack detection, and cloud-based log ingestion from AWS CloudTrail.

The lab environment includes a Wazuh Manager hosted on Ubuntu, a Wazuh Agent installed on a Windows machine, a Kali Linux attack box for adversarial testing, and AWS services configured for cloud log monitoring. The setup provides visibility into both on-premises and cloud infrastructure, allowing for comprehensive security event detection and response.

---

## 2. Lab Architecture
* Wazuh Manager	- Ubuntu Server VM:	Collects, analyzes, and stores logs from connected agents
* Wazuh Agent	- Windows Host:	Sends system logs and monitored events to the manager
* Kali Linux - Kali VM:	Used to simulate attacks for testing alert mechanisms
* AWS CloudTrail - AWS Account	Records API calls and delivers logs to S3 for monitoring


Ubuntu Server (Wazuh Manager): Bridged Adapter, internet-accessible

Windows Host (Wazuh Agent): Bridged Adapter, local network only

Kali Linux: Bridged Adapter, local network only

AWS: Internet-based, accessed via Ubuntu machine

The Ubuntu server also fetches CloudTrail logs from an AWS S3 bucket for cloud activity monitoring.

---

## 3. Prerequisites
VirtualBox installed on host system

Ubuntu Server 20.04+ installed in VirtualBox

Internet connectivity on the Ubuntu server

Windows host or VM with administrative access

Basic familiarity with Linux system administration

---

## 4. Installing the Wazuh Manager (Ubuntu)

Add the Wazuh GPG Key

    curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --dearmor -o /usr/share/keyrings/wazuh-archive-keyring.gpg

Download and Execute Wazuh Installation Script

    curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh
    sudo bash ./wazuh-install.sh -a -i


After installation, access the dashboard:

    https://<ubuntu-vm-ip>

Login credentials are displayed upon installation completion.

---

## 5. Installing the Wazuh Agent (Windows)

Download the Windows Wazuh Agent installer from the official Wazuh website.

---

## 6. Registering the Agent with the Manager

Generate Agent Key on Ubuntu

Run the following on the manager:

    sudo /var/ossec/bin/manage_agents

Select A to add a new agent.

Assign a name (e.g., WindowsHost).

Choose E to extract the authentication key.

Apply the Key in the Windows Agent

Open the Wazuh Agent Manager GUI on Windows.

Paste the key.

Set the manager’s IP address (Ubuntu server).

Save and restart the agent service.

Verify that the agent is registered in the Wazuh Dashboard under "Agents".

![(/assets/Screenshot 2025-07-24 120745.png)](https://github.com/kpuentec/SIEMLab/blob/main/assets/Screenshot%202025-07-24%20120745.png)

---

## 7. File Integrity Monitoring (Windows Agent)
   
Wazuh monitors file changes using the Syscheck module.

Edit the agent’s configuration file:

    C:\Program Files (x86)\ossec-agent\ossec.conf
    
Add the following within the <directories> block:

    <directories realtime="yes">C:\Users\abc\Test</directories>
* Change abc to your system username, and create a Test Directory

Restart the Wazuh Agent service via the GUI or Services panel.

Wazuh will now monitor the specified directory in real time. File creation, modification, or deletion will trigger alerts in the dashboard.

---

## 8. Simulated Attack Scenarios (Kali Linux)

A brute-force SSH attack was performed using Hydra:

hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://192.168.x.x
Failed login attempts were detected by Wazuh, and alerts were generated for:



---

## 9. AWS CloudTrail Integration

Created a new S3 bucket named wazuh-cloudtrail-logs.

Enabled CloudTrail in the AWS Console for all regions.

Configured CloudTrail to deliver logs to the S3 bucket.

Install AWS CLI

    sudo apt update
    sudo apt install awscli -y
    aws configure

Provided:

AWS Access Key ID

AWS Secret Access Key

Default region

Output format (json)

Checked if the integration script existed:

    ls /var/ossec/integrations/aws/aws-s3.py

Executed the integration pull:

    sudo /var/ossec/integrations/aws/aws-s3.py --bucket wazuh-cloudtrail-logs --trail_prefix AWSLogs/<account-id>/CloudTrail --only_logs --skip_on_error
    
*Replaced <account-id> with actual AWS account number.

After several AWS console operations (e.g., logging in, creating S3 buckets), CloudTrail logs were fetched and parsed by Wazuh.

Events like ConsoleLogin, CreateBucket, and DeleteUser appeared in the Cloud Events section of the Wazuh dashboard.

---

## 10. Conclusion
This lab demonstrates how Wazuh can be used to construct a hybrid security monitoring environment that spans on-premise systems and cloud services. It showcases key SIEM capabilities, including:

Centralized log analysis

Real-time file integrity monitoring

Intrusion detection via simulated attacks

Integration with AWS CloudTrail for visibility into cloud infrastructure

This environment enables effective security testing and event correlation across heterogeneous systems.
