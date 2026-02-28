# Splunk SIEM — Security Monitoring

## Overview

Splunk serves as the centralized log aggregation and analysis platform for the homelab. All security-relevant events from network devices and endpoints are forwarded to Splunk for correlation, alerting, and investigation.

Splunk experience developed through **Blue Team Level 1 (BTL1)** coursework covering real-world SOC analyst workflows.

---

## BTL1 Splunk Skills Developed

Through Blue Team Level 1 training, the following Splunk capabilities were practiced:

| Skill | Description |
|---|---|
| SPL Queries | Writing Search Processing Language queries to filter and analyze log data |
| Log Analysis | Investigating Windows Event Logs, network logs, and authentication events |
| Threat Detection | Identifying indicators of compromise (IOCs) within log data |
| Timeline Analysis | Reconstructing attack timelines from log events |
| Alert Triage | Evaluating alerts to distinguish true positives from false positives |
| Incident Investigation | Following structured investigation methodology to determine scope and impact |

---

## Planned Data Sources (Homelab Integration)

| Source | Log Type | Value |
|---|---|---|
| pfSense Firewall | Firewall allow/deny events | Network traffic visibility, lateral movement detection |
| pfSense DHCP | DHCP lease events | Asset tracking, rogue device detection |
| Windows Endpoints | Security, System, Application Event Logs | Authentication monitoring, process execution |
| Linux/RHEL Servers | Auth logs, syslog | SSH brute force detection, privilege escalation |
| Docker Containers | Application logs | Service anomaly detection |

---

## Key SPL Queries (BTL1 Practice)

### Failed Authentication Events (Windows)
```spl
index=* EventCode=4625
| stats count by src_ip, user
| sort -count
| head 20
```

### Successful Login After Multiple Failures (Brute Force Success)
```spl
index=* EventCode=4625 OR EventCode=4624
| transaction user maxspan=1h
| where EventCode=4624 AND event_count > 5
```

### Network Traffic by Source IP
```spl
index=firewall action=block
| stats count by src_ip dest_ip dest_port
| sort -count
```

### Top Blocked Destinations (Potential C2 Detection)
```spl
index=firewall action=block
| stats count by dest_ip
| sort -count
| head 10
```

---

## Detection Use Cases

| Use Case | Log Source | Detection Logic |
|---|---|---|
| Port Scan Detection | pfSense firewall | High volume of blocked connections from single source IP in short window |
| Brute Force (SSH) | Linux auth.log | Multiple EventCode=4625 equivalent failures from same source |
| IoT Phone-Home | pfSense firewall | IoT VLAN traffic attempting to reach internal IPs (should be blocked) |
| Unauthorized Inter-VLAN | pfSense firewall | Any traffic hitting inter-VLAN deny rules |
| New Device on Network | pfSense DHCP | DHCP lease issued to unknown MAC address |

---

## Workflow: Alert Triage Process

When an alert fires in Splunk, I follow this structured workflow:

1. **Validate** — Is this a real event or a false positive? Check context.
2. **Scope** — How many hosts/users are affected? What time range?
3. **Investigate** — Pull related logs to build a timeline of activity.
4. **Classify** — True positive, false positive, or needs escalation?
5. **Document** — Record findings, evidence, and actions taken.

---

*Active BTL1 coursework. Homelab Splunk integration in progress — documentation will be updated as data sources are connected.*
