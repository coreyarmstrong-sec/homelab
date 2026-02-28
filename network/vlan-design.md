# Network Design & VLAN Architecture

## Design Goals

1. **Segmentation** — Isolate traffic by trust level and function
2. **Least Privilege** — Traffic only flows where explicitly permitted
3. **Visibility** — All traffic logged and available to SIEM
4. **Scalability** — Easy to add new segments without restructuring

---

## VLAN Breakdown

| VLAN ID | Name | Subnet | Purpose | Internet Access | Inter-VLAN |
|---|---|---|---|---|---|
| VLAN 10 | LAN | 10.10.10.0/24 | General use devices — laptops, phones | Yes | Restricted |
| VLAN 20 | IoT | 10.10.20.0/24 | Smart home, cameras, IoT devices | Yes | Denied |
| VLAN 30 | Servers | 10.10.30.0/24 | Proxmox, Docker, lab VMs | Yes | Restricted |
| VLAN 99 | Management | 10.10.99.0/24 | pfSense admin, switch management | No | Admin only |

---

## Traffic Flow Rules

### VLAN 10 (LAN) → Can reach:
- Internet ✅
- VLAN 30 (Servers) on specific ports only ✅
- VLAN 20 (IoT) — **DENIED** ❌
- VLAN 99 (Management) — **DENIED** ❌

### VLAN 20 (IoT) → Can reach:
- Internet ✅
- All other VLANs — **DENIED** ❌

### VLAN 30 (Servers) → Can reach:
- Internet ✅
- VLAN 10 responses only (stateful) ✅
- VLAN 20 (IoT) — **DENIED** ❌
- VLAN 99 (Management) — **DENIED** ❌

### VLAN 99 (Management) → Can reach:
- pfSense admin interface ✅
- All VLANs for administrative access ✅
- Internet — **DENIED** ❌

---

## Why This Design?

**IoT isolation** is the most critical decision. IoT devices are notorious for poor security — weak default passwords, unpatched firmware, unexpected outbound connections. By placing them in a dedicated VLAN with internet-only access, a compromised IoT device cannot pivot to reach laptops, servers, or management interfaces.

**Management VLAN isolation** ensures that even if a device on the LAN segment is compromised, the attacker cannot reach the firewall's administrative interface from that segment. Management access requires being on VLAN 99 specifically.

**Explicit deny logging** on inter-VLAN rules means any lateral movement attempt generates a log event in Splunk — turning the firewall into a detection tool, not just an enforcement tool.

---

## IP Addressing Scheme

| Device | VLAN | IP Address | Notes |
|---|---|---|---|
| pfSense (LAN gateway) | 10 | 10.10.10.1 | Default gateway for VLAN 10 |
| pfSense (IoT gateway) | 20 | 10.10.20.1 | Default gateway for VLAN 20 |
| pfSense (Server gateway) | 30 | 10.10.30.1 | Default gateway for VLAN 30 |
| pfSense (Mgmt gateway) | 99 | 10.10.99.1 | Default gateway for VLAN 99 |
| Proxmox Host | 30 | 10.10.30.10 | Static assignment |
| Splunk | 30 | 10.10.30.20 | Static assignment |

DHCP ranges start at .100 for each subnet — .1 through .99 reserved for static assignments.
