# Maintenance Checklist — Arch Linux (Lenovo LOQ 15IAX9)

**Ngày: 25/06/2026 — Kernel 7.x**

---

## Hàng ngày

- [ ] Dùng máy bình thường.

---

## Hàng tuần

- [ ] Đọc tin tức Arch: <https://archlinux.org/news/>
- [ ] Cập nhật hệ thống: `yay -Syu`
- [ ] Kiểm tra lỗi: `systemctl --failed`
- [ ] Kiểm tra dung lượng: `df -h /`

---

## Hàng tháng

- [ ] Dọn cache pacman (cần `pacman-contrib`):
  ```bash
  paccache -rk1
  ```
- [ ] Xoá orphan packages:
  ```bash
  pacman -Qtdq | wc -l
  # Nếu > 0
  sudo pacman -Rns $(pacman -Qtdq)
  ```
- [ ] Dọn journal (cần `sudo`):
  ```bash
  sudo journalctl --vacuum-size=100M
  ```
- [ ] Dọn cache yay:
  ```bash
  yay -Sc --noconfirm
  ```
- [ ] Kiểm tra BTRFS:
  ```bash
  sudo btrfs filesystem usage /
  ```
- [ ] Kiểm tra Timeshift:
  ```bash
  sudo timeshift --list
  ```

### Script dọn dẹp một lần

```bash
#!/bin/bash
echo "=== Dọn pacman cache ==="
paccache -rk1
echo "=== Xoá orphan packages ==="
sudo pacman -Qtdq && sudo pacman -Rns $(sudo pacman -Qtdq) || echo "Không có orphans"
echo "=== Dọn yay cache ==="
yay -Sc --noconfirm
echo "=== Dọn journal ==="
sudo journalctl --vacuum-size=100M
echo "=== Hoàn tất ==="
```

---

## Hàng quý (3 tháng)

- [ ] Kiểm tra NVMe health (cần `nvme-cli`):
  ```bash
  sudo nvme smart-log /dev/nvme0n1
  ```
- [ ] Kiểm tra pin:
  ```bash
  upower -i /org/freedesktop/UPower/devices/battery_BAT0
  ```
- [ ] Kiểm tra BTRFS chi tiết:
  ```bash
  sudo btrfs filesystem show
  sudo btrfs device stats /
  ```
- [ ] Xoá snapshot cũ (giữ 1-2 cái gần nhất):
  ```bash
  sudo timeshift --delete --snapshot 'TÊN-SNAPSHOT'
  ```
- [ ] Backup danh sách gói:
  ```bash
  mkdir -p ~/backup
  pacman -Qqen > ~/backup/pkglist-official-$(date +%Y%m%d).txt
  pacman -Qqem > ~/backup/pkglist-aur-$(date +%Y%m%d).txt
  ```
- [ ] Backup thư mục config:
  ```bash
  tar -czf ~/backup/config-$(date +%Y%m%d).tar.gz \
    ~/.config/{bspwm,sxhkd,polybar,rofi,picom,alacritty,nvim}
  ```
- [ ] Vệ sinh máy: lau bụi quạt, thay thermal paste nếu nhiệt độ cao.

---

## Hàng năm

- [ ] Kiểm tra sức khoẻ ổ cứng (S.M.A.R.T.):
  ```bash
  sudo smartctl -a /dev/nvme0n1
  ```
- [ ] Cân nhắc thay pin nếu health < 80%.
- [ ] Cài lại hệ thống nếu cần (dùng config đã backup).

---

## Lưu ý khi update

### Trước update

```bash
# Backup danh sách gói
pacman -Qqen > ~/backup/pkglist-official.txt
pacman -Qqem > ~/backup/pkglist-aur.txt

# Tạo Timeshift snapshot
sudo timeshift --create --comments "truoc-update-$(date +%Y%m%d)"
```

### Trong khi update

- Đọc kỹ danh sách gói sẽ cập nhật.
- Chú ý các gói quan trọng: **linux, systemd, nvidia-open, grub**.
- Không ngắt quá trình `pacman -Syu` giữa chừng.

### Sau update

```bash
# Kiểm tra kernel
uname -r

# Kiểm tra lỗi
dmesg | grep -i error
journalctl -p 3 -xb | tail -10

# Nếu update kernel → reboot
sudo reboot
```

---

## Dấu hiệu cần can thiệp

| Dấu hiệu | Nguyên nhân thường gặp | Hành động |
|---|---|---|
| Disk đầy (>90%) | Cache pacman, journal, orphan | `paccache -rk1`, `sudo journalctl --vacuum-size=100M` |
| Nhiệt độ cao (>80°C idle) | Bụi quạt, thermal paste khô | Vệ sinh, thay thermal paste |
| Boot chậm | Service không cần thiết | `systemctl list-unit-files \| grep enabled` |
| Ứng dụng crash | Lỗi thư viện, cấu hình sai | Kiểm tra log, reinstall |
| Màn hình nhấp nháy | PSR (Panel Self Refresh) | Thêm `i915.enable_psr=0` vào kernel params |
| Wi-Fi mất kết nối | Driver Realtek | `sudo modprobe -r 8852be && sudo modprobe 8852be` |
| NVIDIA không hoạt động | Module không load | `sudo modprobe nvidia-drm`, kiểm tra `nvidia-open` |
