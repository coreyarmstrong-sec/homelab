# pfSense Firewall Configuration

## Overview

pfSense is the core of this lab's security architecture. It sits between the internet and all internal networks, handles VLAN routing, enforces firewall policy, manages DHCP for all segments, and generates logs fed into Splunk.

**Version:** pfSense CE  
**Role:** Perimeter firewall, VLAN router, DHCP server, DNS resolver, log source

---

## VLAN Interface Configuration

Each VLAN is configured as a sub-interface (virtual interface) on the physical LAN NIC:

| Interface | VLAN Tag | IP Address | Description |
|---|---|---|---|
| LAN | — | 10.10.10.1/24 | Main LAN segment |
| OPT1 (IoT) | VLAN 20 | 10.10.20.1/24 | Isolated IoT segment |
| OPT2 (Servers) | VLAN 30 | 10.10.30.1/24 | Lab/server segment |
| OPT3 (Mgmt) | VLAN 99 | 10.10.99.1/24 | Management segment |

---

## Firewall Rule Logic

pfSense processes rules top-down, first match wins. Rules are organized per interface — traffic is evaluated as it **enters** the interface.

### Core Rule Philosophy
- Default deny all inter-VLAN traffic
- Explicit permit rules for necessary flows
- Log all deny hits for SIEM visibility
- Permit established/related traffic (stateful)

### Example Rule Structure (IoT VLAN)
```
Priority  Action   Source         Destination    Port    Description
1         Block    VLAN20_net     RFC1918_alias  any     Block IoT → all private IPs
2         Pass     VLAN20_net     any            any     Allow IoT → internet only
```

The RFC1918 alias contains all private IP ranges (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16) — rule 1 blocks IoT from reaching anything internal before rule 2 allows internet access.

---

## NAT Configuration

**Type:** NAT Overload (PAT)  
All internal VLANs share the single WAN IP address using port address translation. Each VLAN has its own outbound NAT rule mapping its subnet to the WAN interface.

---

## DHCP Server

pfSense runs DHCP for all four VLANs:

| VLAN | DHCP Range | DNS | Lease Time |
|---|---|---|---|
| LAN (10) | 10.10.10.100 – .200 | 10.10.10.1 | 24 hrs |
| IoT (20) | 10.10.20.100 – .200 | 10.10.20.1 | 24 hrs |
| Servers (30) | 10.10.30.100 – .200 | 10.10.30.1 | 7 days |
| Mgmt (99) | 10.10.99.100 – .150 | 10.10.99.1 | 24 hrs |

Static DHCP mappings are configured for all servers to ensure consistent IP assignment.

---

## DNS Resolver

pfSense Unbound DNS Resolver handles name resolution for all internal clients:
- Forwards external queries to upstream resolvers (1.1.1.1, 8.8.8.8)
- Local hostname resolution for lab devices
- DNS rebinding protection enabled

---

## Logging

pfSense sends firewall logs to Splunk via syslog (UDP 514). All deny rule hits are logged — providing visibility into:
- Inter-VLAN traffic attempts (potential lateral movement)
- Port scans against the firewall
- Outbound connection attempts from unexpected sources

---

## Security Hardening Applied

- WebGUI access restricted to Management VLAN only
- SSH disabled on WAN interface
- Admin account renamed from default
- Anti-lockout rule enabled for management VLAN
- BOGON networks blocked on WAN
- Private networks blocked on WAN (RFC1918 ingress filtering)
