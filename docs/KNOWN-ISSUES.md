# Known issues

Verified defects in the current release (`2026.11.13-v2`), found by unpacking
the published image and by booting it.

## 1. No autologin (open)

LightDM still shows a login screen. `/etc/lightdm/lightdm.conf` has every
autologin key commented out and `lightdm.conf.d/` is empty, so the desktop is
not reached without typing credentials.

- Workaround: log in as `user` / `live`.
- Fix: ship `lightdm.conf.d/50-zaod-autologin.conf` with
  `autologin-user=user`, `autologin-user-timeout=0` and
  `autologin-session=xfce`.

## 2. "Install Zaod OS to disk" does nothing useful (open)

The GRUB menu entry boots `/install/gtk/vmlinuz`, but the installer stage is
disabled in the recipe, so that file is not in the image and the entry fails.

- Workaround: run live only.
- Fix: either add the installer stage to the build or remove the menu entry.

## 3. dpkg database is incomplete (open, v2 only)

Inside the v2 live system `dpkg -l` lists only 69 packages, while v1 listed
1262. Both `status` and `status-old` show the short list, although XFCE,
Firefox and the rest are installed and running. That makes `apt` behave as if
almost nothing is installed.

- Workaround: avoid `apt` inside the live session; it is a read-mostly desktop.
- Fix: stop the build cleanup step from rewriting `/var/lib/dpkg/status`.

## 4. LibreOffice ships although it is not part of the goal (open, 231 MB)

`libreoffice-core`, `common`, `writer`, `math` and the icon style add up to
231 MB of the 3.62 GB installed system. Only Writer and Math are present; Calc
and Impress are not, so the suite is incomplete anyway.

- Fix: drop the `libreoffice*` packages from the package list.
- This is the single easiest weight reduction for 8 GB machines.

## 5. No shim: Secure Boot must be off (by design)

The boot path uses unsigned GRUB EFI binaries. Machines with Secure Boot
enforced will refuse to boot the stick until it is disabled in firmware.

## Fixed since the first release

| Issue | Status |
|---|---|
| `/etc/hostname` contained the build machine name (`codespaces-7c9b13`) | Fixed in v2, now `zaod` |
| Boot command line had no `username=`/`hostname=` parameters | Fixed in v2 |
| No UEFI boot path | Fixed: hybrid ISO with `efi.img` |

## Hardware notes

- 2 GB RAM works but Firefox plus XFCE is tight; 4 GB is comfortable.
- 8 GB storage is enough for the live image, not for installed use plus data.
- Free Fire and Craftland Studio are **not** part of Zaod OS and cannot be: Free
  Fire is an Android-only, closed Garena app and redistributing its APK is a
  licence violation; Craftland Studio is a Windows-only editor. Running an
  emulator gets anti-cheat bans. Zaod OS stays a browser desktop; use the phone
  for Free Fire and the browser for everything web.
