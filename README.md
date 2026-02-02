# P1-1: Proxmox Segmentation Lab 

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

## Network Diagram
<img src="https://github.com/kvntynito/P1-1-proxmox-segmentation-lab/blob/main/diagrams/repo1-network-diagram.png?raw=true" alt="Proxmox Segmentation Lab Network Diagram" width="950">

**Design intent:** FW-EDGE01 (pfSense) provides WAN access and enforces segmentation between **LAN1 (enterprise/blue-team)** and **LAN2 (vulnerable/testing)**.  

**Default stance:** deny-by-default from **LAN2 → LAN1** to limit lateral movement; allow only scenario-driven exceptions.

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

## IP Plan (Example)
| Zone | Subnet | Gateway | Notes |
|------|--------|---------|------|
| WAN | DHCP/ISP | — | pfSense WAN only |
| LAN1 (Enterprise) | 10.10.10.0/24 | 10.10.10.1 | AD + endpoints + SIEM |
| LAN2 (Vulnerable) | 10.20.20.0/24 | 10.20.20.1 | Vuln targets + scanners |

Full details: `docs/ip-addressing.md`

## VM Inventory

### LAN1 — Enterprise / Blue Team (Production-like zone)
- **AD-DC01 (Windows Server)** — Domain Services, DNS  
  **Why it’s included:** I wanted a realistic enterprise identity backbone for domain authentication, DNS, and policy-driven testing.

- **AD-WIN10 / AD-WIN11 (Domain-joined endpoints)** — User workstations  
  **Why it’s included:** These are my “real user” machines to generate authentic endpoint activity for monitoring and investigations (logons, process execution, PowerShell, Sysmon/WEF).

- **AD-FS01 (File Server)** — SMB shares, NTFS permissions *(optional but recommended)*  
  **Why it’s included:** File servers create the kind of everyday east-west traffic enterprises actually have, and they’re perfect for access auditing scenarios (SMB auth, share access, permission changes).

- **SIEM-SPLUNK01 (Splunk Free)** *(optional)*  
  **Why it’s included:** I use this to centralize logs and practice investigation workflows (searching, filtering, pivoting, building a repeatable triage process).

- **SEC-KALI01 (Attacker Simulation)** *(optional)*  
  **Why it’s included:** This gives me a controlled way to generate adversary-like activity so I can validate segmentation rules and detection coverage without touching anything outside the lab.

---

### LAN2 — Vulnerable / Testing (Intentionally insecure zone)
- **VULN-METASPLOITABLE2**  
  **Why it’s included:** It’s a consistent, repeatable vulnerable target that lets me test segmentation controls and generate known-bad telemetry.

- **VULN-DVWA / WebGoat**  
  **Why it’s included:** These give me web-app attack practice targets so I can generate realistic web attack signals and see what shows up in logs.

- **VULN-WIN2019 (Unpatched)**  
  **Why it’s included:** I wanted a Windows target that supports exploitation/lateral movement practice and lets me focus on Windows event/Sysmon investigation patterns.

- **VULN-OPENVAS (Scanner)**  
  **Why it’s included:** This supports a vulnerability management workflow—scan, review findings, remediate, then rescan to confirm.

- **VULN-UBU-OLD (Outdated Linux)**  
  **Why it’s included:** I use an intentionally outdated Linux box to simulate legacy risk and test hardening gaps and detection visibility.

### How I use these zones
I treat **LAN1** as the “production-like” environment where identity, endpoints, and monitoring live, and **LAN2** as a controlled risk zone for vulnerable targets and scanning. Segmentation is enforced at pfSense with a deny-by-default posture between LANs, and I only allow specific flows when a lab scenario requires it (e.g., scanner-to-target traffic or admin/testing access).

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
