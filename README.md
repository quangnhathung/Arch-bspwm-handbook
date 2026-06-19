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

## Cấu trúc tài liệu (click để mở)

### 01-introduction — Giới thiệu tổng quan

- [overview.md](01-introduction/overview.md) — Tổng quan dự án và máy mục tiêu
- [arch-linux.md](01-introduction/arch-linux.md) — Arch Linux là gì?
- [why-bspwm.md](01-introduction/why-bspwm.md) — Tại sao chọn bspwm?
- [system-requirements.md](01-introduction/system-requirements.md) — Yêu cầu hệ thống

### 02-preparation — Chuẩn bị trước khi cài

- [backup-data.md](02-preparation/backup-data.md) — Sao lưu dữ liệu
- [bios-settings.md](02-preparation/bios-settings.md) — Cấu hình BIOS
- [secure-boot.md](02-preparation/secure-boot.md) — Tắt Secure Boot
- [create-usb.md](02-preparation/create-usb.md) — Tạo USB boot Arch
- [boot-live-environment.md](02-preparation/boot-live-environment.md) — Boot live environment

### 03-installation — Cài đặt hệ thống

- [disk-partitioning.md](03-installation/disk-partitioning.md) — Phân vùng ổ đĩa
- [btrfs-layout.md](03-installation/btrfs-layout.md) — BTRFS layout và subvolume
- [mounting.md](03-installation/mounting.md) — Mount hệ thống
- [pacstrap.md](03-installation/pacstrap.md) — Cài base system
- [fstab.md](03-installation/fstab.md) — Tạo fstab
- [locale.md](03-installation/locale.md) — Locale, timezone, hostname
- [network.md](03-installation/network.md) — Cấu hình network
- [users.md](03-installation/users.md) — Tạo user và sudo
- [grub.md](03-installation/grub.md) — GRUB bootloader

### 04-desktop — Thiết lập môi trường đồ họa

- [xorg.md](04-desktop/xorg.md) — Xorg display server
- [bspwm.md](04-desktop/bspwm.md) — Bspwm window manager
- [sxhkd.md](04-desktop/sxhkd.md) — Sxhkd keybinding
- [polybar.md](04-desktop/polybar.md) — Polybar status bar
- [rofi.md](04-desktop/rofi.md) — Rofi launcher
- [picom.md](04-desktop/picom.md) — Picom compositor
- [nitrogen.md](04-desktop/nitrogen.md) — Nitrogen wallpaper
- [fonts.md](04-desktop/fonts.md) — Font chữ
- [themes.md](04-desktop/themes.md) — Themes giao diện

### 05-drivers — Driver phần cứng

- [intel.md](05-drivers/intel.md) — Intel UHD Graphics
- [nvidia.md](05-drivers/nvidia.md) — NVIDIA RTX 4050
- [hybrid-graphics.md](05-drivers/hybrid-graphics.md) — Hybrid graphics
- [envycontrol.md](05-drivers/envycontrol.md) — EnvyControl GPU switch
- [wifi.md](05-drivers/wifi.md) — Wi-Fi Realtek RTL8852BE
- [bluetooth.md](05-drivers/bluetooth.md) — Bluetooth
- [audio.md](05-drivers/audio.md) — PipeWire audio

### 06-package-management — Quản lý gói

- [pacman.md](06-package-management/pacman.md) — Pacman
- [yay.md](06-package-management/yay.md) — Yay AUR helper
- [package-search.md](06-package-management/package-search.md) — Tìm kiếm gói
- [package-cleanup.md](06-package-management/package-cleanup.md) — Dọn dẹp gói
- [maintenance.md](06-package-management/maintenance.md) — Bảo trì hệ thống

### 07-btrfs — BTRFS và Snapshot

- [snapshots.md](07-btrfs/snapshots.md) — BTRFS snapshots
- [timeshift.md](07-btrfs/timeshift.md) — Timeshift
- [restore.md](07-btrfs/restore.md) — Restore từ snapshot
- [backup-strategy.md](07-btrfs/backup-strategy.md) — Chiến lược backup

### 08-bspwm-guide — Hướng dẫn sử dụng bspwm

- [keybindings.md](08-bspwm-guide/keybindings.md) — Danh sách phím tắt
- [workspaces.md](08-bspwm-guide/workspaces.md) — Workspaces
- [window-management.md](08-bspwm-guide/window-management.md) — Quản lý cửa sổ
- [layouts.md](08-bspwm-guide/layouts.md) — Layouts bố cục
- [multi-monitor.md](08-bspwm-guide/multi-monitor.md) — Nhiều màn hình
- [customization.md](08-bspwm-guide/customization.md) — Tùy chỉnh

### 09-troubleshooting — Xử lý sự cố

- [black-screen.md](09-troubleshooting/black-screen.md) — Màn hình đen
- [no-wifi.md](09-troubleshooting/no-wifi.md) — Không có Wi-Fi
- [no-audio.md](09-troubleshooting/no-audio.md) — Không có âm thanh
- [nvidia-issues.md](09-troubleshooting/nvidia-issues.md) — NVIDIA lỗi
- [bspwm-not-starting.md](09-troubleshooting/bspwm-not-starting.md) — bspwm không khởi động
- [grub-repair.md](09-troubleshooting/grub-repair.md) — GRUB lỗi
- [emergency-recovery.md](09-troubleshooting/emergency-recovery.md) — Khôi phục khẩn cấp

### appendix — Phụ lục

- [full-install-script.md](appendix/full-install-script.md) — Full install script
- [post-install-checklist.md](appendix/post-install-checklist.md) — Checklist sau cài
- [maintenance-checklist.md](appendix/maintenance-checklist.md) — Checklist bảo trì
- [command-cheatsheet.md](appendix/command-cheatsheet.md) — Command cheatsheet

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
10. **appendix** — cheat sheet, script, checklist.

## Yêu cầu

- USB 4GB+
- Máy tính có truy cập Internet
- Sẵn sàng xóa toàn bộ dữ liệu cũ trên ổ cứng

## Cảnh báo

> Quá trình này sẽ **xóa sạch toàn bộ dữ liệu Windows hiện tại**.
> Không giữ lại dual boot. Coi máy như một bản cài hoàn toàn mới.
> Sao lưu dữ liệu quan trọng trước khi bắt đầu.
