# Backup Strategy

## Mục tiêu

Xây dựng chiến lược backup thực tế cho desktop cá nhân.

## Kiến thức nền

### Tại sao cần backup riêng ngoài snapshot?

Snapshot BTRFS chỉ bảo vệ khỏi lỗi người dùng (xóa nhầm, cập nhật hỏng).
Nó không bảo vệ khỏi:

- **Hỏng ổ cứng**: Nếu NVMe chết, snapshot cũng mất.
- **Mất máy / trộm**: Mất toàn bộ dữ liệu vật lý.
- **Lỗi filesystem**: Dù hiếm, BTRFS có thể gặp lỗi nghiêm trọng.

### 3-2-1 Rule (chuẩn)

- **3** bản sao dữ liệu.
- **2** loại media khác nhau (ổ trong, ổ ngoài).
- **1** bản ở nơi khác (offsite).

## Chiến lược cho desktop cá nhân

### Dữ liệu cần backup

| Mức độ | Dữ liệu | Dung lượng | Tần suất backup |
|---|---|---|---|
| Quan trọng | Documents, Pictures, Config | Nhỏ (< 10GB) | Hàng tuần |
| Quan trọng | Password manager database | Rất nhỏ | Hàng ngày |
| Trung bình | Browser profile | Vài GB | Hàng tháng |
| Thấp | Cache, log, packages | Lớn | Không cần backup |
| Không cần | Hệ thống (có thể cài lại) | — | Snapshot là đủ |

### Backup config (dễ nhất)

Các file cấu hình đều nằm trong `~/.config/` và `/etc/`.

```bash
# Backup config vào thư mục riêng
mkdir -p ~/backup/config

# Copy config
cp -r ~/.config/bspwm ~/backup/config/
cp -r ~/.config/sxhkd ~/backup/config/
cp -r ~/.config/polybar ~/backup/config/
cp -r ~/.config/alacritty ~/backup/config/

# System config (cần sudo)
sudo cp /etc/default/grub ~/backup/config/
sudo cp /etc/fstab ~/backup/config/
```

### Backup danh sách gói

```bash
# Danh sách gói chính thức
pacman -Qqen > ~/backup/pkglist-official.txt

# Danh sách gói AUR
pacman -Qqem > ~/backup/pkglist-aur.txt
```

Để restore sau khi cài lại:

```bash
# Cài gói chính thức
pacman -S --needed - < ~/backup/pkglist-official.txt

# Cài gói AUR
yay -S --needed - < ~/backup/pkglist-aur.txt
```

### Backup toàn bộ home (rsync)

```bash
# Backup ra ổ cứng ngoài (ví dụ mount tại /mnt/backup)
rsync -avh --delete \
  --exclude '.cache' \
  --exclude '.local/share/Trash' \
  --exclude 'Downloads' \
  ~/ /mnt/backup/home/
```

## Công cụ backup

### Rsync (đơn giản)

```bash
# Backup toàn bộ home, loại trừ cache
rsync -avh --exclude '.cache' ~/ /mnt/backup/home/
```

### Borg Backup (mạnh hơn)

```bash
pacman -S borg
```

Borg hỗ trợ:
- Deduplication: Chỉ lưu dữ liệu thay đổi.
- Compression: Nén dữ liệu.
- Encryption: Mã hóa backup.

```bash
# Khởi tạo repo
borg init --encryption=repokey /mnt/backup/borg/

# Backup
borg create --stats --progress \
  /mnt/backup/borg::{hostname}-{now:%Y-%m-%d} \
  ~/Documents ~/Pictures ~/.config \
  --exclude '*.cache'

# Restore
borg extract /mnt/backup/borg::archive-name
```

### Rclone (cloud)

```bash
pacman -S rclone
```

Rclone hỗ trợ: Google Drive, Dropbox, OneDrive, S3, v.v.

```bash
# Cấu hình
rclone config

# Sync lên cloud
rclone sync ~/Documents remote:backup/Documents
```

## Lịch backup

| Task | Công cụ | Tần suất |
|---|---|---|
| Snapshot BTRFS | Timeshift | Hàng ngày |
| Danh sách gói | pacman -Qqen | Hàng tuần |
| Config | rsync | Hàng tuần |
| Home (quan trọng) | rsync / Borg | Hàng tuần |
| Home (đầy đủ) | rsync | Hàng tháng |
| Cloud (Documents) | rclone | Hàng tuần |

## Kiểm tra backup

Backup không kiểm tra = không có backup.

```bash
# Kiểm tra rsync
rsync -avh --dry-run ~/ /mnt/backup/home/

# Kiểm tra Borg
borg list /mnt/backup/borg/
borg check /mnt/backup/borg/

# Kiểm tra file trên ổ backup
ls -la /mnt/backup/home/
```

## Khôi phục từ backup

### Restore config

```bash
cp -r ~/backup/config/bspwm ~/.config/
```

### Restore home từ rsync

```bash
rsync -avh /mnt/backup/home/ ~/
```

### Restore gói

```bash
pacman -S --needed - < pkglist-official.txt
yay -S --needed - < pkglist-aur.txt
```

## Tổng kết

- Snapshot BTRFS bảo vệ khỏi lỗi cập nhật và cấu hình.
- Backup ra ổ ngoài bảo vệ khỏi hỏng ổ cứng.
- Backup config + danh sách gói cho phép cài lại nhanh.
- 3-2-1 rule: 3 copies, 2 media, 1 offsite.
- Kiểm tra backup định kỳ.
