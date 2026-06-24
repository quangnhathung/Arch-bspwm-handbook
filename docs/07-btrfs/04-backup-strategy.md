# Backup Strategy

## Mục tiêu

Xây dựng chiến lược backup thực tế cho desktop cá nhân — bảo vệ dữ liệu ngoài snapshot.

## Snapshot vs Backup

| Tiêu chí | Snapshot BTRFS | Backup thực sự |
|---|---|---|
| Vị trí | Cùng ổ cứng | Ổ ngoài / Cloud |
| Bảo vệ khỏi hỏng ổ? | Không | Có |
| Bảo vệ khỏi xóa nhầm? | Có (nếu snapshot trước) | Có |
| Bảo vệ khỏi mất máy? | Không | Có (nếu offsite) |
| Tốc độ | Tức thì | Chậm hơn |
| Dung lượng | Thấp (CoW) | Cao (copy nguyên bản) |

**Snapshot ≠ Backup. Cần cả hai.**

## 3-2-1 Rule

- **3** bản sao dữ liệu (1 chính + 2 backup).
- **2** loại media khác nhau (ổ trong, ổ ngoài).
- **1** bản ở nơi khác (offsite / cloud).

Ví dụ: Laptop (bản chính) + Ổ SSD ngoài (local) + Google Drive (cloud).

## Dữ liệu cần backup

| Mức độ | Dữ liệu | Dung lượng | Backup tần suất |
|---|---|---|---|
| **Quan trọng** | Documents, Pictures, Config, Password DB | Nhỏ (< 10 GB) | Hàng tuần |
| **Trung bình** | Browser profile, SSH keys, GPG keys | Vài GB | Hàng tháng |
| **Thấp** | Cache, log, packages | Lớn | Không cần |
| **Không** | Hệ thống (có thể cài lại) | — | Snapshot là đủ |

## Backup danh sách gói

Dễ nhất — cho phép cài lại chính xác các gói.

```bash
# Backup
pacman -Qqen > ~/backup/pkglist-official.txt
pacman -Qqem > ~/backup/pkglist-aur.txt

# Restore
pacman -S --needed - < ~/backup/pkglist-official.txt
yay -S --needed - < ~/backup/pkglist-aur.txt
```

## Backup cấu hình

### Cấu hình hệ thống

```bash
mkdir -p ~/backup/etc
sudo cp /etc/default/grub ~/backup/etc/
sudo cp /etc/fstab ~/backup/etc/
sudo cp /etc/pacman.conf ~/backup/etc/
sudo cp /etc/timeshift/timeshift.json ~/backup/etc/
```

### Cấu hình người dùng

```bash
mkdir -p ~/backup/config
cp -r ~/.config/bspwm ~/backup/config/
cp -r ~/.config/sxhkd ~/backup/config/
cp -r ~/.config/polybar ~/backup/config/
cp -r ~/.config/alacritty ~/backup/config/
cp -r ~/.config/kitty ~/backup/config/
cp -r ~/.config/nvim ~/backup/config/
cp ~/.bashrc ~/backup/
cp ~/.zshrc ~/backup/
```

## Rsync — Backup ra ổ ngoài

```bash
# Backup home, exclude cache
rsync -avh --delete \
  --exclude '.cache' \
  --exclude '.local/share/Trash' \
  --exclude 'Downloads' \
  --exclude '.steam' \
  ~/ /mnt/backup/home/
```

Giải thích:
- `-a`: Archive mode (giữ permissions, timestamps).
- `-v`: Verbose.
- `-h`: Human-readable.
- `--delete`: Xóa file ở đích nếu không còn ở nguồn.
- `--exclude`: Bỏ qua thư mục không cần backup.

## Borg Backup — Nâng cao

Borg hỗ trợ deduplication, nén, mã hóa.

```bash
pacman -S borg
```

### Khởi tạo repo

```bash
borg init --encryption=repokey /mnt/backup/borg/
```

Nhớ passphrase — nếu quên mất dữ liệu.

### Backup

```bash
borg create --stats --progress \
  /mnt/backup/borg::{hostname}-{now:%Y-%m-%d} \
  ~/Documents \
  ~/Pictures \
  ~/.config \
  ~/.ssh \
  --exclude '*.cache'
```

### Restore

```bash
# Liệt kê archive
borg list /mnt/backup/borg/

# Restore toàn bộ
borg extract /mnt/backup/borg::archive-name

# Restore thư mục cụ thể
borg extract /mnt/backup/borg::archive-name path/to/restore
```

### Kiểm tra

```bash
borg check /mnt/backup/borg/
```

## Rclone — Cloud backup

```bash
pacman -S rclone
```

### Cấu hình

```bash
rclone config
```

Hỗ trợ: Google Drive, Dropbox, OneDrive, S3, v.v.

### Sync lên cloud

```bash
# Documents lên Google Drive
rclone sync ~/Documents remote:backup/Documents

# Config lên cloud
rclone sync ~/backup/config remote:backup/config
```

### Lịch tự động với systemd timer

```bash
# rclone sync hàng tuần
rclone sync ~/Documents remote:backup/Documents --progress
```

## Backup schedule

| Công việc | Công cụ | Tần suất |
|---|---|---|
| Snapshot BTRFS | Timeshift | Hàng ngày |
| Danh sách gói | pacman -Qqen | Hàng tuần |
| Cấu hình | rsync | Hàng tuần |
| Documents (local) | Borg / rsync | Hàng tuần |
| Documents (cloud) | rclone | Hàng tuần |
| Full home (ngoại trừ cache) | rsync | Hàng tháng |
| Kiểm tra backup | Borg check / rsync --dry-run | Hàng tháng |

## Kiểm tra backup

**Backup không kiểm tra = không có backup.**

```bash
# Rsync — dry run
rsync -avh --dry-run ~/ /mnt/backup/home/

# Borg — check
borg list /mnt/backup/borg/
borg check /mnt/backup/borg/

# Rclone — kiểm tra file
rclone ls remote:backup/Documents

# Kiểm tra thủ công
ls -la /mnt/backup/home/
```

## Restore từ backup

### Restore cấu hình

```bash
cp -r /mnt/backup/config/bspwm ~/.config/
```

### Restore home từ rsync

```bash
rsync -avh /mnt/backup/home/ ~/
```

### Restore từ Borg

```bash
# Tìm archive gần nhất
borg list /mnt/backup/borg/

# Extract
borg extract /mnt/backup/borg::tên-archive
```

### Restore từ cloud (rclone)

```bash
rclone sync remote:backup/Documents ~/Documents
```

### Restore toàn bộ sau khi cài lại Arch

1. Cài Arch + BSPWM cơ bản.
2. Chạy restore config và home.

```bash
# Restore config
cp -r /mnt/backup/config/* ~/.config/

# Restore home
rsync -avh /mnt/backup/home/ ~/

# Restore packages
pacman -S --needed - < pkglist-official.txt
yay -S --needed - < pkglist-aur.txt
```

## Lưu ý

- **Snapshot BTRFS** = bảo vệ khỏi lỗi update, cấu hình sai (rollback nhanh).
- **Backup ra ổ ngoài** = bảo vệ khỏi hỏng ổ cứng, mất máy.
- **Cloud backup** = bảo vệ khỏi mất cắp, cháy nổ (offsite).
- Mã hóa backup nếu chứa dữ liệu nhạy cảm (Borg, rclone crypt).
- Kiểm tra backup định kỳ — backup hỏng còn tệ hơn không backup.

## Tổng kết

- 3-2-1 rule: 3 bản, 2 media, 1 offsite.
- Snapshot ≠ Backup — cần cả hai.
- Backup danh sách gói + config = cài lại nhanh.
- Rsync cho local, Borg cho dedup/encryption, rclone cho cloud.
- Kiểm tra backup định kỳ.
- Restore từ backup sau khi cài lại Arch.
