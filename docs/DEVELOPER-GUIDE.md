# Zaod OS developer guide

This guide documents how Zaod OS is put together, how an ISO is produced and
released, and where the source lives.

## Design goals

1. Boot a usable desktop on hardware as small as 2 GB RAM / 8 GB disk.
2. Ship a browser and a file manager, plus the drivers needed for real laptops
   (WiFi, Bluetooth, GPU, camera, touchpad).
3. Keep the image lean: no office suite, no app store, no games by default.
4. Branded, quiet boot: Plymouth animation, own wallpaper, own desktop
   launcher, first-run onboarding.
5. Reproducible from a recipe: every release is built from a committed
   live-build configuration, not hand-assembled.

## Architecture

| Layer | Choice |
|---|---|
| Base distribution | Debian 13 (trixie) |
| Kernel | 6.12.116 (Debian kernel) |
| Init / services | systemd, NetworkManager, Blueman, PipeWire |
| Desktop | XFCE 4 with LightDM, XFWM compositor off on weak GPUs |
| Browser | Firefox ESR |
| File manager | Thunar (+ Xarchiver) |
| Firmware | Debian `firmware-*` packages: Intel iwlwifi, Realtek rtl/rtw, Atheros, Broadcom, GPU firmware, uvc camera firmware |
| Feature flags | non-free + non-free-firmware enabled in the apt sources |
| Boot | Hybrid ISO: BIOS via isolinux/GRUB El Torito, UEFI via `efi.img` -> `efi/boot/bootx64.efi` |
| Root filesystem | xz-compressed squashfs, ~1.46 GB compressed, 3.62 GB installed |
| Compression | xz (slower build, smaller download; the right trade-off for weak internet) |

No shim binary is included, therefore Secure Boot must be disabled. Adding shim
signing means a Microsoft-signed binary and a review process; it is a future
option, not a current feature.

### Branding layer

The Zaod layer is a set of files copied into the chroot, not a Debian package:

| Path (inside the ISO) | Purpose |
|---|---|
| `/usr/lib/zaod/desktop.py` | Animated minimal home screen |
| `/usr/lib/zaod/onboarding.py` | First-run onboarding window |
| `/usr/lib/zaod/system-monitor.py` | Lightweight CPU/RAM/disk panel |
| `/usr/lib/zaod/first-run.sh` | One-shot first boot tasks |
| `/usr/share/plymouth/themes/zaod/` | Boot animation (script + plymouth theme) |
| `/usr/share/backgrounds/zaod/desktop.png` | Wallpaper |
| `/usr/share/applications/zaod-system-monitor.desktop` | Menu entry |
| `/etc/skel/.config/autostart/` | Desktop + onboarding autostart entries |
| `/etc/skel/Desktop/*.desktop` | Browser, Files, Settings launchers |
| `/etc/hostname` | `zaod` |

Because it is files-only, `dpkg -l` shows no `zaod` package. A proper `.deb`
packaging of this layer is a planned improvement.

## Source access

The build recipe, branding sources and helper tooling are kept in a **private**
repository (`zaod-os-source`) and are not part of this public repository.
Access is granted by the owner only. If you need the sources, ask the owner for
collaborator access; do not open a public issue asking for the code.

Contents of the private source repository:

```
system/                     live-build recipe
  build-iso.sh              one-command build driver
  config/binary             image name, volume label, bootloader, iso settings
  config/bootstrap          Debian mirror, release, architecture, components
  config/common             global live-build options
  config/chroot             chroot stage options
  config/source             apt sources (trixie + security + non-free-firmware)
  config/package-lists/     zaod.list.chroot_live - the package set
  config/hooks/live/        first-run hook
  config/grub.cfg.in        GRUB menu template
  config/includes.chroot/   the branding layer described above
  assets/                   boot animation and art sources
  tools/                    art generators
studio/                     Zaod Studio (package format, tests, guides)
```

## Building an ISO

From the private source repository, on a Debian 13 (or Debian-container) host:

```bash
sudo apt-get install live-build debootstrap squashfs-tools xorriso \
     mtools dosfstools grub-pc-bin grub-efi-amd64-bin
cd system
sudo ./build-iso.sh            # produces binary/live/*.iso
```

`build-iso.sh` runs `lb config` with the recipe flags, `lb build`, then prints
the ISO path, size and SHA-256. Build host requirements: root (for the chroot
and loop mounts), ~15 GB free disk, 2 GB RAM, and a Debian 13 userspace. A
GitHub Actions `ubuntu-24.04` runner works only inside a Debian 13 container,
because the runner's live-build is too old for the recipe flags.

Release output name: `Zaod-OS-x86_64.iso`, tag format
`zaod-os-iso-YYYY.MM.DD[-vN]`.

## Release checklist

1. Build the ISO and record size + SHA-256.
2. Boot it before publishing: BIOS and UEFI, then confirm kernel, hostname,
   greeter and that the desktop comes up.
3. Create a pre-release tag `zaod-os-iso-YYYY.MM.DD[-vN]` with release notes
   listing what changed and what is still broken.
4. Upload `Zaod-OS-x86_64.iso` as the single asset (limit: 2 GiB per asset).
5. Update `README.md` checksum table and `docs/CHANGELOG.md`.
6. Keep the previous ISO available until the new one is confirmed working.

## Verification of a published image

```bash
# structure
7z l Zaod-OS-x86_64.iso | head -40
# hostname
unsquashfs -cat filesystem.squashfs etc/hostname
# kernel version
unsquashfs -cat filesystem.squashfs boot/config-* | head
# autologin / display manager config
unsquashfs -cat filesystem.squashfs etc/lightdm/lightdm.conf
# live boot test (needs KVM for usable speed)
qemu-system-x86_64 -m 1536 -smp 2 -cdrom Zaod-OS-x86_64.iso -nographic
```

A published ISO is only considered good when it reaches the graphical greeter
in a boot test after a byte-for-byte checksum match.

## Configuration of the live session

| Setting | Value |
|---|---|
| Live user | `user` |
| Live password | `live` |
| Hostname | `zaod` |
| Autologin | not enabled yet |
| Default target | graphical (LightDM) |
| zram swap | enabled via `/etc/default/zramswap` |
| Timezone / locale | Debian live defaults, changeable in XFCE |

## License and contributions

Zaod OS is proprietary; see [LICENSE](../LICENSE). Do not copy, fork, rebrand or
redistribute the ISO, its sources or its branding. Bug reports and hardware
compatibility reports are welcome through this repository's issue tracker.
Debian and every bundled upstream component keep their own licenses.
