# Incident Report — IR-2026-002
## Network Reconnaissance Detection — Nmap Scan

| Field | Details |
|-------|---------|
| Report ID | IR-2026-002 |
| Date | May 2026 |
| Severity | Medium |
| Status | Resolved |
| Analyst | Tyrik Parker |

---

## Executive Summary
A network reconnaissance scan was detected against a Linux endpoint in the homelab environment. The activity was identified through UFW firewall logs forwarded to Splunk Enterprise in real time. The scan was performed using Nmap from an attacker machine simulating pre-attack reconnaissance activity.

---

## Incident Timeline

| Time | Event |
|------|-------|
| T+0:00 | Nmap scan initiated from attacker machine |
| T+0:01 | UFW firewall begins blocking and logging probe attempts |
| T+0:02 | kern.log entries forwarded to Splunk via Universal Forwarder |
| T+0:03 | Splunk alert triggers after 10+ UFW block events detected |
| T+0:05 | Analyst identifies scan activity on SOC dashboard |
| T+0:08 | Scan activity documented and incident report created |

---

## Affected Systems

| System | Role | Status |
|--------|------|--------|
| VICTIM-HOST | Linux Endpoint (Raspberry Pi 5) | Targeted — not compromised |
| ATTACKER-HOST | Kali Linux VM | Attack source |
| SPLUNK-SERVER | SIEM (Ubuntu 24.04 VM) | Operational |

---

## Attack Details

| Field | Details |
|-------|---------|
| Attack Type | Network Reconnaissance / Port Scan |
| Tool Used | Nmap |
| Scan Type | Service Version Detection (-sV) and Aggressive Scan (-A) |
| Ports Probed | Multiple including 22, 80, 443, 3306 |
| Attack Result | Blocked by UFW firewall |
| Source IP | ATTACKER-IP (internal homelab) |
| Destination IP | VICTIM-IP (internal homelab) |

---

## Detection Details

| Field | Details |
|-------|---------|
| Detection Source | Splunk SIEM |
| Log Source | /var/log/kern.log via Splunk Universal Forwarder |
| Search Query | `index=main source="/var/log/kern.log" UFW` |
| Alert Name | Network Reconnaissance Scan Detected |
| Alert Severity | Medium |
| Detection Time | Within 3 minutes of scan initiation |

---

## Key Indicators of Compromise

- Rapid sequential connection attempts across multiple ports from single source
- High volume of UFW BLOCK events within short time window
- Destination ports spanning well known service ranges
- TCP SYN packets without completed handshakes

---

## Root Cause Analysis
The victim machine had UFW firewall enabled which successfully blocked the scan attempts. However the volume and pattern of blocked connection attempts clearly indicated automated reconnaissance activity rather than normal traffic.

---

## Recommendations

| Priority | Recommendation | Action |
|----------|---------------|--------|
| High | Keep firewall logging enabled | Maintain UFW logging at medium level |
| High | Alert on scan patterns | Keep Splunk alert active and tuned |
| Medium | Implement geo-blocking | Block unexpected source IP ranges |
| Medium | Alert on sensitive port scans | Create specific alerts for ports 22, 3306, 443 |
| Low | Deploy IDS/IPS | Add Suricata for deeper network inspection |

---

## MITRE ATT&CK Mapping

| Field | Details |
|-------|---------|
| Tactic | Discovery |
| Technique | T1046 — Network Service Discovery |
| Also Mapped | T1595.001 — Scanning IP Blocks |

---

## Correlation Note
This reconnaissance scan preceded a simulated SSH brute force attack (IR-2026-001), demonstrating a complete attack chain from initial reconnaissance to credential attack. This correlation highlights the importance of monitoring for reconnaissance activity as an early warning indicator of incoming attacks.

---

## Lessons Learned
- Firewall logs are a critical data source for detecting reconnaissance activity
- Nmap scans generate distinctive patterns in firewall logs that are easy to detect with proper alerting
- Correlating reconnaissance alerts with subsequent attack attempts provides valuable threat intelligence
- Early detection of reconnaissance activity can prevent downstream credential attacks

---

## Conclusion
The homelab SOC environment successfully detected network reconnaissance activity using UFW firewall logs forwarded to Splunk SIEM. The detection preceded a follow up brute force attack demonstrating the value of full attack chain visibility in a SOC environment.

---

**Analyst:** Tyrik
**Certification:** CompTIA CySA+
