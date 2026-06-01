# Lab Setup Documentation

## Overview
This document describes the setup and configuration of the homelab environment used in the SIEM Implementation and Log Analysis project.

---

## Physical Hardware

| Device | Specs | Role |
|--------|-------|------|
| HP EliteDesk | x86_64, 16GB RAM | Proxmox hypervisor host |
| Raspberry Pi 5 (Node 1) | 8GB RAM, 7 inch display | SOC display — Grafana dashboard |
| Raspberry Pi 5 (Node 2) | 8GB RAM | Victim endpoint — Splunk forwarder |
| TP-Link Switch | Unmanaged | Network distribution |
| GL.iNet Router | Travel router | Network gateway |

---

## Virtual Machines on Proxmox

| VM | OS | Role | Resources |
|----|-----|------|-----------|
| Splunk SIEM | Ubuntu 24.04 LTS | Splunk Enterprise server | 8GB RAM, 50GB disk |
| Kali Linux | Kali Linux | Attack simulation | 4GB RAM, 40GB disk |
| Windows Server | Windows Server 2022 | Windows endpoint monitoring | 4GB RAM, 50GB disk |

---

## Network Configuration

| Device | IP | Subnet |
|--------|-----|--------|
| GL.iNet Router | 192.168.8.1 | 192.168.8.0/24 |
| Proxmox Host | 192.168.8.x | 192.168.8.0/24 |
| Splunk VM | 192.168.8.x | 192.168.8.0/24 |
| Raspberry Pi 5 | 192.168.8.x | 192.168.8.0/24 |
| Windows Server | 192.168.8.x | 192.168.8.0/24 |

> Note: Exact IP addresses redacted for security

---

## Splunk Enterprise Setup

### Installation
1. Downloaded Splunk Enterprise from splunk.com
2. Installed on Ubuntu 24.04 VM using .deb package
3. Configured to receive logs on port 9997
4. Accessed via web UI on port 8000

### Receiving Configuration
```bash
sudo /opt/splunk/bin/splunk enable listen 9997 -auth admin:password
sudo /opt/splunk/bin/splunk restart
```

---

## Splunk Universal Forwarder Setup

### Linux (Raspberry Pi 5)
```bash
# Install forwarder
sudo dpkg -i splunkforwarder.deb

# Configure inputs
sudo nano /opt/splunkforwarder/etc/system/local/inputs.conf
```

inputs.conf:
```
[monitor:///var/log/auth.log]
index = main
sourcetype = syslog

[monitor:///var/log/syslog*]
index = main
sourcetype = syslog

[monitor:///var/log/kern.log]
index = main
sourcetype = syslog
```

```bash
# Add Splunk server
sudo /opt/splunkforwarder/bin/splunk add forward-server SPLUNK-IP:9997

# Start forwarder
sudo /opt/splunkforwarder/bin/splunk start
sudo /opt/splunkforwarder/bin/splunk enable boot-start
```

### Windows Server 2022
1. Downloaded Splunk Universal Forwarder MSI from splunk.com
2. Installed with receiving indexer set to SPLUNK-IP:9997
3. Configured inputs.conf for Windows Event Logs

inputs.conf location:

C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf

inputs.conf content:
[WinEventLog://Security]
index = main
disabled = 0
[WinEventLog://System]
index = main
disabled = 0
[WinEventLog://Application]
index = main
disabled = 0

---

## UFW Firewall Configuration (Raspberry Pi 5)
```bash
sudo apt install ufw -y
sudo ufw enable
sudo ufw allow ssh
sudo ufw logging medium
sudo systemctl restart ufw
```

---

## Grafana Dashboard Setup

### Installation (Raspberry Pi 5 Node 1)
```bash
wget -q -O /usr/share/keyrings/grafana.key https://apt.grafana.com/gpg.key
echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
sudo apt update
sudo apt install grafana -y
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

### Kiosk Mode Auto Launch
