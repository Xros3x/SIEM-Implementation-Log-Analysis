# Incident Report — IR-2026-001
## SSH Brute Force Attack Detection

| Field | Details |
|-------|---------|
| Report ID | IR-2026-001 |
| Date | May 2026 |
| Severity | High |
| Status | Resolved |
| Analyst | Tyrik Parker |

---

## Executive Summary
A brute force attack was detected against a Linux endpoint in the homelab environment. The attack involved repeated failed SSH authentication attempts generated from an attacker machine using an automated password guessing tool. The attack was detected in real time by Splunk SIEM via authentication log monitoring and a pre-configured high severity alert.

---

## Incident Timeline

| Time | Event |
|------|-------|
| T+0:00 | Hydra brute force tool initiated on attacker machine |
| T+0:01 | Failed SSH authentication attempts begin appearing in auth.log |
| T+0:02 | Splunk Universal Forwarder ships logs to Splunk Enterprise |
| T+0:03 | Splunk alert triggers after threshold of 5 failed attempts exceeded |
| T+0:05 | Analyst identifies spike on SOC dashboard |
| T+0:10 | Attack simulation stopped and incident documented |

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
| Attack Type | SSH Brute Force / Dictionary Attack |
| Tool Used | Hydra |
| Wordlist | rockyou.txt |
| Target Service | SSH Port 22 |
| Target Usernames | root and common usernames |
| Attack Result | Unsuccessful — no valid credentials found |
| Source IP | ATTACKER-IP (internal homelab) |
| Destination IP | VICTIM-IP (internal homelab) |

---

## Detection Details

| Field | Details |
|-------|---------|
| Detection Source | Splunk SIEM |
| Log Source | /var/log/auth.log via Splunk Universal Forwarder |
| Search Query | `index=main host="raspberrypi" "Failed password"` |
| Alert Name | SSH Brute Force Attack Detected |
| Alert Severity | High |
| Detection Time | Within 3 minutes of attack initiation |

---

## Root Cause Analysis
The victim machine had SSH enabled with password based authentication and no account lockout policy configured. This allowed an automated tool to make unlimited login attempts without restriction, generating a high volume of failed authentication events.

---

## Recommendations

| Priority | Recommendation | Action |
|----------|---------------|--------|
| High | Disable root SSH login | Set PermitRootLogin no in sshd_config |
| High | Implement fail2ban | Auto-block IPs after 5 failed attempts |
| High | Use SSH key authentication | Disable password based SSH login |
| Medium | Restrict SSH to trusted IPs | Configure firewall rules on port 22 |
| Medium | Enable account lockout policy | Lock accounts after repeated failures |
| Low | Monitor SSH logs continuously | Keep Splunk alert active and tuned |

---

## MITRE ATT&CK Mapping

| Field | Details |
|-------|---------|
| Tactic | Credential Access |
| Technique | T1110 — Brute Force |
| Sub-technique | T1110.001 — Password Guessing |

---

## Lessons Learned
- Real time log forwarding with Splunk Universal Forwarder enables fast detection of brute force activity
- Pre-configured alerts significantly reduce detection time compared to manual log review
- Default SSH configurations leave systems vulnerable to automated attacks
- Dictionary attacks using common wordlists can generate thousands of attempts in minutes

---

## Conclusion
The homelab SOC environment successfully detected a simulated SSH brute force attack within minutes of initiation using Splunk SIEM and pre-configured alerting. This exercise demonstrated the value of centralized log management, real time alerting, and proactive monitoring in identifying and responding to credential based attacks.

---

**Analyst:** Tyrik
**Certification:** CompTIA CySA+
