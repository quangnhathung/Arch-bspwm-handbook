# Pacman — Trình quản lý gói

## Mục tiêu

Hiểu và sử dụng pacman — trình quản lý gói của Arch Linux.

## Giới thiệu

Pacman (Package Manager) là công cụ quản lý gói của Arch Linux.
Nó quản lý cài đặt, cập nhật, xóa gói phần mềm.

Pacman sử dụng format `.pkg.tar.zst` (tar compressed với zstd).

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

### Cập nhật hệ thống

```bash
# Đồng bộ database và cập nhật tất cả gói
pacman -Syu
```

- `-S`: Sync (đồng bộ database).
- `-y`: Refresh database.
- `-u`: Upgrade (cập nhật gói).

**Luôn chạy `pacman -Syu` trước khi cài gói mới.**

### Cài gói

```bash
# Cài một gói
pacman -S gói

# Cài nhiều gói
pacman -S gói1 gói2 gói3

# Cài gói từ một nhóm
pacman -S nhóm
```

### Tìm kiếm gói

```bash
# Tìm trong database (chưa cài)
pacman -Ss từ_khóa

# Tìm trong gói đã cài
pacman -Qs từ_khóa
```

### Xem thông tin gói

```bash
# Thông tin gói trong database
pacman -Si gói

# Thông tin gói đã cài
pacman -Qi gói

# File của gói đã cài
pacman -Ql gói

# Gói nào sở hữu file
pacman -Qo /path/to/file
```

### Xóa gói

```bash
# Xóa gói (giữ config)
pacman -R gói

# Xóa gói + dependencies không cần thiết
pacman -Rs gói

# Xóa gói + config + dependencies
pacman -Rns gói
```

### Database

```bash
# Refresh database
pacman -Sy

# Force refresh (nếu mirror lỗi)
pacman -Syy
```

## Các option hữu ích

| Option | Mô tả |
|---|---|
| `--noconfirm` | Không hỏi xác nhận |
| `--needed` | Chỉ cài nếu chưa có |
| `--overwrite='*'` | Ghi đè file xung đột (cẩn thận) |
| `--cachedir` | Chỉ định thư mục cache |
| `--dbonly` | Chỉ cập nhật database, không cài file |

## Cache pacman

Pacman lưu các gói đã tải trong `/var/cache/pacman/pkg/`.
Đây là thư mục được mount là subvolume `@pkg`.

### Dọn cache

```bash
# Xóa cache cũ (giữ 3 phiên bản gần nhất)
paccache -r

# Xóa tất cả cache
pacman -Scc
```

### Xem kích thước cache

```bash
du -sh /var/cache/pacman/pkg/
```

## Cấu hình pacman

File cấu hình: `/etc/pacman.conf`

### Parallel downloads

```bash
vim /etc/pacman.conf
```

Tìm và bỏ comment (sửa số):

```
ParallelDownloads = 5
```

### Mirror list

File: `/etc/pacman.d/mirrorlist`

Chọn mirror gần Việt Nam nhất. Arch Linux có mirror ở Việt Nam:

```bash
# Cập nhật mirror list tự động
pacman -S reflector

# Tìm 5 mirror gần nhất, theo tốc độ
reflector --country Vietnam --latest 5 --sort rate --save /etc/pacman.d/mirrorlist
```

## Troubleshooting

### "failed to commit transaction (conflicting files)"

Một số file xung đột với gói khác:

```bash
# Xem file nào xung đột
pacman -S --overwrite='*' gói
```

### "database is locked"

Một tiến trình pacman khác đang chạy:

```bash
# Xóa lock
rm /var/lib/pacman/db.lck
```

### "error: key could not be looked up remotely"

```bash
# Refresh keyring
pacman -Sy archlinux-keyring
```

## Best practices

1. **Luôn đọc tin tức trước khi update**: https://archlinux.org/news/
2. **Không dùng `--force`** (đã bị deprecated).
3. **Kiểm tra pacman -Qtd** để xóa orphan packages.
4. **Không can thiệp vào pacman khi đang chạy**.

## Tổng kết

- Pacman là công cụ quản lý gói mạnh mẽ và đơn giản.
- Lệnh quan trọng nhất: `pacman -Syu` (update), `-S` (install), `-Rs` (remove), `-Ss` (search).
- Cache pacman được quản lý riêng (subvolume @pkg).
- Cấu hình parallel downloads và mirror cho tốc độ tốt hơn.
