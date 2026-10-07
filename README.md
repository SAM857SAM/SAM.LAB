# SAM.LAB — Enterprise Home Lab

A small-enterprise network I built at home to practice real IT, networking, and security work.
Built by **Samual A. Michael** — CSCI student, studying for CCNA and CompTIA Security+.

> Sensitive values (public IP, external hostnames, VPN usernames, passwords, keys) are replaced with `[CONFIDENTIAL]`.

---

## Network at a glance

| Segment | Network | Gateway | Purpose |
|---|---|---|---|
| LAN | 10.0.0.0/24 | 10.0.0.1 (pfSense) | AD, servers, clients |
| DMZ | 10.0.10.0/24 | 10.0.10.1 (pfSense) | Isolated web server |
| VPN pool | 10.0.20.0/24 | pfSense IPsec | Remote admin clients |
| WAN | `[CONFIDENTIAL PUBLIC IP]` | ISP (IP Passthrough) | Internet |

## Systems

| Host | IP | Role | Status |
|---|---|---|---|
| FW01 | 10.0.0.1 | pfSense 2.9.0 — firewall, NAT, DMZ, IPsec VPN | ✅ Built |
| DC01 | 10.0.0.4 | AD DS, DNS, Group Policy (`sam.lab`) | ✅ Built |
| DHCP server | VERIFY | Windows DHCP, scope .100–.200 | ✅ Built |
| WAC01 | 10.0.0.7 | Windows Admin Center | ✅ Built |
| GUAC01 | 10.0.0.11 | Apache Guacamole 1.6.0 (via Cloudflare Tunnel) | ✅ Built |
| DEPLOYWIN | 10.0.0.15 | WDS / MDT / PXE | 🟠 In progress |
| WEB01 | 10.0.10.10 | Windows Server 2022 in DMZ | 🟠 In progress |
| CM01 | 10.0.0.6 | SCCM | ⚪ Planned |
| SOC stack | — | Wazuh, Sysmon, Zabbix, Kali | ⚪ Planned |

## Guides (step by step, with errors and fixes)

| # | Area | Guide |
|---|---|---|
| 01 | Networking | [pfSense firewall install and setup](docs/01-networking/pfsense.md) |
| 01 | Networking | IPsec VPN — *coming soon* |
| 02 | Active Directory | DC01, OUs, groups, GPO — *coming soon* |
| 03 | Deployment | WDS / PXE — *coming soon* |
| 04 | Servers | Hyper-V, WAC, Guacamole — *coming soon* |

## 📋 Pending roadmap (not built yet)
- [ ] **DMZ + WEB01 web server:** IIS, sample site, HTTPS, DMZ→LAN deny, LAN/VPN-only RDP
- [ ] DEPLOYWIN: fix PXE BCD error 0xc000000f
- [ ] GPO: fix wallpaper policy (Denied Security filtering)
- [ ] Confirm DHCP server IP
- [ ] LAN move to 10.10.0.0/24
- [ ] CM01 / SCCM
- [ ] 50-policy AD security baseline (passwords, lockout, BitLocker, USB, auditing)
- [ ] SOC: Sysmon, Wazuh, ELK, TheHive, Zabbix
- [ ] Security tools: Kali, OpenVAS, Suricata, Autopsy
- [ ] RODC, FILE01 shares, ownCloud, NetBox

## Skills shown

Windows Server 2022 · Hyper-V · Active Directory · DNS · DHCP · Group Policy · pfSense · NAT · DMZ segmentation · IKEv2/IPsec VPN · Zero-trust remote access (Cloudflare Tunnel) · Troubleshooting & documentation

## Certifications

| Certification | Status |
|---|---|
| Cisco CCST (VERIFY which) | ✅ Earned — badge link VERIFY |
| CCNA 200-301 | 📘 In progress |
| CompTIA Security+ | 🎯 Planned |
