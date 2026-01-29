# P1-1: Proxmox Segmentation Lab (pfSense + Multi-LAN)

## Overview
This repo documents a segmented Proxmox homelab designed to mimic an enterprise environment. The lab uses **pfSense as the edge firewall/router** with **multiple Proxmox bridges** to separate traffic into distinct security zones (Enterprise LAN vs Vulnerable LAN), enabling realistic security testing, monitoring, and containment scenarios.

## Sanitization Note
To reduce risk, this repo uses **representative** IP ranges, hostnames, and identifiers.  
Architecture, workflows, and security controls are accurate, but specific values may be modified for privacy/security.

## Objectives
- Build a lab with **network segmentation** and **least privilege** routing between zones
- Implement **pfSense** routing + firewall policy between zones
- Maintain a reusable VM layout for SOC / blue-team and vuln-testing workflows
- Provide clear documentation + screenshots for reproducibility

## High-Level Architecture
**Zones**
- **WAN (vmbr0):** home uplink, used only by pfSense WAN
- **LAN1 Enterprise (vmbr1):** AD + endpoints + SIEM tooling (blue-team zone)
- **LAN2 Vulnerable (vmbr2):** intentionally vulnerable systems + scanners (red/vuln zone)

**Edge**
- **pfSense VM (FW-EDGE01):**
  - NIC1 → vmbr0 (WAN)
  - NIC2 → vmbr1 (LAN1 Enterprise)
  - NIC3 → vmbr2 (LAN2 Vulnerable)

> Diagram: see `diagrams/network-diagram.png`

## IP Plan (Example)
| Zone | Subnet | Gateway | Notes |
|------|--------|---------|------|
| WAN | DHCP/ISP | — | pfSense WAN only |
| LAN1 (Enterprise) | 10.10.10.0/24 | 10.10.10.1 | AD + endpoints + SIEM |
| LAN2 (Vulnerable) | 10.20.20.0/24 | 10.20.20.1 | Vuln targets + scanners |

Full details: `docs/ip-addressing.md`

## VM Inventory
**LAN1 – Enterprise / Blue Team**
- AD-DC01 (Windows Server) – Domain Services, DNS
- AD-WIN10, AD-WIN11 – Domain joined endpoints
- SIEM-SPLUNK01 – Splunk Free (optional)
- SEC-KALI01 – attacker simulation (optional)

**LAN2 – Vulnerable / Testing**
- VULN-METASPLOITABLE2
- VULN-DVWA / WebGoat
- VULN-WIN2019 (unpatched)
- VULN-OPENVAS (scanner)
- VULN-UBU-OLD (outdated Linux)

Full list: `docs/vm-inventory.md`

## Firewall Policy (What I enforced)
**Default stance:** deny-by-default between LANs; allow only what is required.

Example rules:
- LAN1 → WAN: allow (updates, package installs)
- LAN2 → WAN: restricted allow (optional) or deny
- LAN2 → LAN1: deny (block lateral movement by default)
- LAN1 → LAN2: allow limited (admin/testing only; e.g., OpenVAS → targets)

Details + screenshots:
- `docs/firewall-policy.md`
- `screenshots/pfsense-firewall-rules.png`

## Evidence (Screenshots)
- Proxmox bridges: `screenshots/proxmox-bridges.png`
- pfSense interfaces: `screenshots/pfsense-interfaces.png`
- Firewall rules: `screenshots/pfsense-firewall-rules.png`
- NAT: `screenshots/pfsense-nat.png`
- VM list: `screenshots/vm-list.png`

## Repro Steps (High Level)
1. Create Proxmox bridges: vmbr0 (WAN), vmbr1 (LAN1), vmbr2 (LAN2)
2. Deploy pfSense VM with 3 NICs mapped to those bridges
3. Configure pfSense interfaces + DHCP scopes for LAN1/LAN2
4. Apply firewall rules to enforce segmentation
5. Deploy VMs into LAN1 or LAN2 per inventory

Walkthrough docs:
- `docs/proxmox-bridges.md`
- `docs/pfsense-config.md`
- `docs/build-notes.md`

## What’s Next
This repo is the foundation for:
- Centralized telemetry pipelines (WEF/Sysmon → Wazuh/Elastic/Splunk)
- Attack simulation + investigation case files using segmented zones

