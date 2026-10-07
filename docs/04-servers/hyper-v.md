# Hyper-V Host — Step by Step

**Goal:** One physical server that runs all the lab VMs.
**Status:** ✅ Built
**Host:** WIN-LU49EQ719FL · Windows Server 2022 · i3-10105F · 24 GB RAM · single C: 476 GB · **workgroup** (not domain-joined)
**IP:** 10.0.0.103 (DHCP reservation on DHCP01; target static 10.0.0.3) · RDP allowed **only from the VPN pool**

| Virtual switch | Type | Use |
|---|---|---|
| LAB-LAN-EX | External | VMs on the real lab LAN (10.0.0.0/24) |
| LAB-LAN | Internal | Spare isolated network |

| VM | Gen | RAM | Role |
|---|---|---|---|
| DC01 | 2 | 4096 MB | AD DS, DNS |
| DHCP (DHCP01) | 2 | dynamic | Windows DHCP (10.0.0.20) |
| FOR01 | 2 | 6 GB fixed, 4 vCPU | Forensics — Autopsy 4.23.1 (10.0.0.12) |
| DEPLOYWIN | 2 | 4096 MB | WDS / PXE |
| DEMO01 | 2 | 6000 MB | Windows 11 test client |
| CLIENT01 / CLIENT02 | 2 | 4096 MB | Windows 11 clients |
| Guac01 | 2 | 2048 MB, 2 vCPU | Ubuntu, Apache Guacamole |

---

## Step 0 — Find and reserve the host IP
> ⚠ **Error:** `ping 10.0.0.3` → *Destination host unreachable*. The host had moved to a DHCP address.

![Host unreachable](../../screenshots/autopsy/err-host-unreachable.jpg)

**Fix:** Find it in the DHCP leases on DHCP01, then reserve it so it never changes:
```powershell
Get-DhcpServerv4Lease -ScopeId 10.0.0.0 | Select IPAddress, ClientId, HostName
Add-DhcpServerv4Reservation -ScopeId 10.0.0.0 -IPAddress 10.0.0.103 -ClientId "[HOST MAC]" -Name "WIN-LU49EQ719FL"
```

## Step 0b — Remote PowerShell to a workgroup host
> ⚠ **Error:** `Enter-PSSession` → *Kerberos … cannot find the computer*. The host isn't in AD/DNS.

**Fix:** Trust only the host's IP and use its local admin:
```powershell
Set-Item WSMan:\localhost\Client\TrustedHosts -Value "10.0.0.103" -Concatenate -Force
Enter-PSSession -ComputerName 10.0.0.103 -Credential WIN-LU49EQ719FL\Administrator
```

![TrustedHosts](../../screenshots/autopsy/01-trustedhosts-pssession.jpg)

> ⚠ **Error:** `Stop-VM CLIENT01` → *machine is locked and cannot be shut down without the force option*.
> **Fix:** `Stop-VM -Name CLIENT01 -Force` (clean shutdown). Avoid `-TurnOff`.

![Stop-VM](../../screenshots/autopsy/err-stop-vm.jpg)

## Step 1 — Install Hyper-V
```powershell
Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart
```

## Step 2 — Create the virtual switches
1. **Hyper-V Manager > Virtual Switch Manager**.
2. New **External** switch `LAB-LAN-EX`, bound to the physical NIC, check **Allow management OS to share**.
3. New **Internal** switch `LAB-LAN`.

Check:
```powershell
Get-VMSwitch
```

## Step 3 — Create a VM (example: DEPLOYWIN)
1. **Action > New > Virtual Machine**.
2. Name `DEPLOYWIN`, **Generation 2**, Memory `4096 MB`, Network `LAB-LAN-EX`.
3. New virtual hard disk, attach the install ISO, **Finish**.

![New VM](../../screenshots/hyper-v/01-new-vm-gen2.jpg)

## Step 4 — Connect the VM to the lab network
1. **VM Settings > Network Adapter > Virtual switch** = `LAB-LAN-EX`.

![vSwitch](../../screenshots/hyper-v/02-vswitch-lab-lan-ex.jpg)

> ⚠ **Error:** VM got a 169.254.x.x (APIPA) address and could not reach the DC.
> **Fix:** It was on `LAB-LAN` (internal). Switched it to `LAB-LAN-EX`.

## Step 5 — Windows 11 VMs need TPM
> ⚠ **Error:** *"This PC doesn't currently meet Windows 11 system requirements — The PC must support TPM 2.0."*

![TPM error](../../screenshots/hyper-v/err-tpm.jpg)

**Fix:** Turn the VM off, then **Settings > Security**: check **Enable Secure Boot** (template *Microsoft Windows*) and **Enable Trusted Platform Module**.

![Enable TPM](../../screenshots/hyper-v/04-enable-tpm.jpg)

## Step 6 — Boot order
> ⚠ **Error:** *"Virtual Machine Boot Summary — The boot loader failed / A boot image was not found."*

![Boot failed](../../screenshots/hyper-v/err-boot-failed.jpg)

**Fix:** **Settings > Firmware** → move **DVD Drive** to the top, then start the VM and **press a key** quickly when it says *Press any key to boot from CD or DVD*.

---

## ✅ Check it worked
![VM list](../../screenshots/hyper-v/03-vm-list.jpg)

```powershell
Get-VM | Select Name, State, MemoryAssigned, Generation
Get-VMNetworkAdapter -VMName * | Select VMName, SwitchName
```

## 📋 Pending (not built yet)
- [ ] Set Hyper-V host to static IP 10.0.0.3 (now DHCP reservation 10.0.0.103)
- [ ] VM checkpoints before big changes (naming standard)
- [ ] Add a separate data SSD — all VMs share the host C: drive (slow)
- [ ] VM backups (Windows Server Backup or Veeam Community)
- [ ] Hyper-V Replica or export schedule
- [ ] New VMs: CM01 (SCCM), SIEM01 (Wazuh), MON01 (Zabbix), KALI01
- [ ] Separate VLAN / vSwitch for Kali test segment
- [ ] Remove Tailscale now that IPsec VPN works
- [ ] Replace evaluation license
