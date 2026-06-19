# Dọn dẹp gói

## Mục tiêu

Giữ hệ thống sạch sẽ bằng cách xóa các gói và file không cần thiết.

## Kiến thức nền

### Tại sao cần dọn dẹp?

- Cache pacman có thể lên đến vài GB.
- Orphan packages (gói mồ côi) chiếm dung lượng.
- Gói không dùng làm chậm hệ thống (dependencies check).
- Dọn dẹp = bảo trì hệ thống.

### Orphan packages là gì?

Orphan packages là gói được cài tự động như dependency của một gói khác,
nhưng gói đó đã bị xóa. Chúng không còn được sử dụng bởi bất kỳ gói nào.

## Các bước dọn dẹp

### Bước 1: Xóa cache pacman

```bash
# Giữ 3 phiên bản gần nhất của mỗi gói
paccache -r

# Giữ 1 phiên bản
paccache -rk 1

# Xóa tất cả cache (không khuyên dùng)
pacman -Scc
```

### Bước 2: Xóa orphan packages

```bash
# Kiểm tra orphan
pacman -Qtd

# Xóa orphan (nếu có)
pacman -Rns $(pacman -Qtdq)
```

Giải thích:
- `-Qtd`: Query packages không cần thiết (orphan) + không có gói nào phụ thuộc.
- `-Qtdq`: Chỉ xuất tên gói.
- `$(...)`: Command substitution — kết quả làm input cho pacman -Rns.

### Bước 3: Xóa cache yay

```bash
# Xóa các PKGBUILD đã clone
yay -Sc
```

### Bước 4: Xóa journal cũ

```bash
# Xem dung lượng journal
journalctl --disk-usage

# Giữ journal 100MB
journalctl --vacuum-size=100M

# Giữ journal 2 tuần
journalctl --vacuum-time=2weeks
```

### Bước 5: Xóa log cũ

```bash
# Xem dung lượng log
du -sh /var/log/

# Xóa log journal
sudo journalctl --vacuum-time=1month

# Xóa log cũ trong /var/log (cẩn thận)
find /var/log -name "*.log.*" -mtime +30 -delete
```

## Tự động dọn dẹp

### Tạo script dọn dẹp

```bash
vim /usr/local/bin/cleanup-arch
```

Nội dung:

```bash
#!/bin/bash
# System cleanup script

echo "=== Cleaning pacman cache ==="
paccache -rk1

echo "=== Removing orphan packages ==="
if pacman -Qtdq &>/dev/null; then
    pacman -Rns $(pacman -Qtdq) --noconfirm
else
    echo "No orphan packages found."
fi

echo "=== Cleaning yay cache ==="
yay -Sc --noconfirm 2>/dev/null || true

echo "=== Cleaning journal ==="
journalctl --vacuum-size=100M 2>/dev/null

echo "=== Done ==="
```

Cấp quyền:

```bash
chmod +x /usr/local/bin/cleanup-arch
```

Chạy:

```bash
sudo cleanup-arch
```

### Cron job (nếu có cron)

Hoặc dùng systemd timer:

```bash
vim /etc/systemd/system/cleanup.timer
```

Nội dung:

```ini
[Unit]
Description=Weekly cleanup

[Timer]
OnCalendar=weekly
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
vim /etc/systemd/system/cleanup.service
```

Nội dung:

```ini
[Unit]
Description=System cleanup

[Service]
Type=oneshot
ExecStart=/usr/local/bin/cleanup-arch
```

Enable:

```bash
systemctl enable --now cleanup.timer
```

## Xem dung lượng hệ thống

```bash
# Tổng quan
du -sh / 2>/dev/null | sort -rh

# Thư mục lớn
du -sh /* 2>/dev/null | sort -rh | head -10

# Xem cụ thể thư mục nào lớn
du -sh /var/* 2>/dev/null | sort -rh | head -10
du -sh /home/* 2>/dev/null | sort -rh | head -10
```

## Kiểm tra định kỳ

| Task | Tần suất |
|---|---|
| `paccache -r` | Hàng tháng |
| Xóa orphan | Hàng tháng |
| `journalctl --vacuum-size=100M` | Hàng tháng |
| `yay -Sc` | Hàng tháng |
| Kiểm tra dung lượng ổ | Hàng tháng |

## Troubleshooting

### "failed to prepare transaction (could not satisfy dependencies)"

```bash
# Kiểm tra xung đột
pacman -Qtd

# Xóa thủ công từng cái
pacman -Rns gói
```

### "error: target not found"

Gói không tồn tại trong database → kiểm tra tên.

### Cache quá lớn (paccache không xóa được)

```bash
# Xóa trực tiếp
rm -rf /var/cache/pacman/pkg/*.pkg.tar.zst
```

## Tổng kết

- Xóa cache pacman định kỳ.
- Xóa orphan packages.
- Xóa journal cũ.
- Script tự động cleanup.
- Kiểm tra dung lượng định kỳ để phát hiện sớm.
