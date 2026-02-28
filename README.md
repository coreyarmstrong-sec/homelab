# 🏠 Homelab — Enterprise Security Environment

**Owner:** Corey Armstrong | Orlando, FL  
**Purpose:** Hands-on cybersecurity and networking skill development  
**Status:** Active — continuously updated

---

## Overview

This repository documents my personal security homelab — a multi-zone, enterprise-style network environment built from scratch to develop real-world skills in network security, firewall management, SIEM operations, virtualization, and Linux system administration.

Everything here mirrors what you'd find in a real enterprise environment. The goal was never to follow a tutorial — it was to build something real, break it, troubleshoot it, and understand why it works.

---

## Network Architecture

```
                        [ INTERNET ]
                             |
                    [ pfSense Firewall ]
                    192.168.1.1 (WAN)
                             |
              _______________+_______________
             |               |               |
        [VLAN 10]       [VLAN 20]       [VLAN 30]
        LAN/General     IoT Devices     Servers/Lab
        10.10.10.0/24   10.10.20.0/24   10.10.30.0/24
             |                               |
        [End Devices]               [Proxmox Hypervisor]
                                    [Docker Containers]
                                    [RHEL Servers]
```

**Design Principle:** Strict inter-VLAN isolation with explicit firewall rules. No traffic crosses VLAN boundaries unless explicitly permitted. IoT devices are completely isolated and cannot initiate connections to any other segment.

---

## Components

| Component | Technology | Role |
|---|---|---|
| Firewall / Router | pfSense | Perimeter security, VLAN routing, NAT, firewall rules |
| Virtualization | Proxmox VE | Type-1 hypervisor hosting VMs and LXC containers |
| Containers | Docker | Self-hosted services (Nextcloud, Jellyfin) |
| SIEM | Splunk | Log aggregation, threat detection, security monitoring |
| Servers | RHEL (Planned) | Linux server administration practice |
| Network Monitoring | Wireshark, Nmap | Traffic analysis and network discovery |

---

## Repository Structure

```
homelab/
├── README.md                  ← You are here
├── network/
│   ├── README.md              ← Network design and topology
│   ├── vlan-design.md         ← VLAN breakdown and IP scheme
│   └── topology-diagram.png   ← Visual network map
├── pfsense/
│   ├── README.md              ← pfSense configuration overview
│   ├── firewall-rules.md      ← Firewall ruleset documentation
│   ├── vlan-config.md         ← VLAN interface configuration
│   └── nat-config.md          ← NAT/PAT configuration
├── splunk/
│   ├── README.md              ← Splunk deployment and use cases
│   ├── data-sources.md        ← Log sources being ingested
│   └── detection-rules.md     ← Custom alerts and detections
└── proxmox-docker/
    ├── README.md              ← Virtualization overview
    └── docker-services.md     ← Containerized services
```

---

## Security Controls Implemented

### Network Segmentation
- 3 VLANs separating general use, IoT, and server/lab traffic
- pfSense enforces inter-VLAN routing rules — default deny between segments
- IoT VLAN has internet access only — cannot reach any internal resource

### Firewall Policy
- Explicit permit rules with implicit deny on all interfaces
- Stateful inspection on all traffic
- Logging enabled on deny rules for SIEM ingestion

### Layer 2 Security
- DHCP Snooping — prevents rogue DHCP servers
- Dynamic ARP Inspection (DAI) — prevents ARP spoofing/poisoning
- Port Security — limits MAC addresses per port

### Access Hardening
- Management VLAN isolated from all other traffic
- SSH-only administrative access (no Telnet)
- Strong password policy enforced across all devices

---

## Skills Demonstrated

- **Network Architecture** — Designed multi-zone topology with security segmentation
- **Firewall Management** — Built and maintained rule sets enforcing least-privilege traffic flow
- **SIEM Operations** — Deployed Splunk for centralized log collection and threat detection
- **Virtualization** — Deployed and managed Proxmox with VMs and containers
- **Linux Administration** — Managing RHEL-based systems (in progress)
- **Troubleshooting** — Diagnosing network issues, misconfigurations, and security events

---

## What's Next

- [ ] Deploy RHEL servers for RHCSA practice and documentation
- [ ] Configure Splunk to ingest pfSense firewall logs
- [ ] Build custom Splunk dashboards for network security monitoring
- [ ] Add IDS/IPS (Suricata) to pfSense
- [ ] Document incident response runbooks

---

*This homelab is actively maintained and updated as I learn new skills and pursue additional certifications.*
