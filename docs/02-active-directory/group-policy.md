# Group Policy (GPO) — Step by Step

**Goal:** Lock down standard users and push a company wallpaper, managed from DC01.

| GPO | Linked to | Status |
|---|---|---|
| GPO-General-User-Restrictions | SAMLAB > Users > General User | ✅ Working |
| GG_wallpaper1 (wallpaper) | SAMLAB > Users | 🔴 Not applying — see Pending |

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
2. Wallpaper name `\\DC01\Wallpapers\SAMLAB.png`, style **Fill**.
3. Also enable **Prevent changing desktop background** (Control Panel > Personalization).

![Wallpaper setting](../../screenshots/gpo/08-wallpaper-setting.jpg)

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

**Look closely:** the user's group list does **not** show `GG_wallpaper1`. Two likely causes:
1. The user's sign-in token is old. Group changes only apply after **sign out / sign in** (or a restart).
2. Since Microsoft update MS16-072, the **computer** account must be able to read the GPO. Removing **Authenticated Users** from Security Filtering breaks this. Fix: keep `GG_wallpaper1` in **Security Filtering** (Read + Apply), and on the **Delegation** tab add **Domain Computers** with **Read**.

Then sign out/in as samlab01 and run `gpresult /r` again.

---

## 📋 Pending (not built yet)
- [ ] Fix GG_wallpaper1 filtering (sign out/in + Domain Computers Read) and confirm with `gpresult /h report.html`
- [ ] Decide one wallpaper GPO name (`GPO-SAMLAB-Wallpaper` vs `SAMLAB Rule` vs `GG_wallpaper1`) and remove duplicates
- [ ] Password policy: 14 chars, complexity, history 24
- [ ] Account lockout: 5 attempts, 15 min
- [ ] Screen lock after 15 min, logon banner
- [ ] BitLocker GPO
- [ ] USB / removable storage block
- [ ] Advanced audit policy + PowerShell logging (for SIEM)
- [ ] Windows Firewall + Defender GPOs
- [ ] Windows LAPS GPO
- [ ] AppLocker (audit first)
