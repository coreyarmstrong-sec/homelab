# Proxmox Virtualization & Docker

## Overview

Proxmox VE runs as a Type-1 (bare metal) hypervisor on dedicated hardware in the homelab. It hosts virtual machines and LXC containers, providing an isolated lab environment for testing, learning, and running self-hosted services.

**Proxmox Version:** Proxmox VE  
**Hardware:** Dedicated homelab server on VLAN 30 (Servers segment)  
**IP:** 10.10.30.10 (static DHCP reservation)

---

## Why Proxmox?

Proxmox was chosen over VMware Workstation or VirtualBox for several reasons:

- **Type-1 hypervisor** — Runs directly on hardware, not inside a host OS
- **Enterprise-grade** — Same technology used in production data centers
- **Web UI + CLI** — Mirrors real-world virtualization management experience
- **LXC containers** — Lightweight OS-level virtualization alongside full VMs
- **Networking** — Supports bridges and VLANs, integrating with pfSense segmentation

---

## Current Virtual Machines / Containers

| Name | Type | OS | Purpose | VLAN |
|---|---|---|---|---|
| docker-host | VM | Ubuntu Server | Docker container host | 30 |
| rhel-lab | VM | RHEL | Linux/RHCSA practice (planned) | 30 |
| windows-lab | VM | Windows Server | Active Directory practice (planned) | 30 |

---

## Docker — Self-Hosted Services

Docker runs on the `docker-host` VM. Services are containerized to keep the host OS clean and to practice container management skills.

### Running Services

| Container | Image | Port | Purpose |
|---|---|---|---|
| Nextcloud | nextcloud:latest | 8080 | Self-hosted cloud storage — file sync, calendar, contacts |
| Jellyfin | jellyfin/jellyfin | 8096 | Self-hosted media server |

### Docker Networking

Containers run on a custom Docker bridge network, not the default `docker0` bridge. This allows:
- Containers to communicate with each other by name
- Controlled exposure — only necessary ports mapped to host
- Future integration with pfSense firewall rules per container traffic

### Key Commands Used Regularly

```bash
# Check running containers
docker ps

# View logs for a container
docker logs nextcloud

# Update a container image
docker pull nextcloud:latest
docker compose up -d

# Inspect container network
docker inspect nextcloud | grep IPAddress

# Enter a running container
docker exec -it nextcloud bash
```

---

## Proxmox + pfSense Integration

All Proxmox VMs are placed on VLAN 30 (Servers segment). pfSense controls what traffic can enter or leave this segment:

- VMs can reach the internet for updates and external services
- VMs cannot initiate connections to the IoT VLAN
- LAN devices can reach specific VM services on specific ports only
- All traffic is logged and visible in Splunk

This design means the virtualization layer is isolated — a compromised VM cannot freely reach other network segments.

---

## Skills Demonstrated

- Type-1 hypervisor deployment and management
- VM creation, configuration, and lifecycle management
- Docker image management and container orchestration
- Docker networking and port mapping
- Integration of virtualization with network security controls
- Self-hosted service administration
