# SAM.LAB — Enterprise Home Lab

A small-enterprise network I built at home to practice real IT, networking, and security work.
Built by **Samual A. Michael** — CSCI student, studying for CCNA and CompTIA Security+.

> Sensitive values (public IP, external hostnames, VPN usernames, passwords, keys, MAC addresses) are replaced with `[CONFIDENTIAL]` or blurred.

---

## Network at a glance

| Segment | Network | Gateway | Purpose |
|---|---|---|---|
| LAN | 10.0.0.0/24 | 10.0.0.1 (pfSense) | AD, servers, clients |
| DMZ | 10.0.10.0/24 | 10.0.10.1 (pfSense) | Isolated web server |
| VPN pool | 10.0.20.0/24 | pfSense IPsec | Remote admin clients |
| WAN | `[CONFIDENTIAL PUBLIC IP]` | ISP (IP Passthrough) | Internet |

```
            Internet
               │
        ┌──────┴──────┐        Cloudflare Tunnel ──► GUAC01 (no open ports)
        │ FW01 pfSense│◄────── IPsec VPN (IKEv2) ── remote laptop
        └──┬───────┬──┘
     LAN   │       │  DMZ (blocked from LAN)
 10.0.0.0/24      10.0.10.0/24 ── WEB01
           │
  Netgear AP (switch mode) ── Hyper-V host
                                ├─ DC01   (AD, DNS)
                                ├─ DHCP   (Windows DHCP)
                                ├─ DEPLOYWIN (WDS/PXE)
                                ├─ WAC01  (Windows Admin Center)
                                ├─ GUAC01 (Guacamole)
                                └─ DEMO01 / CLIENT01-02 (Windows 11)
```

## Systems

| Host | IP | Role | Status |
|---|---|---|---|
| FW01 | 10.0.0.1 | pfSense 2.9.0 — firewall, NAT, DMZ, IPsec VPN | ✅ Built |
| DC01 | 10.0.0.4 | AD DS, DNS, Group Policy (`sam.lab`) | ✅ Built |
| DHCP | VERIFY | Windows DHCP, scope .100–.200 | ✅ Built |
| WAC01 | 10.0.0.7 | Windows Admin Center 2606 | ✅ Built |
| DEMO01 | 10.0.0.9 | Windows 11 domain client | ✅ Built |
| GUAC01 | 10.0.0.11 | Apache Guacamole 1.6.0 + Cloudflare Tunnel | ✅ Built |
| DEPLOYWIN | 10.0.0.15 | WDS / PXE | 🟠 In progress |
| WEB01 | 10.0.10.10 | Windows Server 2022 in DMZ | 🟠 In progress |
| CM01 | 10.0.0.6 | SCCM | ⚪ Planned |
| SOC stack | — | Wazuh, Sysmon, Zabbix, Kali | ⚪ Planned |

## Guides (step by step, with real errors and fixes)

| Area | Guide | Status |
|---|---|---|
| Networking | [pfSense firewall](docs/01-networking/pfsense.md) | ✅ |
| Networking | [IPsec VPN (IKEv2)](docs/01-networking/ipsec-vpn.md) | ✅ |
| Networking | [Windows DHCP](docs/01-networking/dhcp.md) | ✅ |
| Networking | [DMZ + WEB01 web server](docs/01-networking/dmz-web01.md) | 🟠 |
| Active Directory | [DC01, OUs, groups, domain join](docs/02-active-directory/dc01-active-directory.md) | ✅ |
| Active Directory | [Group Policy](docs/02-active-directory/group-policy.md) | 🟠 |
| Deployment | [WDS / PXE](docs/03-deployment/wds-pxe.md) | 🟠 |
| Servers | [Hyper-V host](docs/04-servers/hyper-v.md) | ✅ |
| Servers | [Windows Admin Center + RSAT](docs/04-servers/windows-admin-center.md) | ✅ |
| Servers | [Apache Guacamole](docs/04-servers/guacamole.md) | ✅ |
| Servers | [Cloudflare Tunnel + Access](docs/04-servers/cloudflare-tunnel.md) | ✅ |

Every guide ends with a **📋 Pending** checklist of what's not built yet.

## 📋 Pending roadmap (not built yet)
- [ ] **DMZ + WEB01 web server:** IIS, sample site, HTTPS, DMZ→LAN deny-all, LAN/VPN-only RDP
- [ ] DEPLOYWIN: fix PXE BCD error 0xc000000f
- [ ] GPO: fix wallpaper policy (Denied Security filtering)
- [ ] Confirm DHCP server IP (conflict with AP at 10.0.0.2)
- [ ] LAN move to 10.10.0.0/24
- [ ] CM01 / SCCM
- [ ] 50-policy AD security baseline (passwords, lockout, BitLocker, USB, auditing)
- [ ] SOC: Sysmon, Wazuh, ELK, TheHive, Zabbix
- [ ] Security tools: Kali, OpenVAS, Suricata, Autopsy
- [ ] RODC, FILE01 shares, ownCloud, NetBox

## Skills shown

Windows Server 2022 · Hyper-V · Active Directory · DNS · DHCP · Group Policy · WDS/PXE · pfSense · NAT · DMZ segmentation · IKEv2/IPsec VPN · PKI (internal CA) · Zero-trust remote access (Cloudflare Tunnel + Access) · Linux (Ubuntu, LVM, building from source) · Tomcat / MariaDB · Troubleshooting & documentation

## Certifications

| Certification | Status |
|---|---|
| Cisco CCST (VERIFY which) | ✅ Earned — badge link VERIFY |
| CCNA 200-301 | 📘 In progress |
| CompTIA Security+ | 🎯 Planned |
