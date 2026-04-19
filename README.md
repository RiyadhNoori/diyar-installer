# diyar-installer

> The installation engine for **Diyar OS — ديار**
> Reliable rsync-based live-to-HDD installer for Debian-based systems.

---

## Why a custom installer?

Both Calamares and hand-written GTK3 Python installers fail on custom Debian live ISOs for the same reasons:
- Complex C++/Python partitioning code with poor error recovery
- Fragile `grub-install` invocations that miss the `--recheck` flag
- `fstab` entries using `/dev/sdX` paths instead of UUIDs — breaks on next boot
- Packages that re-download from the internet mid-install

**diyar-installer** uses the proven pattern from **refractainstaller** (used by Devuan, ExTiX, AntiX, and dozens of successful Debian spins):

```
rsync -aAXHv /run/live/rootfs/ → /target/
chroot /target → grub-install → update-grub → UUID fstab
```

This copies the *exact running live system* — no re-download, no mystery, works offline.

---

## Repository structure

```
diyar-installer/
├── core/
│   ├── engine.sh          ← rsync engine + orchestration
│   ├── disk.sh            ← partition naming, UUID resolution
│   ├── chroot_setup.sh    ← hostname, locale, fstab, user, grub
│   └── log.sh             ← logging utilities
├── ui/
│   └── diyar-installer    ← whiptail/dialog wizard UI
├── hooks/
│   └── post-install/
│       └── 01-arabic-setup.sh  ← keyboard, IBus, fontconfig, LightDM
├── conf/
│   └── installer.conf     ← runtime configuration
├── diyar-install.desktop  ← live session desktop launcher
├── Makefile               ← install/uninstall
└── README.md
```

---

## Installation flow

```
① Welcome screen
② Select target disk (auto-detected, shown with size + model)
③ Partition mode: auto (wipe+GPT/MBR) or manual (pre-partitioned)
④ Hostname
⑤ Username + password
⑥ Locale + timezone (Arabic locales first)
⑦ Confirmation summary
⑧ Install (rsync + chroot + grub) with live progress bar
⑨ Done → reboot
```

---

## BIOS and UEFI support

| Firmware | Partition table | Boot partition | Grub package |
|---|---|---|---|
| BIOS/Legacy | MBR | none | `grub-pc` |
| UEFI | GPT | 512MB FAT32 ESP | `grub-efi-amd64` |

Auto-detected by checking `/sys/firmware/efi`.

---

## Usage on the live ISO

```bash
sudo diyar-installer
```

Or click the **"Install Diyar OS"** icon on the desktop.

---

## Building into the ISO

In `diyar-os/config/package-lists/installer.list.chroot`:
```
whiptail
rsync
parted
grub-pc
grub-efi-amd64
grub-efi-amd64-bin
```

In `diyar-os/config/hooks/live/0060-install-diyar-installer.hook.chroot`:
```bash
#!/bin/bash
cd /opt/diyar-installer && make install
```

---

## License

GPL v3 — same as the refractainstaller pattern it is based on.
