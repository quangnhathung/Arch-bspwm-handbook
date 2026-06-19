# Bảo trì hệ thống

## Mục tiêu

Hiểu quy trình bảo trì Arch Linux để giữ hệ thống ổn định và an toàn.

## Nguyên tắc

Arch Linux là rolling release → cập nhật liên tục. Bảo trì quan trọng hơn
các bản phân phối cố định (Ubuntu, Fedora) vì một lần update hỏng có thể
phá vỡ hệ thống.

## Quy trình bảo trì hàng tuần

### 1. Đọc tin tức Arch

Trước khi update, kiểm tra tin tức:

```bash
# Mở trang news
xdg-open https://archlinux.org/news/

# Hoặc dùng CLI
curl -s https://archlinux.org/news/ | grep -oP '(?<=<title>)[^<]+' | head -5
```

Arch thường đăng cảnh báo trước khi có thay đổi lớn (kernel, systemd, v.v.).

### 2. Cập nhật hệ thống

```bash
# Update đầy đủ
yay -Syu

# Chú ý:
# - Nếu có "warning: ... skipping" → đọc kỹ
# - Nếu có pacman news → đọc trước
# - Nếu update kernel → reboot sau
```

### 3. Kiểm tra sau update

```bash
# Kiểm tra lỗi
systemctl --failed

# Kiểm tra log kernel
dmesg | grep -i error | tail -10

# Kiểm tra disk space
df -h /
```

### 4. Dọn dẹp (theo lịch)

Xem bài package-cleanup.md.

## Bảo trì hàng tháng

### 1. Kiểm tra orphan packages

```bash
pacman -Qtdq | wc -l
# Nếu > 0 → xóa
pacman -Rns $(pacman -Qtdq)
```

### 2. Kiểm tra journal

```bash
journalctl --disk-usage
# Nếu > 200MB → vacuum
journalctl --vacuum-size=100M
```

### 3. Kiểm tra BTRFS

```bash
# Kiểm tra dung lượng subvolume
btrfs filesystem usage /

# Kiểm tra lỗi
btrfs device stats /
```

### 4. Kiểm tra pacman cache

```bash
du -sh /var/cache/pacman/pkg/
# Nếu > 2GB → paccache -rk 1
```

### 5. Snapshot trước update lớn

```bash
# Nếu dùng Timeshift
sudo timeshift --create --comments "before-update-$(date +%Y%m%d)"
```

## Bảo trì 6 tháng / năm

### 1. Kiểm tra S.M.A.R.T (NVMe health)

```bash
pacman -S nvme-cli
sudo nvme smart-log /dev/nvme0n1
```

Kiểm tra:
- `temperature`: < 60°C là tốt.
- `percentage_used`: < 100%.
- `media_errors`: phải là 0.

### 2. Kiểm tra battery health

```bash
pacman -S acpi
acpi -i
```

### 3. Vệ sinh vật lý

- Lau quạt laptop.
- Thay thermal paste (nếu cần).
- Kiểm tra ốc vít.

## Những việc KHÔNG nên làm

| Không nên | Vì sao |
|---|---|
| `pacman -Syu --force` | Đã bị deprecated, có thể hỏng hệ thống |
| `pacman -Sy` (không -u) | Partial upgrade → thường gây conflict |
| Xóa cache tất cả (`-Scc`) | Mất file cài, không rollback được |
| Tắt bảo vệ filesystem | Check `--overwrite='*'` quá mức |
| Update kernel khi đang dùng máy | Cần reboot, mất work chưa save |

## Partial upgrade là gì?

Partial upgrade là cập nhật một số gói mà không cập nhật toàn bộ.
Ví dụ: chỉ cài firefox mới mà không cập nhật thư viện phụ thuộc.

**Rất nguy hiểm**: có thể gây broken dependencies.

```bash
# KHÔNG làm thế này:
pacman -S firefox  # Không chạy -Syu trước

# LUÔN làm thế này:
pacman -Syu
# Hoặc
pacman -Syu firefox
```

## Kernel update

Khi kernel được cập nhật:

1. `mkinitcpio -P` tự động chạy (nếu dùng linux).
2. Cần reboot để dùng kernel mới.
3. Kiểm tra kernel hiện tại: `uname -r`.

## Backup trước update lớn

### Snapshot BTRFS (nhanh nhất)

```bash
sudo timeshift --create --comments "truoc-update-thang-6"
```

### Backup cấu hình

```bash
# Backup danh sách gói
pacman -Qqen > ~/backup/pkglist.txt
pacman -Qqem > ~/backup/pkglist-aur.txt

# Backup config quan trọng
mkdir -p ~/backup/etc
sudo cp /etc/default/grub ~/backup/etc/
sudo cp /etc/fstab ~/backup/etc/
sudo cp /etc/pacman.conf ~/backup/etc/
```

## Khi update bị lỗi

1. **Đọc log lỗi**: `journalctl -p 3 -xb`.
2. **Không reboot ngay** nếu chưa chắc chắn.
3. **Rollback snapshot** nếu có.
4. **Vào Arch forum / Reddit**: tìm lỗi tương tự.
5. **Downgrade gói** nếu cần:

```bash
# Cài từ cache
pacman -U /var/cache/pacman/pkg/gói-cũ.pkg.tar.zst
```

## Tổng kết

- Update thường xuyên (hàng tuần).
- Đọc tin tức Arch trước update lớn.
- Dọn dẹp định kỳ.
- Snapshot trước update lớn.
- Không partial upgrade.
- Biết cách rollback khi có lỗi.
