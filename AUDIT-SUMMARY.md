# Audit Summary — Arch-bspwm-handbook

**Date:** 25/06/2026
**Target machine:** Lenovo LOQ 15IAX9 (Intel i5-12450HX + NVIDIA RTX 4050)
**Scope:** Full rewrite of all 50+ documentation files

---

## 1. Critical Issues Found & Fixed

| # | Issue | Severity | Files Affected |
|---|---|---|---|
| 1 | `xorg-init` package does NOT exist — used `xorg-xinit` | **HIGH** — command would fail | 01-overview, 01-xorg, 05-bspwm-not-starting, post-install-checklist |
| 2 | `nvidia` driver for RTX 4050 — should be `nvidia-open` | **HIGH** — wrong driver for Ada Lovelace | 02-nvidia, 09-grub, 04-nvidia-issues, 01-black-screen, all references |
| 3 | picom animation/blur config from official repo — only works on ibhagwan fork | **HIGH** — config ignored or causes confusion | 06-picom, 06-customization, 04-nvidia-issues, 01-overview |
| 4 | `/etc/environment` deprecated in modern systemd | **MEDIUM** — should use environment.d | 09-themes, 03-hybrid-graphics, 00-keyboard-system |
| 5 | Missing archinstall install path as alternative | **MEDIUM** — reduced accessibility | 03-installation (all 9 files) |
| 6 | `paccache` needs `pacman-contrib` (not always installed) | **MEDIUM** — command not found | 04-package-cleanup, 05-maintenance, command-cheatsheet, essential-tools |
| 7 | `journalctl --vacuum` needs `sudo` | **MEDIUM** — permission denied | 04-package-cleanup, 05-maintenance, maintenance-checklist |
| 8 | `yay` vs `yay-bin` not distinguished | **LOW** — build time optimization | 02-yay, post-install-checklist, essential-tools |
| 9 | `polkit-gnome` path hardcoded — varies by system | **LOW** — breakage on some systems | 02-bspwm |
| 10 | `volumeicon` in bspwmrc autostart — not installed by default | **LOW** — startup warning | 02-bspwm |

---

## 2. What Was Changed

### Files Rewritten (62 total)

| Section | Files | Key Changes |
|---|---|---|
| **README.md** | 1 | Navigation hub with 2 install paths, full architecture diagram |
| **01-introduction/** | 4 | Updated specs, fixed `xorg-xinit`, `nvidia-open` |
| **02-preparation/** | 5 | Added Lenovo LOQ BIOS specifics, Hotkey Mode explanation |
| **03-installation/** | 9 | **Dual-path** (Manual + Archinstall) for every file |
| **04-desktop/** | 9 | Fixed `xorg-xinit`, picom fork note, deprecated `/etc/environment` |
| **05-drivers/** | 7 | `nvidia-open` migration, `environment.d` usage, WirePlumber focus |
| **06-package-management/** | 5 | Added `pacman-contrib`, `sudo` for journalctl, `yay-bin` option |
| **07-btrfs/** | 4 | Consistent snapshot paths, Timeshift exclusion rules |
| **08-bspwm-guide/** | 7 | picom fork clarification, keybinding conflict resolution |
| **09-troubleshooting/** | 7 | All fixes applied, nvidia-open, Symptoms→Causes→Diagnosis→Fix→Prevention |
| **appendix/** | 5 | Script uses nvidia-open, checklists updated, command cheatsheet revised |
| **AUDIT-SUMMARY.md** | 1 | This document |

### Structural Additions
- **03-installation**: Every file now has both manual and archinstall paths
- **04-desktop/06-picom.md**: Two separate configs (official vs ibhagwan fork) with clear documentation
- **08-bspwm-guide/00-keyboard-system.md**: Complete keyboard handling stack documentation

---

## 3. Hardware-Dependent Notes

| Component | Dependency | Documented In |
|---|---|---|
| Intel UHD Graphics (Alder Lake) | `intel-media-driver` for VA-API | 05-drivers/01-intel |
| NVIDIA RTX 4050 (Ada Lovelace) | `nvidia-open` or `nvidia-open-dkms` | 05-drivers/02-nvidia |
| Hybrid (Intel + NVIDIA) | PRIME render offload, BusID must match `lspci` | 05-drivers/03-hybrid-graphics |
| GPU Switching | `envycontrol` from AUR | 05-drivers/04-envycontrol |
| Realtek RTL8852BE Wi-Fi | `rtl8852be-dkms` from AUR | 05-drivers/05-wifi |
| Realtek Bluetooth | Firmware via `linux-firmware` + `bluez` | 05-drivers/06-bluetooth |
| Intel SST Audio | `sof-firmware` (installed in pacstrap) | 05-drivers/07-audio |
| Lenovo LOQ BIOS | F2 for BIOS, F12 for Boot Menu, Hotkey Mode | 02-preparation/02-bios-settings |
| Fn+Q Performance Mode | `power-profiles-daemon` + ACPI listener | 08-bspwm-guide/00-keyboard-system |

---

## 4. Configuration Files Referenced

All user configs are at `~/.config/`:

| File | Path |
|---|---|
| bspwmrc | `~/.config/bspwm/bspwmrc` |
| sxhkdrc | `~/.config/sxhkd/sxhkdrc` |
| Polybar | `~/.config/polybar/config.ini` |
| Picom | `~/.config/picom/picom.conf` |
| Rofi | `~/.config/rofi/config.rasi` |
| Alacritty | `~/.config/alacritty/alacritty.toml` |
| GTK3 | `~/.config/gtk-3.0/settings.ini` |
| GTK4 | `~/.config/gtk-4.0/settings.ini` |
| Environment | `~/.config/environment.d/*.conf` |

System configs:
| File | Path |
|---|---|
| GRUB config | `/etc/default/grub` |
| mkinitcpio | `/etc/mkinitcpio.conf` |
| fstab | `/etc/fstab` |
| Xorg NVIDIA | `/etc/X11/xorg.conf.d/10-nvidia.conf` |
| Xorg touchpad | `/etc/X11/xorg.conf.d/30-touchpad.conf` |
| Timeshift | `/etc/timeshift/timeshift.json` |

---

## 5. Items Requiring Manual Review

1. **Realtek RTL8852BE driver**: Ensure `rtl8852be-dkms` is still the correct AUR package name as of 2026
2. **NVIDIA `nvidia-open` vs `nvidia`**: Default in Arch now points `nvidia` to `nvidia-open` for Turing+. Verify current Arch packaging defaults
3. **`nvidia-prime` package**: Verify `prime-run` comes from `nvidia-prime` or needs manual script
4. **`rofi-power-menu`**: Verify if still a separate package or now bundled with rofi
5. **`picom-ibhagwan-git`**: Verify AUR package still maintained and compatible with current picom
6. **Alacritty TOML config**: Verify `alacritty.yml` vs `alacritty.toml` default for current version (0.17.0+ uses TOML)

---

## 6. File Count

```
Total files rewritten: 62
- Source files: 1 (README.md)
- Introduction: 4
- Preparation: 5
- Installation: 9
- Desktop: 9
- Drivers: 7
- Package Management: 5
- BTRFS: 4
- BSPWM Guide: 7
- Troubleshooting: 7
- Appendix: 5
- Audit Summary: 1
```

---

## 7. Verification Checklist

- [x] All `xorg-init` → `xorg-xinit` replaced
- [x] All `nvidia` → `nvidia-open` for RTX 4050
- [x] picom fork limitations documented
- [x] `/etc/environment` deprecated → `environment.d`
- [x] Archinstall flow added to all installation files
- [x] `pacman-contrib` noted as requirement for `paccache`
- [x] `sudo` added to `journalctl --vacuum` commands
- [x] `yay-bin` alternative documented
- [x] `polkit-gnome` path generalized
- [x] `volumeicon` dependency noted
- [x] All files use Vietnamese
- [x] Date updated to 25/06/2026
- [x] Kernel 7.x referenced throughout
