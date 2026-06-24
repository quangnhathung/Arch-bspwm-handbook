# Dọn dẹp gói

## Mục tiêu

Giữ hệ thống sạch sẽ bằng cách xóa cache, orphan packages, journal cũ.

## Kiến thức nền

### Tại sao cần dọn dẹp?

- Cache pacman có thể lên đến vài GB.
- Orphan packages (gói mồ côi) chiếm dung lượng và có thể gây xung đột.
- Journald log tích tụ theo thời gian.
- Dọn dẹp = bảo trì hệ thống.

### Orphan packages là gì?

Gói được cài tự động làm dependency của một gói khác, nhưng gói đó đã bị xóa.
Chúng không còn được sử dụng bởi bất kỳ gói nào.

## 1. Dọn cache pacman

### paccache (cần pacman-contrib)

`paccache` nằm trong gói `pacman-contrib`, không có sẵn khi cài Arch cơ bản.

```bash
# Cài pacman-contrib nếu chưa có
pacman -S pacman-contrib

# Giữ 3 phiên bản gần nhất (mặc định)
paccache -r

# Giữ 1 phiên bản (tiết kiệm nhất)
paccache -rk1

# Xem kích thước cache
du -sh /var/cache/pacman/pkg/
```

### Xóa toàn bộ cache

```bash
pacman -Scc
```

Chỉ dùng khi thực sự cần dung lượng. Sẽ mất khả năng downgrade.

## 2. Xóa orphan packages

```bash
# Kiểm tra orphan
pacman -Qtd

# Xóa orphan (nếu có)
pacman -Rns $(pacman -Qtdq)
```

Giải thích:
- `-Qtd`: Query packages không cần thiết (orphan) + không có gói nào phụ thuộc.
- `-Qtdq`: Chỉ xuất tên gói (quiet).
- `$(...)`: Dùng output làm input cho pacman.

### Orphan AUR

```bash
yay -Yc
```

## 3. Dọn cache yay

```bash
# Xóa các PKGBUILD đã clone
yay -Sc
```

## 4. Dọn journal (cần sudo)

```bash
# Xem dung lượng journal
journalctl --disk-usage

# Giới hạn dung lượng (cần sudo)
sudo journalctl --vacuum-size=100M

# Giới hạn thời gian (cần sudo)
sudo journalctl --vacuum-time=2weeks
```

**Lưu ý**: `journalctl --vacuum-*` cần `sudo` để thực thi.

## 5. Script dọn dẹp tự động

Tạo script `/usr/local/bin/cleanup-arch`:

```bash
#!/bin/bash
# System cleanup script
# Ngày: 25/06/2026

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
sudo journalctl --vacuum-size=100M 2>/dev/null

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

### Systemd timer (chạy hàng tuần)

`/etc/systemd/system/cleanup.timer`:

```ini
[Unit]
Description=Weekly cleanup

[Timer]
OnCalendar=weekly
Persistent=true

[Install]
WantedBy=timers.target
```

`/etc/systemd/system/cleanup.service`:

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

## 6. Phân tích dung lượng ổ đĩa

```bash
# Tổng quan
df -h /

# Thư mục lớn nhất
du -sh /* 2>/dev/null | sort -rh | head -10

# Cụ thể từng thư mục
du -sh /var/* 2>/dev/null | sort -rh | head -10
du -sh /home/* 2>/dev/null | sort -rh | head -10

# Dùng ncdu (giao diện TUI)
pacman -S ncdu
ncdu /
```

## Lịch dọn dẹp

| Task | Lệnh | Tần suất |
|---|---|---|
| Cache pacman | `paccache -rk1` | Hàng tháng |
| Orphan packages | `pacman -Rns $(pacman -Qtdq)` | Hàng tháng |
| Cache yay | `yay -Sc` | Hàng tháng |
| Journal | `sudo journalctl --vacuum-size=100M` | Hàng tháng |
| Phân tích ổ đĩa | `ncdu /` | Hàng quý |

## Troubleshooting

### "failed to prepare transaction (could not satisfy dependencies)"

```bash
# Kiểm tra orphan còn sót
pacman -Qtd

# Xóa thủ công
pacman -Rns tên-gói
```

### Cache quá lớn, paccache không xóa được

```bash
# Xóa trực tiếp
rm -rf /var/cache/pacman/pkg/*.pkg.tar.zst
```

### "paccache: command not found"

```bash
# Cài pacman-contrib
pacman -S pacman-contrib
```

### journalctl không có tác dụng

```bash
# Nhớ thêm sudo
sudo journalctl --vacuum-size=100M
```

## Tổng kết

- `paccache` cần gói `pacman-contrib`.
- Xóa orphan packages bằng `pacman -Rns $(pacman -Qtdq)`.
- `journalctl --vacuum-*` cần sudo.
- Tự động hóa với script + systemd timer.
- Kiểm tra dung lượng định kỳ bằng `ncdu` hoặc `du`.
