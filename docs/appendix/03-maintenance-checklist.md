# Maintenance Checklist

## Mục tiêu

Danh sách kiểm tra bảo trì định kỳ cho Arch Linux.

## Hàng ngày

- [ ] Dùng máy bình thường.

## Hàng tuần

- [ ] Đọc tin tức Arch: https://archlinux.org/news/
- [ ] Cập nhật hệ thống: `yay -Syu`
- [ ] Kiểm tra lỗi sau update: `systemctl --failed`
- [ ] Kiểm tra disk space: `df -h /`

## Hàng tháng

- [ ] Dọn cache pacman: `paccache -rk1`
- [ ] Xóa orphan packages:

```bash
pacman -Qtdq | wc -l
# Nếu > 0
pacman -Rns $(pacman -Qtdq)
```

- [ ] Dọn journal: `journalctl --vacuum-size=100M`
- [ ] Dọn cache yay: `yay -Sc`
- [ ] Kiểm tra BTRFS: `sudo btrfs filesystem usage /`
- [ ] Kiểm tra snapshot Timeshift: `sudo timeshift --list`

### Lệnh dọn dẹp một lần

```bash
#!/bin/bash
echo "=== Cleaning pacman cache ==="
paccache -rk1
echo "=== Removing orphan packages ==="
pacman -Qtdq && pacman -Rns $(pacman -Qtdq) || echo "No orphans"
echo "=== Cleaning yay cache ==="
yay -Sc --noconfirm
echo "=== Cleaning journal ==="
journalctl --vacuum-size=100M
echo "=== Done ==="
```

## Hàng quý (3 tháng)

- [ ] Kiểm tra NVMe health:

```bash
sudo nvme smart-log /dev/nvme0n1
```

- [ ] Kiểm tra pin:

```bash
upower -i /org/freedesktop/UPower/devices/battery_BAT0
```

- [ ] Kiểm tra dung lượng BTRFS:

```bash
sudo btrfs filesystem show
sudo btrfs device stats /
```

- [ ] Xóa snapshot cũ (giữ 1-2 snapshot gần nhất):

```bash
sudo timeshift --delete --snapshot 'TÊN-CŨ'
```

- [ ] Backup danh sách gói:

```bash
pacman -Qqen > ~/backup/pkglist-official.txt
pacman -Qqem > ~/backup/pkglist-aur.txt
```

- [ ] Backup config:

```bash
tar -czf ~/backup/config-$(date +%Y%m%d).tar.gz ~/.config/{bspwm,sxhkd,polybar,rofi,picom,alacritty}
```

- [ ] Vệ sinh máy (lau bụi quạt, thay thermal paste nếu cần).

## Hàng năm

- [ ] Kiểm tra sức khỏe ổ cứng tổng thể.
- [ ] Cân nhắc thay pin nếu health < 80%.
- [ ] Reinstall hệ thống (nếu cần) với config đã backup.

## Lưu ý khi update

### Trước update

```bash
# Backup danh sách gói
pacman -Qqen > ~/backup/pkglist-official.txt
pacman -Qqem > ~/backup/pkglist-aur.txt

# Tạo snapshot
sudo timeshift --create --comments "truoc-update-$(date +%Y%m%d)"
```

### Trong khi update

- Đọc danh sách gói sẽ update.
- Chú ý các gói: kernel, systemd, nvidia, grub.
- Nếu có `pacman -Syu` — đừng ngắt giữa chừng.

### Sau update

```bash
# Nếu update kernel → reboot
sudo reboot

# Kiểm tra kernel version
uname -r

# Kiểm tra lỗi
dmesg | grep -i error
journalctl -p 3 -xb | tail -10
```

## Dấu hiệu cần can thiệp

| Dấu hiệu | Hành động |
|---|---|
| Disk đầy (>90%) | Dọn cache, xóa orphan, mở rộng subvolume |
| Nhiệt độ cao (>80°C idle) | Kiểm tra quạt, thermal paste |
| Boot chậm | Kiểm tra journal, disable service không cần |
| Ứng dụng crash | Kiểm tra log, reinstall |
| Màn hình nhấp nháy | Tắt PSR (i915.enable_psr=0) |
| Wi-Fi mất kết nối | Restart NetworkManager, update driver |
