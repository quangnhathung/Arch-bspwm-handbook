# Màn hình đen

## Symptoms

- Sau khi boot, màn hình đen, không thấy gì.
- Có thể thấy cursor nhấp nháy hoặc không.
- Máy vẫn hoạt động (đèn nguồn sáng, quạt chạy).

## Cause

### Nguyên nhân thường gặp

1. **GRUB không tìm thấy kernel** — boot dừng ở GRUB shell.
2. **NVIDIA driver lỗi** — Xorg crash do NVIDIA.
3. **Xorg không khởi động** — lỗi cấu hình Xorg.
4. **Kernel panic** — lỗi nghiêm trọng, không boot được.
5. **Màn hình được chuyển sang output sai** — xrandr sai.
6. **EnvyControl sai chế độ** — chuyển sang chế độ không tương thích.

## Diagnosis

### Kiểm tra boot process

Khi boot, bỏ `quiet` trong GRUB để xem log:

1. Ở GRUB menu, nhấn `e` để sửa boot entry.
2. Xóa `quiet` và thêm `systemd.log_level=debug`.
3. Nhấn `Ctrl+X` hoặc `F10` để boot.
4. Quan sát log — dòng cuối cùng trước khi đen cho biết nguyên nhân.

### Nếu vào được TTY

Nhấn `Ctrl+Alt+F2` (hoặc F3-F6) để chuyển sang TTY text.
Nếu thấy TTY login → Xorg lỗi, hệ thống vẫn chạy.

```bash
# Login vào TTY
# Kiểm tra Xorg log
cat /var/log/Xorg.0.log | grep -i error

# Kiểm tra journal
journalctl -p 3 -xb | tail -30
```

### Nếu không vào được TTY → kernel panic

- Cần boot từ USB để sửa.
- Nếu có snapshot BTRFS → restore (xem bài restore.md).

## Fix

### Fix 1: NVIDIA driver lỗi

Nếu nghi ngờ NVIDIA:

```bash
# Từ GRUB, nhấn e, thêm vào cuối dòng linux:
nvidia-drm.modeset=0

# Hoặc xóa hoàn toàn NVIDIA params
```

Nếu boot được → vô hiệu hóa NVIDIA tạm thời:

```bash
# Blacklist NVIDIA module
sudo vim /etc/modprobe.d/blacklist-nvidia.conf

# Thêm:
blacklist nvidia
blacklist nvidia_modeset
blacklist nvidia_uvm
blacklist nvidia_drm
```

Reboot. Nếu OK → cần cấu hình lại NVIDIA.

### Fix 2: Xorg lỗi

```bash
# Xóa file cấu hình Xorg lỗi
sudo rm /etc/X11/xorg.conf.d/10-nvidia.conf

# Hoặc rename để backup
sudo mv /etc/X11/xorg.conf.d/10-nvidia.conf /etc/X11/xorg.conf.d/10-nvidia.conf.bak
```

### Fix 3: Kernel panic

Boot từ USB, mount hệ thống, kiểm tra log:

```bash
# Mount
mount /dev/nvme0n1p2 /mnt
mount -o subvol=@ /dev/nvme0n1p2 /mnt
mount /dev/nvme0n1p1 /mnt/efi

# Xem log
cat /mnt/var/log/journal/*/system.journal

# Sửa kernel params
vim /mnt/etc/default/grub
```

### Fix 4: Restore snapshot

```bash
# Boot USB → mount → restore snapshot
# Xem docs/07-btrfs/restore.md
```

### Fix 5: Reinstall GRUB

```bash
# Boot USB
arch-chroot /mnt
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB --recheck
grub-mkconfig -o /boot/grub/grub.cfg
```

## Prevention

1. **Luôn tạo snapshot trước khi thay đổi lớn** (update, cấu hình NVIDIA).
2. **Kiểm tra cấu hình Xorg** trước khi reboot.
3. **Giữ kernel parameter `nvidia-drm.modeset=1`** thay vì modeset=0.
4. **Dùng Timeshift** snapshot tự động hàng ngày.
5. **Kiểm tra log** sau mỗi lần cấu hình GPU.
