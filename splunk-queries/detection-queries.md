# Splunk Detection Queries

All SPL queries used in the SIEM Implementation and Log Analysis project.

---

## SSH Brute Force Detection

### Basic Detection
```spl
index=main host="raspberrypi" "Failed password"
| stats count by host
```

### Timeline View
```spl
index=main host="raspberrypi" "Failed password"
| timechart count
```

### Attack Timeline with Details
```spl
index=main host="raspberrypi" "Failed password"
| table _time, host, source
| sort - _time
```

### Top Targeted Usernames
```spl
index=main host="raspberrypi" "Failed password for"
| rex "Failed password for (?<username>\S+)"
| stats count by username
| sort - count
```

---

## Network Reconnaissance Detection

### Basic UFW Detection
```spl
index=main source="/var/log/kern.log" UFW
| stats count by host
```

### Scan Activity Over Time
```spl
index=main source="/var/log/kern.log" UFW
| timechart count
```

### Top Scanned Ports
```spl
index=main source="/var/log/kern.log" UFW
| rex "DPT=(?<dest_port>\d+)"
| stats count by dest_port
| sort - count
```

---

## Windows Event Log Detection

### Failed Logon Events
```spl
index=main sourcetype="WinEventLog:Security" EventCode=4625
| stats count by host
```

### Failed Logon Timeline
```spl
index=main sourcetype="WinEventLog:Security" EventCode=4625
| timechart count
```

### All Windows Security Events
```spl
index=main sourcetype="WinEventLog:Security"
| stats count by EventCode
| sort - count
```

### Process Creation Monitoring
```spl
index=main sourcetype="WinEventLog:Security" EventCode=4688
| stats count by host
```

---

## Cross Platform Queries

### All Security Events Across All Endpoints
```spl
index=main (sourcetype="WinEventLog:Security")
OR (source="/var/log/kern.log")
OR (host="raspberrypi")
| timechart count by sourcetype
```

### Top Targeted Hosts
```spl
index=main
| stats count by host
| sort - count
```

### Security Event Summary
```spl
index=main (sourcetype="WinEventLog:Security" EventCode=4625)
OR (source="/var/log/kern.log" UFW)
OR (host="raspberrypi" "Failed password")
| stats count by sourcetype
| sort - count
```
