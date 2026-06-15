# Indicators of Compromise — Network Traffic Analysis

IOCs extracted during investigation of simulated network reconnaissance and SSH brute force attacks.

---

## IOC Table

| IOC Type | Value | Source | Confidence | Notes |
|----------|-------|--------|------------|-------|
| Attacker IP | ATTACKER-IP | Splunk UFW logs + Wireshark | High | Generated 7,012 firewall block events |
| Target IP | VICTIM-IP | Splunk UFW logs + Wireshark | High | Raspberry Pi 5 victim endpoint |
| Target Port | 22 (SSH) | Wireshark packet capture | High | Primary attack target |
| Attack Tool | Nmap | Packet signature — SYN scan pattern | High | Sequential SYN packets to multiple ports |
| Attack Tool | Hydra | Packet signature — RST pattern on port 22 | High | Automated password guessing |
| Scan Type | SYN Scan | Wireshark tcp.flags.syn filter | High | No completed handshakes during scan |
| Brute Force Pattern | SYN→SYN-ACK→RST | Wireshark TCP stream | High | Repeated failed SSH connections |
| Firewall Events | 7,012 | Splunk kern.log UFW | High | Far exceeds normal traffic baseline |
| SSH Packets | 121 | Wireshark capture | High | All within short time window |
| Protocol Distribution | TCP 95.2% | Wireshark Protocol Hierarchy | High | Highly abnormal — attack indicator |

---

## Attack Timeline

| Time | Event | Source |
|------|-------|--------|
| T+0:00 | Nmap scan initiated from attacker machine | Wireshark |
| T+0:01 | UFW begins blocking sequential port probes | kern.log |
| T+0:02 | Splunk detects spike in UFW block events | Splunk alert |
| T+0:03 | Hydra SSH brute force initiated | Wireshark |
| T+0:04 | Failed SSH attempts appear in auth.log | auth.log |
| T+0:05 | Splunk SSH brute force alert triggers | Splunk alert |
| T+0:06 | Analyst identifies suspicious IP via SPL | Splunk investigation |
| T+0:08 | Pivot to Wireshark confirms attack patterns | Wireshark analysis |
| T+0:12 | IOCs extracted and documented | Investigation |

---

## Threat Intelligence Enrichment

| IOC | VirusTotal | AbuseIPDB | Notes |
|-----|-----------|-----------|-------|
| ATTACKER-IP | Internal IP — not applicable | Internal IP — not applicable | Homelab internal address |
| 100.72.59.55 | External IP — check manually | External IP — check manually | Appeared in UFW logs |

> Note: In a real SOC environment all external IPs would be enriched through threat intelligence platforms including VirusTotal, AbuseIPDB, and Shodan to determine reputation and historical attack activity.

---

## MITRE ATT&CK Mapping

| Technique | ID | Evidence |
|-----------|-----|---------|
| Network Service Discovery | T1046 | Sequential SYN packets to multiple ports in UFW logs |
| Scanning IP Blocks | T1595.001 | 7,012 UFW block events from single source |
| Password Guessing | T1110.001 | Repeated SYN→RST pattern on port 22 |
| Valid Accounts | T1078 | SSH authentication attempts against root account |
