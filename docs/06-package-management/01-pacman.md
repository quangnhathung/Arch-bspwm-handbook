# Pacman — Trình quản lý gói

## Mục tiêu

Hiểu và sử dụng pacman — trình quản lý gói của Arch Linux.

## Giới thiệu

Pacman (Package Manager) là công cụ quản lý gói của Arch Linux.
Nó quản lý cài đặt, cập nhật, xóa gói phần mềm.

Pacman sử dụng định dạng `.pkg.tar.zst` (tar nén với zstd).

Ngày viết: 25/06/2026, kernel 7.x.

## Cấu trúc lệnh pacman

```
pacman <tác vụ> [options] [gói]
```

### Các tác vụ chính

| Tác vụ | Option | Mô tả |
|---|---|---|
| Synchronize | `-S` | Cài gói từ repository |
| Remove | `-R` | Xóa gói |
| Query | `-Q` | Truy vấn gói đã cài |
| Files | `-F` | Truy vấn file thuộc gói nào |
| Upgrade | `-U` | Cài từ file `.pkg.tar.zst` |

## Các lệnh thường dùng

### Cập nhật hệ thống (`-Syu`)

```bash
pacman -Syu
```

- `-S`: Sync (đồng bộ database).
- `-y`: Refresh database.
- `-u`: Upgrade (cập nhật gói).

**Luôn chạy `pacman -Syu` trước khi cài gói mới.**
**Cảnh báo: Không bao giờ chạy `pacman -Sy` mà không có `-u` — đây là partial upgrade, có thể gây hỏng hệ thống.**

### Cài gói (`-S`)

```bash
# Cài một gói
pacman -S gói

# Cài nhiều gói
pacman -S gói1 gói2 gói3

# Cài gói từ một nhóm
pacman -S nhóm
```

### Xóa gói (`-R`)

```bash
# Xóa gói (giữ cấu hình)
pacman -R gói

# Xóa gói + dependencies không cần thiết
pacman -Rs gói

# Xóa gói + cấu hình + dependencies
pacman -Rns gói
```

### Truy vấn gói đã cài (`-Q`)

```bash
# Liệt kê tất cả gói đã cài
pacman -Q

# Tìm gói đã cài theo từ khóa
pacman -Qs từ_khóa

# Thông tin chi tiết gói đã cài
pacman -Qi gói

# File của gói đã cài
pacman -Ql gói

# Gói nào sở hữu file này?
pacman -Qo /usr/bin/firefox
```

### Tìm gói trong database (`-F`)

```bash
# Tìm gói nào sở hữu file (không cần cài)
pacman -F /path/to/file

# Cập nhật database của -F
pacman -Fy
```

### Cài từ file (`-U`)

```bash
# Cài gói từ file .pkg.tar.zst (downgrade hoặc offline)
pacman -U /var/cache/pacman/pkg/gói-cũ.pkg.tar.zst
```

## Quản lý cache

Pacman lưu các gói đã tải trong `/var/cache/pacman/pkg/`.

### paccache (từ pacman-contrib)

```bash
# Cài pacman-contrib nếu chưa có
pacman -S pacman-contrib

# Giữ 3 phiên bản gần nhất (mặc định)
paccache -r

# Giữ 1 phiên bản gần nhất
paccache -rk1

# Xem kích thước cache
du -sh /var/cache/pacman/pkg/
```

`paccache` nằm trong gói `pacman-contrib`, không có sẵn khi cài Arch cơ bản.
Cần cài riêng:

```bash
pacman -S pacman-contrib
```

### Xóa toàn bộ cache (cẩn thận)

```bash
pacman -Scc
```

Chỉ dùng khi cần gấp dung lượng. Sẽ mất khả năng downgrade từ cache.

## Quản lý mirror

File cấu hình mirror: `/etc/pacman.d/mirrorlist`

### Dùng reflector (tự động)

```bash
# Cài reflector
pacman -S reflector

# Tìm 5 mirror gần nhất theo tốc độ
reflector --country Vietnam --latest 5 --sort rate --save /etc/pacman.d/mirrorlist
```

### Rating thủ công

Mở file mirrorlist và di chuyển mirror gần nhất lên đầu:

```bash
vim /etc/pacman.d/mirrorlist
```

Mirror Việt Nam (ví dụ): `fpt.vn`, `viettelnap.vn`.

## Cấu hình pacman

File: `/etc/pacman.conf`

### Parallel downloads

```ini
ParallelDownloads = 5
```

Bỏ comment và đặt số luồng tải song song.

### Color

```ini
Color
```

Hiển thị màu trong output.

### VerbosePkgLists

```ini
VerbosePkgLists
```

Hiện danh sách gói chi tiết khi cập nhật.

### ILoveCandy

```ini
ILoveCandy
```

Progress bar dạng candy (tùy chọn, cho vui).

## Partial upgrade — Cảnh báo nguy hiểm

**Không bao giờ** chạy `pacman -Sy` (chỉ refresh database mà không upgrade).

Hậu quả: Cài gói mới lên hệ thống với database mới nhưng gói cũ chưa được cập nhật → xung đột thư viện → hỏng hệ thống.

```bash
# SAI — nguy hiểm:
pacman -Sy firefox

# ĐÚNG:
pacman -Syu firefox
```

Nếu lỡ chạy `pacman -Sy`, chạy ngay `pacman -Su` để đồng bộ lại.

## Troubleshooting

### "failed to commit transaction (conflicting files)"

```bash
pacman -S --overwrite='*' gói
```

### "database is locked"

Xóa lock file:

```bash
rm /var/lib/pacman/db.lck
```

### "error: key could not be looked up remotely"

```bash
pacman -Sy archlinux-keyring
```

## Best practices

1. **Đọc tin tức Arch trước khi update lớn**: https://archlinux.org/news/
2. **Không dùng `--force`** (đã bị deprecated).
3. **Xóa orphan packages định kỳ**: `pacman -Qtdq`.
4. **Không can thiệp pacman khi đang chạy**.
5. **Cài `pacman-contrib` để dùng `paccache`**.
6. **Luôn snapshot trước update lớn**.

## Tổng kết

- Pacman quản lý gói với các tác vụ `-S` (cài), `-R` (xóa), `-Q` (truy vấn), `-F` (file), `-U` (file).
- Lệnh quan trọng nhất: `pacman -Syu` (cập nhật toàn bộ).
- Không bao giờ partial upgrade.
- Dọn cache với `paccache` (cần `pacman-contrib`).
- ParallelDownloads và mirror tốt giúp tăng tốc.
