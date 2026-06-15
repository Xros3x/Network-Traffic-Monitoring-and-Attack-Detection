# Splunk Detection Queries — Network Traffic Analysis

All SPL queries used in the Network Traffic Monitoring and Attack Detection project.

---

## Suspicious IP Identification

### Extract and rank source IPs from firewall logs
```spl
index=main source="/var/log/kern.log" UFW
| eval src_ip=mvindex(split(_raw,"SRC="),1)
| eval src_ip=mvindex(split(src_ip," "),0)
| stats count by src_ip
| sort - count
```

### UFW events over time
```spl
index=main source="/var/log/kern.log" UFW
| timechart count
```

### UFW events by destination port
```spl
index=main source="/var/log/kern.log" UFW
| eval dst_port=mvindex(split(_raw,"DPT="),1)
| eval dst_port=mvindex(split(dst_port," "),0)
| stats count by dst_port
| sort - count
```

---

## SSH Brute Force Detection

### Failed SSH authentication events
```spl
index=main host="raspberrypi" "Failed password"
| stats count by host
```

### SSH failed login timeline
```spl
index=main host="raspberrypi" "Failed password"
| timechart count
```

---

## Attack Chain Correlation

### Full attack timeline — both attacks combined
```spl
index=main (source="/var/log/kern.log" UFW) OR (host="raspberrypi" "Failed password")
| eval attack_type=if(match(_raw,"UFW"),"Network Scan","SSH Brute Force")
| timechart count by attack_type
```

### MTTD calculation
```spl
index=main (source="/var/log/kern.log" UFW) OR (host="raspberrypi" "Failed password")
| eval attack_type=if(match(_raw,"UFW"),"Network Scan","SSH Brute Force")
| stats min(_time) as first_seen max(_time) as last_seen by attack_type
| eval duration=last_seen-first_seen
| convert ctime(first_seen) ctime(last_seen)
```
