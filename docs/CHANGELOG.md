# Changelog

## 2026.11.13-v2 (latest)

Fixes:

- `/etc/hostname` is now `zaod`. The v1 image leaked the CI build machine name
  `codespaces-7c9b13`.
- Boot command line now carries `username=user hostname=zaod`.
- Verified boot: the image reaches the graphical greeter in a QEMU boot test.

Known regressions:

- The dpkg status database lists only 69 packages (see KNOWN-ISSUES.md #3).

| File | Size | SHA-256 |
|---|---|---|
| `Zaod-OS-x86_64.iso` | 2,125,385,728 bytes | `676af3d0a285fc4e0baf2acb892cf5067ef541fa6fdb17e514036b6a8875c2e5` |

## 2026.11.13 (superseded)

First published ISO.

- Debian 13 (trixie), kernel 6.12.116, XFCE + LightDM, Firefox ESR, hybrid
  BIOS/UEFI boot, 1262 packages, 3.62 GB installed, xz squashfs.
- Branding: Plymouth animation, wallpaper, desktop launcher, onboarding,
  system monitor.

| File | Size | SHA-256 |
|---|---|---|
| `Zaod-OS-x86_64.iso` | 2,125,692,928 bytes | `8da968573b8f79c7611324abd32d2c2670a92df2d21d61c0d4f7638af6efc6d4` |
