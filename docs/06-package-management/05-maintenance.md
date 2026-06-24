# Bảo trì hệ thống

## Mục tiêu

Hiểu quy trình bảo trì Arch Linux để giữ hệ thống ổn định và an toàn.

## Nguyên tắc

Arch Linux là rolling release → cập nhật liên tục.
Bảo trì quan trọng hơn các bản phân phối cố định vì một lần update hỏng
có thể phá vỡ hệ thống.

Ngày: 25/06/2026, kernel 7.x.

## Hàng tuần

### 1. Đọc tin tức Arch

Trước khi update, kiểm tra tin tức:

```bash
# Mở trang news
xdg-open https://archlinux.org/news/

# Hoặc CLI
curl -s https://archlinux.org/news/ | grep -oP '(?<=<title>)[^<]+' | head -5
```

Arch thường đăng cảnh báo trước thay đổi lớn (kernel, systemd, thư viện).

### 2. Cập nhật hệ thống

```bash
yay -Syu
```

Lưu ý:
- Nếu có "warning: ... skipping" → đọc kỹ trước khi tiếp tục.
- Nếu có pacman news → đọc trước.
- Nếu update kernel → reboot sau.
- **Không bao giờ** chạy `pacman -Sy` mà không có `-u`.

### 3. Kiểm tra sau update

```bash
# Dịch vụ lỗi
systemctl --failed

# Log kernel
dmesg | grep -i error | tail -10

# Dung lượng ổ
df -h /
```

### 4. Snapshot trước update lớn

```bash
sudo timeshift --create --comments "truoc-update-tuan-$(date +%Y%m%d)"
```

## Hàng tháng

### 1. Dọn dẹp cache và orphan

```bash
paccache -rk1
pacman -Rns $(pacman -Qtdq) --noconfirm
yay -Sc --noconfirm
sudo journalctl --vacuum-size=100M
```

### 2. Kiểm tra BTRFS

```bash
# Dung lượng subvolume
btrfs filesystem usage /

# Thống kê lỗi thiết bị
btrfs device stats /

# Kiểm tra checksum
btrfs scrub start /
btrfs scrub status /
```

### 3. Kiểm tra dung lượng cache

```bash
du -sh /var/cache/pacman/pkg/
```

Nếu > 2GB → `paccache -rk1`.

### 4. Tạo snapshot hàng tháng

```bash
sudo timeshift --create --comments "thang-$(date +%Y%m)"
```

## Hàng quý

### 1. Kiểm tra NVMe health

```bash
pacman -S nvme-cli
sudo nvme smart-log /dev/nvme0n1
```

Kiểm tra:
- `temperature`: < 60°C.
- `percentage_used`: < 100%.
- `media_errors`: phải là 0.
- `power_cycles`: bình thường.

### 2. Kiểm tra battery health

```bash
pacman -S acpi
acpi -i

# Hoặc chi tiết hơn
cat /sys/class/power_supply/BAT0/energy_full_design
cat /sys/class/power_supply/BAT0/energy_full
cat /sys/class/power_supply/BAT0/energy_now
```

Tính health: `(energy_full / energy_full_design) × 100%`.

### 3. Backup cấu hình

```bash
# Danh sách gói
pacman -Qqen > ~/backup/pkglist-official-$(date +%Y%m).txt
pacman -Qqem > ~/backup/pkglist-aur-$(date +%Y%m).txt

# Cấu hình hệ thống
sudo cp /etc/default/grub ~/backup/etc/
sudo cp /etc/fstab ~/backup/etc/
sudo cp /etc/pacman.conf ~/backup/etc/

# Cấu hình người dùng
cp -r ~/.config/bspwm ~/backup/config/
cp -r ~/.config/sxhkd ~/backup/config/
cp -r ~/.config/polybar ~/backup/config/
cp -r ~/.config/alacritty ~/backup/config/
```

## Những việc KHÔNG nên làm

| Không nên | Vì sao |
|---|---|
| `pacman -Sy` (không `-u`) | Partial upgrade → hỏng dependencies |
| `pacman -Syu --force` | Đã bị deprecated, nguy hiểm |
| Xóa toàn bộ cache `-Scc` | Mất khả năng downgrade |
| Tắt hệ thống giữa lúc update | Hỏng gói đang cài dở |
| Update kernel không reboot | Kernel cũ vẫn chạy |

## Partial upgrade — Giải thích chi tiết

Partial upgrade = cập nhật một số gói mà không cập nhật toàn bộ.

```bash
# SAI — partial upgrade
pacman -Sy firefox

# ĐÚNG
pacman -Syu
# hoặc
pacman -Syu firefox
```

**Hậu quả**: Gói mới yêu cầu thư viện mới hơn gói hiện tại → không chạy được.
Arch không hỗ trợ partial upgrade. Nếu lỡ làm, chạy `pacman -Su` ngay.

## Downgrade gói

Khi bản mới bị lỗi, có thể cài bản cũ từ cache:

```bash
# Liệt kê cache
ls /var/cache/pacman/pkg/ | grep firefox

# Cài bản cũ
pacman -U /var/cache/pacman/pkg/firefox-xxx.pkg.tar.zst
```

Thêm vào `/etc/pacman.conf` để giữ nhiều phiên bản:

```ini
# Keep at least 3 versions
CleanMethod = KeepCurrent
```

Hoặc cài từ Arch Archive:

```bash
# Ví dụ downgrade linux kernel
wget https://archive.archlinux.org/packages/l/linux/linux-xxx.pkg.tar.zst
pacman -U linux-xxx.pkg.tar.zst
```

## Kernel update

Khi kernel được cập nhật:

1. `mkinitcpio -P` tự động chạy (với linux).
2. Cần reboot để dùng kernel mới.
3. Kiểm tra: `uname -r` → kernel 7.x.
4. Giữ kernel cũ trong GRUB đề phòng (Advanced Options).

## Snapshot trước update lớn

**Luôn tạo snapshot trước khi update**, đặc biệt là:
- Kernel update.
- Systemd update.
- Database update (mariadb, postgresql).
- Desktop environment update.

```bash
sudo timeshift --create --comments "truoc-kernel-update-$(date +%Y%m%d)"
```

## Khi update bị lỗi

1. **Đọc log**: `journalctl -p 3 -xb`.
2. **Không reboot** nếu chưa chắc chắn.
3. **Rollback snapshot** nếu có.
4. **Downgrade gói lỗi** từ cache.
5. **Vào Arch forum / Reddit** tìm lỗi tương tự.

## Lịch bảo trì tóm tắt

| Tần suất | Công việc |
|---|---|
| Hàng tuần | Đọc news, `yay -Syu`, `systemctl --failed`, `dmesg` |
| Hàng tháng | Dọn cache, orphan, journal, BTRFS scrub, snapshot |
| Hàng quý | NVMe health, battery health, backup config |

## Tổng kết

- Update thường xuyên (hàng tuần) — đừng để lâu quá 1 tháng.
- Đọc tin tức Arch trước update lớn.
- Không partial upgrade — luôn `pacman -Syu`.
- Snapshot trên BTRFS trước update lớn.
- Biết cách downgrade và rollback khi có lỗi.
- Bảo trì định kỳ giúp hệ thống bền vững.
