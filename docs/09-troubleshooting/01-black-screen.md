# Màn hình đen sau khi boot

*Áp dụng cho Lenovo LOQ 15IAX9 — NVIDIA RTX 30/40 series — Kernel 7.x — 25/06/2026*

## Triệu chứng (Symptoms)

- Màn hình đen ngay sau khi chọn GRUB, không có logo POST hoặc splash Arch.
- Con trỏ nhấp nháy ở góc trên bên trái hoặc không có gì cả.
- Đèn nguồn sáng, quạt chạy nhưng màn hình không hiển thị.
- Máy vẫn hoạt động (nghe được âm thanh boot) nhưng không thấy gì.

## Nguyên nhân (Causes)

1. **GRUB không tìm thấy kernel** — boot dừng ở GRUB shell hoặc "No such partition".
2. **NVIDIA driver lỗi** — Xorg crash do module NVIDIA không tương thích với kernel 7.x.
3. **Cấu hình Xorg sai** — file 10-nvidia.conf cũ hoặc sai Driver (dùng `nvidia-open` thay vì `nvidia`).
4. **Kernel panic** — lỗi module nghiêm trọng, không boot được, không vào TTY.
5. **EnvyControl sai chế độ** — chuyển sang chế độ hybrid/nvidia không tương thích với output hiện tại.
6. **xrandr sai output** — Xorg render ra output không tồn tại (eDP-1, HDMI-A-1, DP-1).

## Chẩn đoán (Diagnosis)

### Bước 1: Xem boot log

Từ GRUB menu, nhấn `e` trên boot entry, **xóa chữ `quiet`**, thêm `systemd.log_level=debug`:

```
linux /vmlinuz-linux ... systemd.log_level=debug
```

Nhấn `Ctrl+X` để boot. Dòng cuối cùng trước khi đen cho biết nguyên nhân.

### Bước 2: Vào TTY

Nhấn `Ctrl+Alt+F2` (hoặc F3–F6) — nếu vào được TTY login, hệ thống vẫn ổn, chỉ Xorg lỗi.

```bash
# Xem Xorg log
cat /var/log/Xorg.0.log | grep -iE "error|fail|fatal"

# Xem journal mức error
journalctl -p 3 -xb | tail -40
```

### Bước 3: Nếu không vào TTY

→ Kernel panic. Cần boot từ USB Arch để chẩn đoán (xem bài 07-emergency-recovery).

## Khắc phục (Fix)

### Fix 1: NVIDIA modeset=0 (boot tạm)

Từ GRUB, nhấn `e`, thêm vào cuối dòng `linux`:

```
nvidia-drm.modeset=0
```

Boot được → vào TTY, kiểm tra và sửa NVIDIA:

```bash
# Kiểm tra driver hiện tại
lsmod | grep nvidia

# Nếu dùng nvidia thay vì nvidia-open → cài lại
sudo pacman -S nvidia-open-dkms nvidia-utils
sudo sed -i 's/nvidia/nvidia_open/g' /etc/mkinitcpio.conf
mkinitcpio -P
```

### Fix 2: Blacklist NVIDIA (tạm thời để loại trừ)

```bash
echo -e "blacklist nvidia\nblacklist nvidia_modeset\nblacklist nvidia_uvm\nblacklist nvidia_drm" | sudo tee /etc/modprobe.d/blacklist-nvidia.conf
mkinitcpio -P
```

Nếu boot OK → nguyên nhân là NVIDIA.

### Fix 3: Xóa cấu hình Xorg lỗi

```bash
sudo rm /etc/X11/xorg.conf.d/10-nvidia.conf
# Hoặc backup
sudo mv /etc/X11/xorg.conf.d/10-nvidia.conf{,.bak}
```

### Fix 4: Kernel panic — khôi phục từ USB

```bash
# Boot USB → mount BTRFS
mount /dev/nvme0n1p2 /mnt
mount -o subvol=@ /dev/nvme0n1p2 /mnt
mount /dev/nvme0n1p1 /mnt/efi
arch-chroot /mnt

# Kiểm tra log
journalctl -p 3 -xb | tail -30

# Sửa kernel params hoặc reinstall kernel
sudo pacman -S linux
mkinitcpio -P
grub-mkconfig -o /boot/grub/grub.cfg
```

### Fix 5: Restore snapshot

```bash
# Trong chroot
timeshift --list
timeshift --restore --snapshot 'TÊN-SNAPSHOT'
# Sau đó rebuild GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

## Phòng ngừa (Prevention)

1. **Luôn tạo snapshot trước khi cập nhật NVIDIA/Grub/kernel.**
2. **Kiểm tra cấu hình Xorg trước khi reboot:**

```bash
bash -n /etc/X11/xorg.conf.d/*.conf 2>/dev/null || echo "Lỗi cú pháp Xorg"
```

3. **Giữ `nvidia-drm.modeset=1` cho bản cài đặt ổn định** — chỉ dùng `modeset=0` để debug.
4. **Dùng `nvidia-open-dkms` thay vì `nvidia` trên RTX 30/40 series** — tương thích tốt hơn với kernel 7.x.
5. **Luôn có USB Arch recovery trong túi.**
