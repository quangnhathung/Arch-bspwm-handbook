# Arch Linux + bspwm Handbook — Lenovo LOQ 15IAX9

Tài liệu hướng dẫn cài đặt Arch Linux từ con số 0, cấu hình window manager bspwm,
và các thành phần desktop cơ bản trên laptop **Lenovo LOQ 15IAX9**.

## Đối tượng

Người dùng muốn chuyển từ Windows sang Linux, cài mới hoàn toàn, không dual boot.

## Mục tiêu

| Mục tiêu | Chi tiết |
|---|---|
| Cài đặt | Arch Linux, BTRFS, GRUB UEFI |
| Desktop | Xorg + bspwm + sxhkd |
| Đồ họa | Intel UHD + NVIDIA RTX 4050 (hybrid) |
| Mạng | Wi-Fi Realtek RTL8852BE + LAN |
| Âm thanh | PipeWire + Intel SST |
| Snapshot | BTRFS + Timeshift |

## Cấu trúc tài liệu

### 01-introduction — Giới thiệu

- [01-overview.md](docs/01-introduction/01-overview.md)
- [02-arch-linux.md](docs/01-introduction/02-arch-linux.md)
- [03-why-bspwm.md](docs/01-introduction/03-why-bspwm.md)
- [04-system-requirements.md](docs/01-introduction/04-system-requirements.md)

### 02-preparation — Chuẩn bị

- [01-backup-data.md](docs/02-preparation/01-backup-data.md)
- [02-bios-settings.md](docs/02-preparation/02-bios-settings.md)
- [03-secure-boot.md](docs/02-preparation/03-secure-boot.md)
- [04-create-usb.md](docs/02-preparation/04-create-usb.md)
- [05-boot-live-environment.md](docs/02-preparation/05-boot-live-environment.md)

### 03-installation — Cài đặt

- [01-disk-partitioning.md](docs/03-installation/01-disk-partitioning.md)
- [02-btrfs-layout.md](docs/03-installation/02-btrfs-layout.md)
- [03-mounting.md](docs/03-installation/03-mounting.md)
- [04-pacstrap.md](docs/03-installation/04-pacstrap.md)
- [05-fstab.md](docs/03-installation/05-fstab.md)
- [06-locale.md](docs/03-installation/06-locale.md)
- [07-network.md](docs/03-installation/07-network.md)
- [08-users.md](docs/03-installation/08-users.md)
- [09-grub.md](docs/03-installation/09-grub.md)

### 04-desktop — Desktop

- [01-xorg.md](docs/04-desktop/01-xorg.md)
- [02-bspwm.md](docs/04-desktop/02-bspwm.md)
- [03-sxhkd.md](docs/04-desktop/03-sxhkd.md)
- [04-polybar.md](docs/04-desktop/04-polybar.md)
- [05-rofi.md](docs/04-desktop/05-rofi.md)
- [06-picom.md](docs/04-desktop/06-picom.md)
- [07-nitrogen.md](docs/04-desktop/07-nitrogen.md)
- [08-fonts.md](docs/04-desktop/08-fonts.md)
- [09-themes.md](docs/04-desktop/09-themes.md)

### 05-drivers — Driver

- [01-intel.md](docs/05-drivers/01-intel.md)
- [02-nvidia.md](docs/05-drivers/02-nvidia.md)
- [03-hybrid-graphics.md](docs/05-drivers/03-hybrid-graphics.md)
- [04-envycontrol.md](docs/05-drivers/04-envycontrol.md)
- [05-wifi.md](docs/05-drivers/05-wifi.md)
- [06-bluetooth.md](docs/05-drivers/06-bluetooth.md)
- [07-audio.md](docs/05-drivers/07-audio.md)

### 06-package-management — Quản lý gói

- [01-pacman.md](docs/06-package-management/01-pacman.md)
- [02-yay.md](docs/06-package-management/02-yay.md)
- [03-package-search.md](docs/06-package-management/03-package-search.md)
- [04-package-cleanup.md](docs/06-package-management/04-package-cleanup.md)
- [05-maintenance.md](docs/06-package-management/05-maintenance.md)

### 07-btrfs — BTRFS

- [01-snapshots.md](docs/07-btrfs/01-snapshots.md)
- [02-timeshift.md](docs/07-btrfs/02-timeshift.md)
- [03-restore.md](docs/07-btrfs/03-restore.md)
- [04-backup-strategy.md](docs/07-btrfs/04-backup-strategy.md)

### 08-bspwm-guide — Hướng dẫn bspwm

- [01-keybindings.md](docs/08-bspwm-guide/01-keybindings.md)
- [02-workspaces.md](docs/08-bspwm-guide/02-workspaces.md)
- [03-window-management.md](docs/08-bspwm-guide/03-window-management.md)
- [04-layouts.md](docs/08-bspwm-guide/04-layouts.md)
- [05-multi-monitor.md](docs/08-bspwm-guide/05-multi-monitor.md)
- [06-customization.md](docs/08-bspwm-guide/06-customization.md)

### 09-troubleshooting — Xử lý sự cố

- [01-black-screen.md](docs/09-troubleshooting/01-black-screen.md)
- [02-no-wifi.md](docs/09-troubleshooting/02-no-wifi.md)
- [03-no-audio.md](docs/09-troubleshooting/03-no-audio.md)
- [04-nvidia-issues.md](docs/09-troubleshooting/04-nvidia-issues.md)
- [05-bspwm-not-starting.md](docs/09-troubleshooting/05-bspwm-not-starting.md)
- [06-grub-repair.md](docs/09-troubleshooting/06-grub-repair.md)
- [07-emergency-recovery.md](docs/09-troubleshooting/07-emergency-recovery.md)

### appendix — Phụ lục

- [01-full-install-script.md](docs/appendix/01-full-install-script.md)
- [02-post-install-checklist.md](docs/appendix/02-post-install-checklist.md)
- [03-maintenance-checklist.md](docs/appendix/03-maintenance-checklist.md)
- [04-command-cheatsheet.md](docs/appendix/04-command-cheatsheet.md)

## Luồng thực hiện

1. Đọc **01-introduction** để hiểu tổng quan.
2. Làm theo **02-preparation** để chuẩn bị USB boot.
3. Boot USB và làm theo **03-installation** từ đầu đến cuối.
4. Sau khi reboot vào hệ thống mới, làm theo **04-desktop** để có desktop.
5. Cấu hình driver theo **05-drivers**.
6. Đọc **06-package-management** để biết quản lý gói.
7. Thiết lập snapshot theo **07-btrfs**.
8. Đọc **08-bspwm-guide** để làm quen với bspwm.
9. Gặp lỗi → vào **09-troubleshooting**.
10. **appendix** — script, checklist, cheatsheet.

## Yêu cầu

- USB 4GB+
- Máy tính có truy cập Internet
- Sẵn sàng xóa toàn bộ dữ liệu cũ trên ổ cứng

## Cảnh báo

> Quá trình này sẽ **xóa sạch toàn bộ dữ liệu Windows hiện tại**.
> Không giữ lại dual boot. Coi máy như một bản cài hoàn toàn mới.
> Sao lưu dữ liệu quan trọng trước khi bắt đầu.
