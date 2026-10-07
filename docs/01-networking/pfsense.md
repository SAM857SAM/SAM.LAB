# pfSense Firewall (FW01) — Step by Step

**Goal:** Put a pfSense firewall between the internet and the lab, with a separate DMZ.
**Version:** pfSense CE 2.9.0 · **Hardware:** i3 PC, Intel I350-T2 dual NIC + onboard Realtek

| Interface | NIC | Address |
|---|---|---|
| WAN | igb1 | `[CONFIDENTIAL PUBLIC IP]` (ISP IP Passthrough) |
| LAN | igb0 | 10.0.0.1/24 |
| DMZ | re0 | 10.0.10.1/24 |

---

## Step 1 — Make the install USB
1. Download the pfSense CE installer from Netgate.
2. Write it to a USB drive with **Rufus** or **balenaEtcher**.

## Step 2 — Boot the installer
1. Plug the USB into FW01 and open the BIOS boot menu.
2. Select the USB drive.

> ⚠ **Error:** The USB would not boot.
> **Fix:** Secure Boot was blocking it. In BIOS, **disable Secure Boot** but keep **UEFI** on.

> 📷 *Screenshot coming soon: pfSense installer boot screen*

## Step 3 — Install and assign interfaces
1. Accept defaults and install to the SSD.
2. At **Assign Interfaces**, say **No** to VLANs.
3. WAN = `igb1`, LAN = `igb0`, Optional 1 = `re0`.

> 💡 Tip: Unplug/replug a cable and watch which interface shows **up**. That tells you which port is which.

## Step 4 — Set the LAN IP
1. From the console menu, choose **2) Set interface(s) IP address**.
2. Select **LAN**, type `10.0.0.1`, mask `24`.
3. When asked to enable DHCP on LAN, choose **No** (Windows Server handles DHCP).

> ⚠ **Error:** The old home router already used 10.0.0.1, so two gateways fought.
> **Fix:** Put the Netgear RAX36 in **AP mode** at `10.0.0.2` with **DHCP off**. pfSense is now the only gateway.

## Step 5 — Setup wizard (WebGUI)
1. From a LAN PC, open `https://10.0.0.1`.
2. Accept the certificate warning (self-signed — this is normal).
3. Hostname `FW01`, Domain `sam.lab`, DNS `10.0.0.4`, Time zone `America/Chicago`.
4. Change the default admin password.

> 📷 *Screenshot coming soon: pfSense dashboard*

## Step 6 — Set up the DMZ
1. Go to **Interfaces > OPT1**, check **Enable**, rename to **DMZ**.
2. IPv4 Type **Static**, address `10.0.10.1/24`, then **Save > Apply**.
3. Go to **Firewall > Rules > DMZ** and add only the narrow rules you need (start with ping to `10.0.10.1`).
4. Never allow DMZ → LAN.

## Step 7 — WAN public IP
1. On the ISP gateway, turn on **IP Passthrough** to pfSense's WAN MAC.
2. Reboot pfSense. WAN should now show `[CONFIDENTIAL PUBLIC IP]`.

> ⚠ **Error:** VPN from outside hung forever.
> **Fix:** WAN had a private 192.168.1.x address behind the ISP's NAT. IP Passthrough gave pfSense the real public IP.

---

## ✅ Check it worked
On a LAN PC with **Wi-Fi turned off** (so traffic can't sneak around pfSense):
```cmd
ipconfig /all
ping 10.0.0.1
ping 8.8.8.8
tracert 8.8.8.8
```
- Gateway shows `10.0.0.1`.
- `tracert` first hop is `10.0.0.1`.

> ⚠ **Gotcha:** Ping worked even when pfSense was wrong — the PC was using Wi-Fi. Always disable Wi-Fi when testing.

## 📋 Pending (not built yet)
- [ ] Final DMZ → LAN **deny** rule set
- [ ] LAN/VPN → WEB01 management rule (RDP only from LAN/VPN)
- [ ] WEB01: install IIS and test a sample site
- [ ] WEB01: HTTPS certificate
- [ ] WAN 443 → WEB01 port forward (only if the site goes public)
- [ ] Suricata IDS/IPS on pfSense
- [ ] Firewall aliases and rule naming standard
- [ ] VLANs (802.1Q), Kali test segment
- [ ] LAN move to 10.10.0.0/24
- [ ] pfSense config backup schedule

## Security choices
- No RDP open to the internet.
- DMZ is isolated from LAN.
- pfSense DHCP off — one DHCP server only.
- Default password changed.
