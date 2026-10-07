# Troubleshooting — PXE Boot "\Boot\BCD 0xc000000f"

**Status:** 🔴 Open · **Server:** DEPLOYWIN 10.0.0.15 (WDS)

## Problem
PXE clients get an address, load **Windows Boot Manager (Server IP: 10.0.0.15)**, then stop:

```
File:   \Boot\BCD
Status: 0xc000000f
Info:   The Boot Configuration Data for your PC is missing or contains errors.
```

![BCD error](../../screenshots/wds/err-bcd.jpg)

A Hyper-V test VM shows *"A boot image was not found."*

## Environment
| Item | Value |
|---|---|
| WDS | AD-integrated, native mode, `C:\RemoteInstall` |
| Boot image | Microsoft Windows Setup (amd64) — stock `boot.wim` from the ISO |
| PXE response | Respond to all, delay 0 |
| WDS DHCP tab | Both boxes unchecked (DHCP is on DHCP01) |
| DHCP01 scope | Options **066 / 067 set** |
| Listening | UDP 67, 69, 4011 on 10.0.0.15 |

![WDS listening](../../screenshots/wds/09-wds-listening.jpg)

## Already tried
| Attempt | Result |
|---|---|
| Ping DEPLOYWIN from clients | Works |
| Fixed duplicate IP (10.0.0.10 → 10.0.0.15) | Reachability fixed |
| Windows Firewall off | No change (turn it back on) |
| PXE response known-only → respond to all | Clients now reach Boot Manager |
| Added boot image | Still BCD 0xc000000f |

## Likely causes (ranked)
| # | Cause | Why it fits | Test |
|---|---|---|---|
| 1 | DHCP options 066/067 send clients to the wrong boot file | Error comes right after Boot Manager loads; not needed when WDS is on the same subnet | Remove 066/067 |
| 2 | Stock Windows 11 `boot.wim` | Microsoft blocks install-media boot images in WDS for Windows 11 | Build a WinPE boot image with the Windows ADK |
| 3 | UEFI vs BIOS boot file mismatch | 067 may point to a BIOS file | UEFI needs `boot\x64\wdsmgfw.efi` |

## Fix plan
```powershell
# 1 — on DHCP01
Get-DhcpServerv4OptionValue -ScopeId 10.0.0.0 -All
Remove-DhcpServerv4OptionValue -ScopeId 10.0.0.0 -OptionId 66,67

# 2 — on DEPLOYWIN
Get-WdsBootImage
Get-WinEvent -LogName 'Microsoft-Windows-Deployment-Services-Diagnostics/Admin' -MaxEvents 30

# 4 — turn firewalls back on
Set-NetFirewallProfile -All -Enabled True
Enable-NetFirewallRule -DisplayGroup "Windows Deployment Services"
```
3. Re-test with a Gen 2 VM, then a physical PC. Write down firmware mode, Secure Boot state, and the exact message.

## Done when
- [ ] A client loads WinPE / Windows Setup over PXE
- [ ] Firewalls are back on
- [ ] Fix written into the [WDS guide](../03-deployment/wds-pxe.md)

## Lessons
- Don't use DHCP 066/067 with WDS on the same subnet.
- MDT is retired and WDS boot-image support is limited for Windows 11 → long term, use Configuration Manager OSD (CM01).
