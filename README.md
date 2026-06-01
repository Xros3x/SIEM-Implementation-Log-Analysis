# SIEM Implementation and Log Analysis

![Splunk](https://img.shields.io/badge/-Splunk-000000?&style=for-the-badge&logo=Splunk&logoColor=white)
![Linux](https://img.shields.io/badge/-Linux-FCC624?&style=for-the-badge&logo=linux&logoColor=black)
![Ubuntu](https://img.shields.io/badge/-Ubuntu-E95420?&style=for-the-badge&logo=ubuntu&logoColor=white)
![Kali](https://img.shields.io/badge/-Kali_Linux-557C94?&style=for-the-badge&logo=kali-linux&logoColor=white)
![Raspberry Pi](https://img.shields.io/badge/-Raspberry_Pi-A22846?&style=for-the-badge&logo=Raspberry-Pi&logoColor=white)
![Windows Server](https://img.shields.io/badge/-Windows_Server-0078D6?&style=for-the-badge&logo=windows&logoColor=white)

## Objective
Deploy and configure Splunk Enterprise SIEM in a homelab environment to ingest logs from multiple endpoints, build real time detection rules, and simulate real world attacks to practice threat detection, alert creation, and incident response documentation.

---

## Environment

| Component | Details |
|-----------|---------|
| SIEM | Splunk Enterprise on Ubuntu 24.04 VM |
| Hypervisor | Proxmox VE on HP EliteDesk |
| Victim Endpoint 1 | Raspberry Pi 5 — Raspberry Pi OS |
| Victim Endpoint 2 | Windows Server 2022 VM |
| Attack Machine | Kali Linux VM on Proxmox |
| Log Forwarder | Splunk Universal Forwarder on all endpoints |
| Network | Isolated homelab environment |

---

## Tools Used

| Tool | Purpose |
|------|---------|
| Splunk Enterprise | SIEM platform — log ingestion, search, alerting |
| Splunk Universal Forwarder | Real time log shipping from endpoints |
| Hydra | SSH brute force attack simulation |
| Nmap | Network reconnaissance simulation |
| Kali Linux | Attack platform |
| UFW | Linux firewall — generates network logs |

---

## Lab Architecture

```
Kali Linux VM (Attacker)
        |
        | attacks
        ↓
Raspberry Pi 5 (Victim - Linux)     Windows Server 2022 (Victim - Windows)
        |                                       |
        | auth.log + kern.log                   | Windows Event Logs
        | via Splunk Forwarder                  | via Splunk Forwarder
        ↓                                       ↓
        |_______________________________________|
                        |
                        ↓
              Splunk Enterprise SIEM
              (Ubuntu 24.04 VM on Proxmox)
                        |
                        ↓
              SOC Monitoring Dashboard
              (Grafana on Raspberry Pi 5)
```

---

## Log Sources Configured

| Source | Endpoint | Log Path | Sourcetype |
|--------|---------|----------|------------|
| SSH Authentication | Raspberry Pi 5 | /var/log/auth.log | syslog |
| Firewall/Network | Raspberry Pi 5 | /var/log/kern.log | syslog |
| System Logs | Raspberry Pi 5 | /var/log/syslog | syslog |
| Security Events | Windows Server 2022 | Windows Event Log | WinEventLog:Security |
| System Events | Windows Server 2022 | Windows Event Log | WinEventLog:System |
| Application Events | Windows Server 2022 | Windows Event Log | WinEventLog:Application |

---

## Attacks Simulated & Detections

### 1. SSH Brute Force Attack
**MITRE ATT&CK:** T1110.001 — Password Guessing

**Attack:**
```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://VICTIM-IP -t 4 -I
```

**Detection Query:**

index=main host="raspberrypi" "Failed password" | stats count by host

**Alert Configuration:**
- Name: SSH Brute Force Attack Detected
- Severity: High
- Trigger: More than 5 failed password events
- Type: Real Time

**Result:** ✅ Successfully detected and alerted

---

### 2. Network Reconnaissance Scan
**MITRE ATT&CK:** T1046 — Network Service Discovery

**Attack:**
```bash
nmap -sV VICTIM-IP
nmap -A VICTIM-IP
```

**Detection Query:**

index=main source="/var/log/kern.log" UFW | stats count by host

**Alert Configuration:**
- Name: Network Reconnaissance Scan Detected
- Severity: Medium
- Trigger: More than 10 UFW block events
- Type: Real Time

**Result:** ✅ Successfully detected and alerted

---

### 3. Windows Failed Logon Detection
**MITRE ATT&CK:** T1078 — Valid Accounts

**Detection Query:**

index=main sourcetype="WinEventLog:Security" EventCode=4625 | stats count by host

**Alert Configuration:**
- Name: Windows Failed Logon Detected
- Severity: High
- Trigger: More than 5 failed logon events
- Type: Real Time

**Result:** ✅ Successfully detected and alerted

---

## SPL Detection Queries

| Detection | Query |
|-----------|-------|
| SSH Brute Force | `index=main host="raspberrypi" "Failed password" \| stats count by host` |
| Nmap Scan | `index=main source="/var/log/kern.log" UFW \| stats count by host` |
| Windows Failed Logon | `index=main sourcetype="WinEventLog:Security" EventCode=4625 \| stats count by host` |
| All Security Events | `index=main (sourcetype="WinEventLog:Security") OR (source="/var/log/kern.log") OR (host="raspberrypi") \| timechart count by sourcetype` |
| Top Targeted Hosts | `index=main \| stats count by host \| sort - count` |

---

## SOC Dashboard

Built a custom Splunk Dashboard Studio monitoring dashboard covering all endpoints and attack types simultaneously.

**Panels included:**
- SSH Failed Login Attempts Over Time
- Total Failed SSH Attempts
- SSH Attack Timeline
- Top Targeted Usernames
- Network Scan Activity Over Time
- Total Network Scan Events
- Top Scanned Ports
- Windows Failed Logon Attempts Over Time
- Total Windows Failed Logons
- Security Event Timeline Across All Endpoints

---

## Incident Reports

| Report | Severity | Status |
|--------|---------|--------|
| [IR-2026-001 SSH Brute Force](incident-reports/IR-2026-001-SSH-BruteForce.md) | High | Resolved |
| [IR-2026-002 Nmap Reconnaissance](incident-reports/IR-2026-002-Nmap-Reconnaissance.md) | Medium | Resolved |

---

## Key Findings

- Splunk Universal Forwarder successfully shipped real time logs from both Linux and Windows endpoints to Splunk Enterprise
- Custom SPL detection queries identified brute force and reconnaissance activity within minutes of attack initiation
- Real time alerting pipeline validated across all three attack simulations
- SOC dashboard provided unified visibility across Linux and Windows endpoints simultaneously
- Mean Time to Detect averaged under 3 minutes across all simulations

---

## MITRE ATT&CK Coverage

| Technique ID | Name | Tactic | Detected |
|---|---|---|---|
| T1110.001 | Password Guessing | Credential Access | ✅ |
| T1046 | Network Service Discovery | Discovery | ✅ |
| T1595.001 | Scanning IP Blocks | Discovery | ✅ |
| T1078 | Valid Accounts | Credential Access | ✅ |

---

## Skills Demonstrated

- Splunk Enterprise deployment and configuration
- Multi-platform log ingestion (Linux and Windows)
- SPL query development and alert tuning
- Real time security monitoring and dashboarding
- Attack simulation and detection validation
- Incident response documentation
- MITRE ATT&CK framework mapping

---

## References

- [Splunk Documentation](https://docs.splunk.com)
- [MITRE ATT&CK Framework](https://attack.mitre.org)
- [Hydra Documentation](https://github.com/vanhauser-thc/thc-hydra)
- [Nmap Documentation](https://nmap.org/docs.html)
