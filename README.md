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

- [01-overview.md](docs/01-introduction/01-overview.md) — Tổng quan dự án, triết lý, phạm vi
- [02-arch-linux.md](docs/01-introduction/02-arch-linux.md) — Arch Linux là gì? rolling release, pacman, AUR
- [03-why-bspwm.md](docs/01-introduction/03-why-bspwm.md) — Lý do chọn bspwm, so sánh tiling vs stacking
- [04-system-requirements.md](docs/01-introduction/04-system-requirements.md) — Cấu hình máy mục tiêu và yêu cầu phần cứng

### 02-preparation — Chuẩn bị

- [01-backup-data.md](docs/02-preparation/01-backup-data.md) — Sao lưu dữ liệu Windows trước khi xóa
- [02-bios-settings.md](docs/02-preparation/02-bios-settings.md) — Cấu hình BIOS: UEFI, Secure Boot, Fast Boot, AHCI
- [03-secure-boot.md](docs/02-preparation/03-secure-boot.md) — Giải thích và tắt Secure Boot
- [04-create-usb.md](docs/02-preparation/04-create-usb.md) — Tạo USB boot Arch bằng Rufus
- [05-boot-live-environment.md](docs/02-preparation/05-boot-live-environment.md) — Boot USB, kết nối mạng (iwctl, tethering)

### 03-installation — Cài đặt

- [01-disk-partitioning.md](docs/03-installation/01-disk-partitioning.md) — Xóa sạch ổ, tạo GPT + EFI + BTRFS
- [02-btrfs-layout.md](docs/03-installation/02-btrfs-layout.md) — Tạo subvolume @, @home, @log, @pkg, @swap
- [03-mounting.md](docs/03-installation/03-mounting.md) — Mount đúng vị trí với options tối ưu
- [04-pacstrap.md](docs/03-installation/04-pacstrap.md) — Cài base system + giải thích từng gói
- [05-fstab.md](docs/03-installation/05-fstab.md) — Sinh fstab với UUID
- [06-locale.md](docs/03-installation/06-locale.md) — Timezone, locale, hostname
- [07-network.md](docs/03-installation/07-network.md) — NetworkManager + iwd
- [08-users.md](docs/03-installation/08-users.md) — Tạo user, sudo, password
- [09-grub.md](docs/03-installation/09-grub.md) — GRUB UEFI, kernel params (nvidia-drm.modeset)

### 04-desktop — Desktop

- [01-xorg.md](docs/04-desktop/01-xorg.md) — Xorg display server, .xinitrc
- [02-bspwm.md](docs/04-desktop/02-bspwm.md) — Bspwm window manager, bspwmrc, rules
- [03-sxhkd.md](docs/04-desktop/03-sxhkd.md) — Sxhkd keybinding, sxhkdrc đầy đủ
- [04-polybar.md](docs/04-desktop/04-polybar.md) — Polybar status bar + modules
- [05-rofi.md](docs/04-desktop/05-rofi.md) — Rofi application launcher
- [06-picom.md](docs/04-desktop/06-picom.md) — Picom compositor (shadow, fade, vsync)
- [07-nitrogen.md](docs/04-desktop/07-nitrogen.md) — Nitrogen wallpaper manager
- [08-fonts.md](docs/04-desktop/08-fonts.md) — Font chữ (FiraCode, Noto, Nerd Font)
- [09-themes.md](docs/04-desktop/09-themes.md) — GTK theme, icon, cursor (Nordic, Papirus)

### 05-drivers — Driver

- [01-intel.md](docs/05-drivers/01-intel.md) — Intel UHD Graphics, Mesa, VA-API
- [02-nvidia.md](docs/05-drivers/02-nvidia.md) — NVIDIA RTX 4050, nvidia-drm.modeset
- [03-hybrid-graphics.md](docs/05-drivers/03-hybrid-graphics.md) — Intel + NVIDIA Optimus, PRIME render offload
- [04-envycontrol.md](docs/05-drivers/04-envycontrol.md) — EnvyControl: switch Intel / NVIDIA / Hybrid
- [05-wifi.md](docs/05-drivers/05-wifi.md) — Realtek RTL8852BE, driver từ AUR
- [06-bluetooth.md](docs/05-drivers/06-bluetooth.md) — BlueZ, blueman
- [07-audio.md](docs/05-drivers/07-audio.md) — PipeWire, WirePlumber, Intel SST

### 06-package-management — Quản lý gói

- [01-pacman.md](docs/06-package-management/01-pacman.md) — Pacman: install, update, remove, search
- [02-yay.md](docs/06-package-management/02-yay.md) — Yay AUR helper, cài từ AUR
- [03-package-search.md](docs/06-package-management/03-package-search.md) — Tìm kiếm gói trong repo và AUR
- [04-package-cleanup.md](docs/06-package-management/04-package-cleanup.md) — Dọn cache, orphan, journal
- [05-maintenance.md](docs/06-package-management/05-maintenance.md) — Bảo trì hệ thống, tránh partial upgrade

### 07-btrfs — BTRFS

- [01-snapshots.md](docs/07-btrfs/01-snapshots.md) — Tạo snapshot thủ công
- [02-timeshift.md](docs/07-btrfs/02-timeshift.md) — Timeshift tự động snapshot
- [03-restore.md](docs/07-btrfs/03-restore.md) — Rollback từ snapshot khi hỏng
- [04-backup-strategy.md](docs/07-btrfs/04-backup-strategy.md) — Chiến lược backup (rsync, Borg, cloud)

### 08-bspwm-guide — Hướng dẫn bspwm

- [01-keybindings.md](docs/08-bspwm-guide/01-keybindings.md) — Danh sách đầy đủ phím tắt
- [02-workspaces.md](docs/08-bspwm-guide/02-workspaces.md) — Workspace ảo, chuyển đổi, gán ứng dụng
- [03-window-management.md](docs/08-bspwm-guide/03-window-management.md) — Focus, move, resize, split, state
- [04-layouts.md](docs/08-bspwm-guide/04-layouts.md) — Tiled, monocle, floating
- [05-multi-monitor.md](docs/08-bspwm-guide/05-multi-monitor.md) — Nhiều màn hình, autorandr
- [06-customization.md](docs/08-bspwm-guide/06-customization.md) — Tùy chỉnh giao diện, theme, scripts

### 09-troubleshooting — Xử lý sự cố

- [01-black-screen.md](docs/09-troubleshooting/01-black-screen.md) — Màn hình đen sau boot
- [02-no-wifi.md](docs/09-troubleshooting/02-no-wifi.md) — Wi-Fi không hoạt động
- [03-no-audio.md](docs/09-troubleshooting/03-no-audio.md) — Không có âm thanh
- [04-nvidia-issues.md](docs/09-troubleshooting/04-nvidia-issues.md) — NVIDIA lỗi, screen tearing
- [05-bspwm-not-starting.md](docs/09-troubleshooting/05-bspwm-not-starting.md) — bspwm không khởi động
- [06-grub-repair.md](docs/09-troubleshooting/06-grub-repair.md) — GRUB không boot
- [07-emergency-recovery.md](docs/09-troubleshooting/07-emergency-recovery.md) — Khôi phục khẩn cấp từ USB

### appendix — Phụ lục

- [01-full-install-script.md](docs/appendix/01-full-install-script.md) — Script tự động cài toàn bộ
- [02-post-install-checklist.md](docs/appendix/02-post-install-checklist.md) — Checklist sau cài
- [03-maintenance-checklist.md](docs/appendix/03-maintenance-checklist.md) — Checklist bảo trì định kỳ
- [04-command-cheatsheet.md](docs/appendix/04-command-cheatsheet.md) — Tổng hợp lệnh hữu ích

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
