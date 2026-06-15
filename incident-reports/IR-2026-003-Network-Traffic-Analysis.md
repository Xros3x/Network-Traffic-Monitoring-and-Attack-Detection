# Incident Report — IR-2026-003
## Network Traffic Analysis — Reconnaissance and Brute Force Attack Chain

| Field | Details |
|-------|---------|
| Report ID | IR-2026-003 |
| Date | June 2026 |
| Severity | High |
| Status | Resolved |
| Analyst | Tyrik Parker |
| Detection Method | Splunk SIEM + Wireshark Packet Analysis |

---

## Executive Summary

A two-stage attack was detected against a Linux endpoint in the homelab environment. The attacker first conducted network reconnaissance using Nmap to identify open services, followed by a targeted SSH brute force attack using Hydra. The attack was detected through Splunk SIEM firewall log analysis which identified an anomalous source IP generating 7,012 firewall block events. Investigation was escalated to Wireshark packet level analysis which confirmed both attack signatures through packet pattern examination.

---

## Detection Methodology

This investigation followed the real world SOC analyst workflow:

1. Splunk alert fired on high volume UFW firewall block events
2. SPL query extracted and ranked source IPs by firewall hit count
3. Single IP with 7,012 hits identified as anomalous
4. Pivoted to Wireshark for packet level confirmation
5. SYN packet flood confirmed Nmap reconnaissance signature
6. RST packet pattern on port 22 confirmed SSH brute force
7. IOCs extracted and documented

---

## Incident Timeline

| Time | Event | Detection Source |
|------|-------|-----------------|
| T+0:00 | Nmap scan initiated from attacker machine | Wireshark |
| T+0:01 | UFW begins blocking sequential port probes | kern.log |
| T+0:02 | Splunk detects spike in UFW firewall events | Splunk SIEM |
| T+0:03 | Hydra SSH brute force attack initiated | Wireshark |
| T+0:04 | Failed SSH attempts logged in auth.log | auth.log |
| T+0:05 | Splunk SSH brute force alert triggers | Splunk SIEM |
| T+0:06 | Analyst identifies suspicious IP via SPL | Splunk investigation |
| T+0:08 | Wireshark confirms attack patterns at packet level | Packet analysis |
| T+0:12 | IOCs extracted and incident documented | Investigation |

---

## Affected Systems

| System | Role | Status |
|--------|------|--------|
| VICTIM-HOST | Linux Endpoint (Raspberry Pi 5) | Targeted — not compromised |
| ATTACKER-HOST | Kali Linux VM | Attack source |
| SPLUNK-SERVER | SIEM (Ubuntu 24.04 VM) | Operational |

---

## Attack Details

### Stage 1 — Network Reconnaissance

| Field | Details |
|-------|---------|
| Attack Type | Network Reconnaissance / Port Scan |
| Tool | Nmap with -A -O flags |
| Method | TCP SYN scan across multiple ports |
| Ports Targeted | Multiple including 22, 80, 443, 993, 587, 8888 |
| Detection | UFW firewall block events in kern.log |
| Packet Signature | Sequential SYN packets with no completed handshakes |
| MITRE Technique | T1046 — Network Service Discovery |

### Stage 2 — SSH Brute Force

| Field | Details |
|-------|---------|
| Attack Type | SSH Brute Force / Dictionary Attack |
| Tool | Hydra with rockyou.txt wordlist |
| Target | SSH service on port 22 |
| Target Account | root |
| Detection | auth.log failed password events + Wireshark RST packets |
| Packet Signature | Repeating SYN→SYN-ACK→RST pattern on port 22 |
| MITRE Technique | T1110.001 — Password Guessing |

---

## Wireshark Findings

### Protocol Distribution
| Protocol | Packets | Percentage |
|----------|---------|------------|
| TCP | 2,376 | 95.2% |
| SSH | 121 | 4.8% |
| UDP | 21 | 0.8% |
| ICMP | 8 | 0.3% |
| ARP | 52 | 2.1% |

TCP at 95.2% of all traffic is highly abnormal and a strong indicator of active attack behavior.

### Conversation Analysis
Single IP pair accounted for 2,393 packets and 174 kB of traffic — highest conversation volume by significant margin. Confirms the suspicious IP identified in Splunk was the primary attack source.

### TCP Stream Analysis
SSH TCP stream showed repeating SYN→SYN-ACK→RST pattern confirming failed authentication attempts consistent with automated brute force tool behavior.

---

## Splunk Findings

| Metric | Value |
|--------|-------|
| Total UFW events analyzed | 9,031 |
| Events from suspicious IP | 7,012 |
| Percentage of total events | 77.7% |
| SSH failed password events | Multiple |
| Alerts triggered | 2 (Network Scan + SSH Brute Force) |
| Mean Time to Detect | Under 3 minutes |

---

## IOC Summary

| IOC | Value | Confidence |
|-----|-------|------------|
| Attacker IP | ATTACKER-IP | High |
| Target IP | VICTIM-IP | High |
| Target Port | 22 (SSH) | High |
| Attack Tools | Nmap + Hydra | High |
| Firewall Events | 7,012 | High |

---

## Root Cause Analysis

The victim machine had SSH enabled with password-based authentication and no account lockout policy or rate limiting configured. UFW firewall was enabled and successfully blocked unauthorized access attempts. The attack followed a classic reconnaissance-then-exploit pattern where the attacker first mapped the network with Nmap to identify open services before targeting SSH specifically with a brute force attack.

---

## Recommendations

| Priority | Recommendation | Action |
|----------|---------------|--------|
| Critical | Disable SSH password authentication | Enable SSH key-based auth only |
| High | Implement fail2ban | Auto-block IPs after 5 failed attempts |
| High | Disable root SSH login | Set PermitRootLogin no in sshd_config |
| High | Implement rate limiting on SSH | Use iptables or UFW rate limiting rules |
| Medium | Restrict SSH to trusted IPs | Firewall whitelist for SSH access |
| Medium | Enable IDS/IPS | Deploy Suricata for signature-based detection |
| Low | Regular firewall log review | Weekly UFW log analysis in Splunk |

---

## Lessons Learned

- Starting investigation from SIEM logs rather than known attacker IP mirrors real world SOC workflow where attacker identity is unknown at detection time
- Firewall logs provide strong first-stage detection — a single IP generating 7,012 blocks in a short window is immediately suspicious regardless of other context
- Wireshark packet analysis provides definitive confirmation of attack type and tools used beyond what log analysis alone can show
- The reconnaissance-to-brute-force attack chain is a common real world pattern — detecting the scan phase early can prevent the follow-up attack
- Protocol distribution anomalies (TCP at 95%+) are a reliable indicator of active attack activity

---

## Conclusion

The homelab SOC environment successfully detected a two-stage attack using both SIEM log analysis and packet level investigation. This investigation demonstrated the complete SOC analyst workflow from initial alert through deep packet analysis to IOC extraction and formal documentation. The combination of Splunk for log-based detection and Wireshark for packet confirmation provides comprehensive attack visibility that neither tool alone can achieve.

---

**Analyst:** Tyrik Parker
**Certification:** CompTIA CySA+
**Clearance:** Active Secret
