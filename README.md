# 🏗️ Arch Linux + bspwm Handbook — Lenovo LOQ 15IAX9

**Cài đặt Arch Linux từ con số 0 — Desktop bspwm tối giản — Lenovo LOQ 15IAX9 (Intel i5-12450HX + NVIDIA RTX 4050)**

---

## Đối tượng

Người dùng muốn chuyển từ Windows sang Linux, cài mới hoàn toàn (không dual boot), và xây dựng môi trường desktop tối giản với **bspwm window manager** trên laptop Lenovo LOQ 15IAX9.

## Hardware mục tiêu

| Linh kiện    | Thông số                                   |
| ------------ | ------------------------------------------ |
| CPU          | Intel Core i5-12450HX (Alder Lake, 8 nhân) |
| GPU tích hợp | Intel UHD Graphics                         |
| GPU rời      | NVIDIA GeForce RTX 4050 6GB                |
| RAM          | 16GB DDR5                                  |
| Storage      | Micron 512GB NVMe PCIe 4.0                 |
| Wi-Fi        | Realtek RTL8852BE                          |
| Âm thanh     | Intel SST + SOF firmware                   |

---

## Hai lộ trình cài đặt

Tài liệu cung cấp **2 cách tiếp cận**:

| Phương pháp      | Mô tả                                   | Thời gian  | Phù hợp cho                         |
| ---------------- | --------------------------------------- | ---------- | ----------------------------------- |
| **Cài thủ công** | Từng bước từ USB → base → desktop       | 2–4 giờ    | Người muốn hiểu sâu                 |
| **Archinstall**  | Dùng script archinstall có sẵn trên ISO | 30–60 phút | Người muốn nhanh, đã có kinh nghiệm |

> ⚠️ **Cả hai lộ trình đều xóa sạch toàn bộ dữ liệu cũ.** Sao lưu trước khi bắt đầu.

---

## Điều hướng tài liệu

### 01-introduction — Giới thiệu

| Bài                                                                         | Nội dung                                   |
| --------------------------------------------------------------------------- | ------------------------------------------ |
| [01-overview.md](docs/01-introduction/01-overview.md)                       | Tổng quan, triết lý, phạm vi, máy mục tiêu |
| [02-arch-linux.md](docs/01-introduction/02-arch-linux.md)                   | Arch Linux, rolling release, pacman, AUR   |
| [03-why-bspwm.md](docs/01-introduction/03-why-bspwm.md)                     | Tại sao bspwm, tiling vs stacking, so sánh |
| [04-system-requirements.md](docs/01-introduction/04-system-requirements.md) | Yêu cầu phần cứng, phần mềm, BIOS          |

### 02-preparation — Chuẩn bị

| Bài                                                                            | Nội dung                                          |
| ------------------------------------------------------------------------------ | ------------------------------------------------- |
| [01-backup-data.md](docs/02-preparation/01-backup-data.md)                     | Sao lưu dữ liệu Windows                           |
| [02-bios-settings.md](docs/02-preparation/02-bios-settings.md)                 | Cấu hình BIOS: UEFI, Secure Boot, Fast Boot, AHCI |
| [03-secure-boot.md](docs/02-preparation/03-secure-boot.md)                     | Tắt Secure Boot                                   |
| [04-create-usb.md](docs/02-preparation/04-create-usb.md)                       | Tạo USB boot (Rufus, dd, balenaEtcher)            |
| [05-boot-live-environment.md](docs/02-preparation/05-boot-live-environment.md) | Boot USB, kết nối mạng (iwctl, tethering)         |

### 03-installation — Cài đặt

| Bài                                                                     | Nội dung                                      |
| ----------------------------------------------------------------------- | --------------------------------------------- |
| [01-disk-partitioning.md](docs/03-installation/01-disk-partitioning.md) | Xóa sạch ổ, GPT + EFI + BTRFS                 |
| [02-btrfs-layout.md](docs/03-installation/02-btrfs-layout.md)           | Subvolume @, @home, @log, @pkg                |
| [03-mounting.md](docs/03-installation/03-mounting.md)                   | Mount với options tối ưu                      |
| [04-pacstrap.md](docs/03-installation/04-pacstrap.md)                   | Cài base system + giải thích từng gói         |
| [05-fstab.md](docs/03-installation/05-fstab.md)                         | Sinh fstab với UUID                           |
| [06-locale.md](docs/03-installation/06-locale.md)                       | Timezone, locale, hostname                    |
| [07-network.md](docs/03-installation/07-network.md)                     | NetworkManager + iwd                          |
| [08-users.md](docs/03-installation/08-users.md)                         | Tạo user, sudo, password                      |
| [09-grub.md](docs/03-installation/09-grub.md)                           | GRUB UEFI, kernel params (nvidia-drm.modeset) |

### 04-desktop — Thiết lập desktop

| Bài                                              | Nội dung                                                     |
| ------------------------------------------------ | ------------------------------------------------------------ |
| [01-xorg.md](docs/04-desktop/01-xorg.md)         | Xorg display server, `~/.xinitrc` (dùng `xorg-xinit`)        |
| [02-bspwm.md](docs/04-desktop/02-bspwm.md)       | Bspwm, bspwmrc, rules                                        |
| [03-sxhkd.md](docs/04-desktop/03-sxhkd.md)       | Sxhkd keybinding (lưu ý xung đột phím)                       |
| [04-polybar.md](docs/04-desktop/04-polybar.md)   | Polybar status bar                                           |
| [05-rofi.md](docs/04-desktop/05-rofi.md)         | Rofi application launcher                                    |
| [06-picom.md](docs/04-desktop/06-picom.md)       | Picom compositor — dùng **ibhagwan fork** cho animation/blur |
| [07-nitrogen.md](docs/04-desktop/07-nitrogen.md) | Nitrogen wallpaper                                           |
| [08-fonts.md](docs/04-desktop/08-fonts.md)       | FiraCode, Noto, Nerd Font                                    |
| [09-themes.md](docs/04-desktop/09-themes.md)     | GTK theme, icon, cursor (Nordic, Papirus)                    |

### 05-drivers — Driver đồ họa và phần cứng

| Bài                                                            | Nội dung                                                           |
| -------------------------------------------------------------- | ------------------------------------------------------------------ |
| [01-intel.md](docs/05-drivers/01-intel.md)                     | Intel UHD, Mesa, VA-API                                            |
| [02-nvidia.md](docs/05-drivers/02-nvidia.md)                   | NVIDIA RTX 4050 — dùng **nvidia-open** (khuyến nghị cho RTX 30/40) |
| [03-hybrid-graphics.md](docs/05-drivers/03-hybrid-graphics.md) | Intel + NVIDIA Optimus, PRIME render offload, `prime-run`          |
| [04-envycontrol.md](docs/05-drivers/04-envycontrol.md)         | EnvyControl: switch Intel / NVIDIA / Hybrid                        |
| [05-wifi.md](docs/05-drivers/05-wifi.md)                       | Realtek RTL8852BE, driver từ AUR                                   |
| [06-bluetooth.md](docs/05-drivers/06-bluetooth.md)             | BlueZ, blueman                                                     |
| [07-audio.md](docs/05-drivers/07-audio.md)                     | PipeWire, WirePlumber, Intel SST + sof-firmware                    |

### 06-package-management — Quản lý gói

| Bài                                                                       | Nội dung                                |
| ------------------------------------------------------------------------- | --------------------------------------- |
| [01-pacman.md](docs/06-package-management/01-pacman.md)                   | Pacman: install, update, remove, search |
| [02-yay.md](docs/06-package-management/02-yay.md)                         | Yay AUR helper                          |
| [03-package-search.md](docs/06-package-management/03-package-search.md)   | Tìm kiếm trong repo và AUR              |
| [04-package-cleanup.md](docs/06-package-management/04-package-cleanup.md) | Dọn cache, orphan, journal              |
| [05-maintenance.md](docs/06-package-management/05-maintenance.md)         | Bảo trì, tránh partial upgrade          |

### 07-btrfs — BTRFS và snapshot

| Bài                                                          | Nội dung                   |
| ------------------------------------------------------------ | -------------------------- |
| [01-snapshots.md](docs/07-btrfs/01-snapshots.md)             | Snapshot thủ công          |
| [02-timeshift.md](docs/07-btrfs/02-timeshift.md)             | Timeshift tự động          |
| [03-restore.md](docs/07-btrfs/03-restore.md)                 | Rollback từ snapshot       |
| [04-backup-strategy.md](docs/07-btrfs/04-backup-strategy.md) | Backup: rsync, Borg, cloud |

### 08-bspwm-guide — Hướng dẫn sử dụng bspwm

| Bài                                                                    | Nội dung                               |
| ---------------------------------------------------------------------- | -------------------------------------- |
| [00-keyboard-system.md](docs/08-bspwm-guide/00-keyboard-system.md)     | Hệ thống phím: Fn, keycode, keybinding |
| [01-keybindings.md](docs/08-bspwm-guide/01-keybindings.md)             | Danh sách đầy đủ phím tắt              |
| [02-workspaces.md](docs/08-bspwm-guide/02-workspaces.md)               | Workspace ảo, chuyển đổi, gán ứng dụng |
| [03-window-management.md](docs/08-bspwm-guide/03-window-management.md) | Focus, move, resize, split, state      |
| [04-layouts.md](docs/08-bspwm-guide/04-layouts.md)                     | Tiled, monocle, floating               |
| [05-multi-monitor.md](docs/08-bspwm-guide/05-multi-monitor.md)         | Nhiều màn hình, autorandr              |
| [06-customization.md](docs/08-bspwm-guide/06-customization.md)         | Tùy chỉnh giao diện, theme, scripts    |

### 09-troubleshooting — Xử lý sự cố

| Bài                                                                          | Nội dung                   |
| ---------------------------------------------------------------------------- | -------------------------- |
| [01-black-screen.md](docs/09-troubleshooting/01-black-screen.md)             | Màn hình đen sau boot      |
| [02-no-wifi.md](docs/09-troubleshooting/02-no-wifi.md)                       | Wi-Fi không hoạt động      |
| [03-no-audio.md](docs/09-troubleshooting/03-no-audio.md)                     | Không có âm thanh          |
| [04-nvidia-issues.md](docs/09-troubleshooting/04-nvidia-issues.md)           | NVIDIA lỗi, screen tearing |
| [05-bspwm-not-starting.md](docs/09-troubleshooting/05-bspwm-not-starting.md) | bspwm không khởi động      |
| [06-grub-repair.md](docs/09-troubleshooting/06-grub-repair.md)               | GRUB không boot            |
| [07-emergency-recovery.md](docs/09-troubleshooting/07-emergency-recovery.md) | Khôi phục khẩn cấp từ USB  |

### appendix — Phụ lục

| Bài                                                                        | Nội dung                   |
| -------------------------------------------------------------------------- | -------------------------- |
| [01-full-install-script.md](docs/appendix/01-full-install-script.md)       | Script tự động cài toàn bộ |
| [02-post-install-checklist.md](docs/appendix/02-post-install-checklist.md) | Checklist sau cài          |
| [03-maintenance-checklist.md](docs/appendix/03-maintenance-checklist.md)   | Checklist bảo trì định kỳ  |
| [04-command-cheatsheet.md](docs/appendix/04-command-cheatsheet.md)         | Tổng hợp lệnh hữu ích      |
| [05-essential-tools.md](docs/appendix/05-essential-tools.md)               | Essential Tools cho dev    |

---

## Luồng đọc đề xuất

```
01-introduction ──► 02-preparation ──► 03-installation ──► 04-desktop ──► 05-drivers
                                                                                │
                                                                                ▼
                                                    08-bspwm-guide ◄── 07-btrfs ◄── 06-package-management
                                                           │
                                                           ▼
                                                    09-troubleshooting (khi cần)
                                                           │
                                                           ▼
                                                    appendix (tra cứu)
```

1. **01-introduction** — đọc trước để hiểu tổng quan
2. **02-preparation** — chuẩn bị USB, BIOS
3. **03-installation** — cài base system
4. **04-desktop** — Xorg + bspwm + các thành phần desktop
5. **05-drivers** — cấu hình đồ họa, Wi-Fi, âm thanh
6. **06-package-management** — làm quen pacman, yay
7. **07-btrfs** — snapshot để đề phòng
8. **08-bspwm-guide** — học cách dùng bspwm
9. **09-troubleshooting** — chỉ đọc khi gặp lỗi
10. **appendix** — tra cứu nhanh, script tự động

---

## Yêu cầu trước khi bắt đầu

- USB 4GB+
- Máy tính có truy cập Internet
- Sẵn sàng xóa toàn bộ dữ liệu cũ
- Đọc trước các bài trong 01-introduction

---

## Cảnh báo quan trọng

> ⚠️ **Toàn bộ dữ liệu trên ổ cứng sẽ bị xóa sạch.** Không dual boot.
>
> ⚠️ **Đây là hệ thống Linux thuần.** Không chạy được ứng dụng Windows (.exe)
> nếu không dùng Wine (không bao gồm trong tài liệu này).
>
> ⚠️ **Mọi thao tác đều có thể gây hỏng hệ thống nếu làm sai.**
> Đọc kỹ từng bước trước khi thực hiện.
>
> ⚠️ **Sao lưu dữ liệu quan trọng trước khi bắt đầu.**

---

## Ghi chú sửa đổi

- **25/06/2026** — Viết lại toàn bộ tài liệu. Sửa các lỗi: `xorg-init` → `xorg-xinit`,
  picom ibhagwan fork, khuyến nghị `nvidia-open`, bổ sung lộ trình Archinstall,
  cảnh báo `volumeicon` và `polkit-gnome` path.
