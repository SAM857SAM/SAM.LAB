# Troubleshooting — Wallpaper GPO "Denied (Security)"

**Status:** 🔴 Open · **GPO:** GG_wallpaper1 · **Test user:** samlab01

## Problem
The wallpaper GPO should apply to members of the `GG_wallpaper1` group. For samlab01, `gpresult /r` says:

```
GG_wallpaper1
    Filtering:  Denied (Security)
```

![Denied security](../../screenshots/gpo/err-denied-security.jpg)

## Environment
| Item | Value |
|---|---|
| GPO | GG_wallpaper1, linked to SAMLAB\Users |
| Setting | Desktop Wallpaper → `\\DC01\SAMLAB\Wallpapers\SAMLAB.png` |
| Share | `\\DC01\Wallpapers` (C:\SAMLAB\Wallpapers), Everyone Read |
| Delegation | GG_wallpaper1 = **Read** only; Authenticated Users = Read |
| User | samlab01 — member of Domain Users, GG_Operations, GG_wallpaper1 |

![Delegation](../../screenshots/gpo/09-wallpaper-delegation.jpg)

## Clues
1. In `gpresult`, the user's groups show **GG_Operations but not GG_wallpaper1** → the logon token is old.
2. The group has **Read** but not **Apply group policy**.
3. The GPO path (`\\DC01\SAMLAB\Wallpapers`) does **not** match the share (`\\DC01\Wallpapers`).

## Fix plan (one at a time, re-test after each)
```powershell
# On DC01 — check and fix permissions
Get-GPPermission -Name GG_wallpaper1 -All | Format-Table Trustee, Permission
Set-GPPermission -Name GG_wallpaper1 -TargetName GG_wallpaper1 -TargetType Group -PermissionLevel GpoApply
```
1. Sign **out and back in** as samlab01 (or restart) so the token picks up the new group.
2. Give `GG_wallpaper1` **Apply** (command above). Keep Authenticated Users / Domain Computers on **Read** (MS16-072).
3. Change the wallpaper path to `\\DC01\Wallpapers\SAMLAB.png`.

```cmd
REM On the client, as samlab01
whoami /groups | findstr GG_
gpupdate /force
gpresult /r /scope user
dir \\DC01\Wallpapers\SAMLAB.png
```

## Done when
- [ ] `gpresult` lists GG_wallpaper1 under **Applied** for samlab01
- [ ] Wallpaper shows after sign-in; a user outside the group doesn't get it
- [ ] Fix written into the [Group Policy guide](../02-active-directory/group-policy.md)

## Lessons
- Group membership changes need a **new logon**.
- Security-filtered GPOs need **Apply** for the target group and **Read** for computers.
- Copy UNC paths from the share's properties instead of typing them.
