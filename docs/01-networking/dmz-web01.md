# DMZ + WEB01 Web Server — Plan and Progress

**Goal:** Host a website on a server that is isolated from the internal LAN. If the web server is hacked, the attacker still can't reach AD or the file servers.

| Item | Value |
|---|---|
| DMZ network | 10.0.10.0/24 |
| DMZ gateway | pfSense `re0` 10.0.10.1 |
| WEB01 | Physical PC, Windows Server 2022, 10.0.10.10 |
| Status | 🟠 In progress |

```
Internet ──► pfSense WAN ──► (443 only, later) ──► WEB01 10.0.10.10   [DMZ]
                     │
                     └──► LAN 10.0.0.0/24 (AD, DNS, servers)   ✖ DMZ cannot reach LAN
```

---

## ✅ Done
1. pfSense `OPT1 (re0)` enabled as **DMZ**, static `10.0.10.1/24` — see [pfSense guide](pfsense.md).
2. WEB01 installed with Windows Server 2022 and cabled straight to the DMZ port.
3. WEB01 IP `10.0.10.10/24`, gateway `10.0.10.1`.
4. Firewall rules (DMZ tab):
   - **Block** WEB01 → `10.0.0.0/24` (*BLOCK WEB01 TO SAM.LAB LAN*)
   - **Pass** ICMP WEB01 → DMZ address (testing)

![DMZ rules](../../screenshots/pfsense/07-dmz-rules.jpg)

Check:
```cmd
ping 10.0.10.1      :: works
ping 10.0.0.4       :: must FAIL (blocked)
```

---

## 📋 Pending — build steps in order
- [ ] **1. Lock down DMZ rules (best practice order, top to bottom):**
  - Pass DMZ net → DMZ address, DNS (UDP 53) only if WEB01 uses pfSense DNS
  - Pass DMZ net → any, TCP 80/443 (Windows Update / downloads)
  - **Block DMZ net → RFC1918** (all private networks: LAN, VPN, home)
  - Block everything else (default deny)
- [ ] **2. Management from LAN/VPN only** — LAN tab rule: `GG_IT` admin PC / VPN pool `10.0.20.0/24` → `10.0.10.10` TCP 3389 (and 5985 for WAC). Never from WAN.
- [ ] **3. Install IIS** on WEB01:
  ```powershell
  Install-WindowsFeature Web-Server -IncludeManagementTools
  ```
  Test from LAN: `http://10.0.10.10` shows the IIS page.
- [ ] **4. Deploy a sample site** (e.g. a portfolio page) to `C:\inetpub\wwwroot`.
- [ ] **5. Harden WEB01:** keep it **workgroup** (not domain-joined), local admin with strong password, Windows Firewall on, remove default IIS page, turn off unused features, auto updates.
- [ ] **6. HTTPS certificate** — Let's Encrypt with win-acme (needs public DNS) or Cloudflare origin cert.
- [ ] **7. Publish (only if needed):** pfSense **NAT > Port Forward** WAN TCP 443 → 10.0.10.10:443. Or better: Cloudflare Tunnel from WEB01 so no port is opened.
- [ ] **8. Logging:** send IIS + Windows logs to the SIEM (Wazuh) when built.
- [ ] **9. Test from outside:** cellular phone → `https://[CONFIDENTIAL WEBSITE]`.
- [ ] **10. Scan it:** Nmap / OpenVAS from Kali — only 443 should be open.

## Why this design
- **DMZ ≠ LAN.** Public-facing servers get attacked most. Blocking DMZ → LAN stops an attacker from moving to the domain controller.
- **Workgroup, not domain.** A hacked domain-joined web server can leak domain credentials.
- **Admin access only from inside.** RDP is never exposed to the internet.
