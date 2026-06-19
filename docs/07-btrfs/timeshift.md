# Timeshift — Snapshot GUI

## Mục tiêu

Cài đặt và cấu hình Timeshift để tự động quản lý snapshot BTRFS.

## Kiến thức nền

### Timeshift là gì?

Timeshift là công cụ snapshot cho Linux, giống như System Restore trên Windows.
Nó tự động tạo snapshot theo lịch và cung cấp giao diện để restore.

### Timeshift hoạt động thế nào?

Timeshift sử dụng BTRFS snapshot (nếu filesystem là BTRFS) hoặc rsync (nếu ext4).
Nó tạo snapshot theo lịch:

- **Hourly**: Mỗi giờ (giữ snapshot gần đây).
- **Daily**: Hàng ngày.
- **Weekly**: Hàng tuần.
- **Monthly**: Hàng tháng.

Và cho phép chọn số lượng snapshot cần giữ cho mỗi loại.

### BTRFS mode vs RSYNC mode

| Mode | Cách hoạt động | Dung lượng | Tốc độ |
|---|---|---|---|
| BTRFS | Snapshot (CoW) | Thấp | Rất nhanh |
| RSYNC | Copy file | Cao | Chậm |

Chúng ta dùng **BTRFS mode** vì đã có BTRFS.

## Các bước thực hiện

### Bước 1: Cài Timeshift

```bash
pacman -S timeshift
```

### Bước 2: Cấu hình Timeshift (lần đầu)

```bash
sudo timeshift --first-run
```

Hoặc:

```bash
sudo timeshift-gtk
```

### Bước 3: Cấu hình trong GUI

1. **Select Snapshot Type**: Chọn **BTRFS**.
2. **Select BTRFS Devices**: Chọn `/dev/nvme0n1p2`.
3. **Snapshot Location**: Chọn `/.snapshots` (Timeshift tự tạo).
4. **Schedule**:
   - Daily: 3 snapshots
   - Weekly: 2 snapshots
   - Monthly: 1 snapshot
5. **User Home**: Exclude (bỏ qua) — vì @home là subvolume riêng.
6. Nhấn **Create** để tạo snapshot đầu tiên.

### Bước 4: Kiểm tra snapshot

```bash
timeshift --list
```

Output:

```
Num     Name                                Tags
0    > 2025-06-19_20-00-01                  O
```

### Bước 5: Tạo snapshot thủ công

```bash
sudo timeshift --create --comments "truoc-khi-update"
```

### Bước 6: Kiểm tra dung lượng snapshot

```bash
sudo timeshift --list
```

## Cấu hình qua CLI

### File cấu hình

File: `/etc/timeshift/timeshift.json`

```json
{
  "backup_device_uuid": "uuid-cua-btrfs-partition",
  "parent_device_uuid": "",
  "do_first_run": false,
  "btrfs_mode": true,
  "include_btrfs_home": false,
  "exclude": [
    "/home/**",
    "/var/cache/pacman/pkg/**",
    "/swap/**"
  ],
  "stop_on_errors": false,
  "schedule_monthly": "1",
  "schedule_weekly": "2",
  "schedule_daily": "3",
  "count_monthly": "1",
  "count_weekly": "2",
  "count_daily": "3",
  "count_boot": "5"
}
```

## Best practices

1. **Exclude @home**: Dữ liệu cá nhân không cần snapshot thường xuyên.
2. **Exclude cache**: `@pkg` (pacman cache) không cần trong snapshot.
3. **Exclude swap**: `@swap` có nodatacow, không snapshot được.
4. **Snapshot hàng ngày**: Đủ cho desktop cá nhân.
5. **Kiểm tra dung lượng**: Timeshift snapshot có thể rất lớn nếu không giới hạn.

## Troubleshooting

### Timeshift báo "BTRFS not detected"

```bash
# Kiểm tra filesystem
lsblk -f /dev/nvme0n1p2

# Phải là btrfs
```

### Timeshift không thấy snapshot

```bash
# Kiểm tra thư mục
ls -la /timeshift/

# Nếu snapshot ở /.snapshots → symlink
ls -la /.snapshots
```

### Dung lượng đầy do snapshot

```bash
# Xóa snapshot cũ
sudo timeshift --delete --snapshot '2025-06-01_20-00-01'
```

## Tổng kết

- Timeshift tự động tạo snapshot BTRFS theo lịch.
- Cấu hình GUI dễ dùng.
- Exclude @home, @pkg, @swap để tiết kiệm dung lượng.
- Có thể tạo snapshot thủ công trước update.
- Snapshot là lớp bảo vệ đầu tiên — vẫn cần backup ra ổ ngoài.
