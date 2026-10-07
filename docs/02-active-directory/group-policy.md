# Group Policy (GPO) — Step by Step

**Goal:** Lock down standard users and push a company wallpaper, managed from DC01.
**Status:** 🟠 Part A working · Part B open

| GPO | Linked to | Status |
|---|---|---|
| GPO-General-User-Restrictions | SAMLAB > Users > General User | ✅ Working |
| GG_wallpaper1 (wallpaper) | SAMLAB > Users | 🔴 Not applying — see [troubleshooting](../07-troubleshooting/wallpaper-gpo.md) |

---

## Part A — Restrict standard users

### Step 1 — Create and link the GPO
1. **Group Policy Management** > right-click **General User** OU > **Create a GPO in this domain, and Link it here**.
2. Name `GPO-General-User-Restrictions`.

![New GPO](../../screenshots/gpo/01-new-gpo.jpg)

![GPO linked](../../screenshots/gpo/02-gpo-linked.jpg)

### Step 2 — Settings (User Configuration > Policies > Administrative Templates)
| Path | Setting | Value |
|---|---|---|
| System | Prevent access to the command prompt | Enabled |
| System | Prevent access to registry editing tools | Enabled |
| Control Panel | Prohibit access to Control Panel and PC settings | Enabled |
| System > Ctrl+Alt+Del Options | Remove Task Manager / Lock / Logoff / Change Password | Enabled |

![cmd and regedit](../../screenshots/gpo/03-cmd-regedit.jpg)

![Control Panel](../../screenshots/gpo/04-control-panel.jpg)

![Ctrl+Alt+Del](../../screenshots/gpo/05-ctrl-alt-del.jpg)

### Step 3 — Test on a client
Sign in as a General User, then:
```powershell
gpupdate /force
whoami
gpresult /r
```

![gpupdate](../../screenshots/gpo/11-gpupdate-whoami.jpg)

![gpresult applied](../../screenshots/gpo/12-gpresult-applied.jpg)

Open **cmd** → *"The command prompt has been disabled by your administrator."* ✅

![cmd disabled](../../screenshots/gpo/13-cmd-disabled.jpg)

---

## Part B — Company wallpaper

### Step 1 — Share the wallpaper
1. On DC01 create `C:\SAMLAB\Wallpapers`, put `SAMLAB.png` in it.
2. **Properties > Sharing > Advanced Sharing** → share name `Wallpapers`.
3. **Permissions**: Everyone = **Read** only. NTFS: Users = Read.

![Share](../../screenshots/gpo/06-share-wallpapers.jpg)

![Share permissions](../../screenshots/gpo/07-share-permissions.jpg)

Check from a client: open `\\dc01\Wallpapers`.

![Share from client](../../screenshots/gpo/14-share-from-client.jpg)

### Step 2 — Wallpaper GPO
1. User Configuration > Policies > Administrative Templates > Desktop > Desktop > **Desktop Wallpaper** = Enabled.
2. Wallpaper name **`\\DC01\Wallpapers\SAMLAB.png`** (must match the share name), style **Fill**.
3. Also enable **Prevent changing desktop background** (Control Panel > Personalization).

![Wallpaper setting](../../screenshots/gpo/08-wallpaper-setting.jpg)

> ⚠ **Mistake found:** the GPO was saved with `\\DC01\SAMLAB\Wallpapers\SAMLAB.png`, but the share is `\\DC01\Wallpapers`. Fix the path to `\\DC01\Wallpapers\SAMLAB.png`.

### Step 3 — Target one group (GG_wallpaper1)
1. Add user `samlab01` to group `GG_wallpaper1`.
2. GPO **Delegation / Security Filtering** set to `GG_wallpaper1`.

![Delegation](../../screenshots/gpo/09-wallpaper-delegation.jpg)

![samlab01 member of](../../screenshots/gpo/10-samlab01-member-of.jpg)

> ⚠ **Error:** GPMC — *"No mapping between account names and security IDs was done."*
> **Fix:** The group name was typed wrong or not created yet. Create the group first, then add it with **Check Names**.

![No mapping](../../screenshots/gpo/err-no-mapping.jpg)

> ⚠ **Error (still open):** `gpresult /r` as samlab01 → **GG_wallpaper1 — Filtering: Denied (Security)**.

![Denied security](../../screenshots/gpo/err-denied-security.jpg)

**Look closely:** the user's group list does **not** show `GG_wallpaper1`, and on the Delegation tab the group only has **Read** (no Apply). Fix plan (in order):
1. **Sign out / sign in** as samlab01 — group changes only reach the logon token at sign-in.
2. Give the group **Apply**: Scope > Security Filtering > add `GG_wallpaper1` (or `Set-GPPermission -Name GG_wallpaper1 -TargetName GG_wallpaper1 -TargetType Group -PermissionLevel GpoApply`).
3. Keep **Read** for Authenticated Users or Domain Computers (required since MS16-072).
4. Fix the UNC path (above).

Full plan: [Wallpaper GPO troubleshooting](../07-troubleshooting/wallpaper-gpo.md).

---

## 📋 Pending (not built yet)
- [ ] Fix GG_wallpaper1 (fresh logon, GpoApply, UNC path) and confirm with `gpresult /h report.html`
- [ ] Decide one wallpaper GPO name (`GPO-SAMLAB-Wallpaper` vs `SAMLAB Rule` vs `GG_wallpaper1`) and remove duplicates
- [ ] Password policy: 14 chars, complexity, history 24
- [ ] Account lockout: 5 attempts, 15 min
- [ ] Screen lock after 15 min, logon banner
- [ ] BitLocker GPO (FOR01 already encrypting by hand)
- [ ] USB / removable storage block
- [ ] Advanced audit policy + PowerShell logging (for SIEM)
- [ ] Windows Firewall + Defender GPOs
- [ ] Windows LAPS GPO
- [ ] AppLocker (audit first)
