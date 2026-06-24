# Timeshift — Snapshot GUI

## Mục tiêu

Cài đặt và cấu hình Timeshift để tự động quản lý snapshot BTRFS.

## Kiến thức nền

### Timeshift là gì?

Timeshift là công cụ snapshot cho Linux, giống như System Restore trên Windows.
Nó tự động tạo snapshot theo lịch và cung cấp giao diện để restore.

### BTRFS mode vs RSYNC mode

| Mode | Cơ chế | Dung lượng | Tốc độ |
|---|---|---|---|
| BTRFS | Snapshot (CoW) | Thấp | Rất nhanh |
| RSYNC | Copy file | Cao | Chậm |

Chúng ta dùng **BTRFS mode**.

## Cài đặt

```bash
pacman -S timeshift
```

Giao diện đồ họa (tùy chọn):

```bash
pacman -S timeshift-gtk
```

## Cấu hình lần đầu

### CLI

```bash
sudo timeshift --first-run
```

### GUI

```bash
sudo timeshift-gtk
```

### Các bước setup

1. **Snapshot Type**: Chọn **BTRFS**.
2. **Snapshot Device**: Chọn partition BTRFS (`/dev/nvme0n1p2`).
3. **Snapshot Location**: `/.snapshots` (Timeshift tự tạo nếu chưa có).
4. **Schedule**:
   - Daily: 3 snapshots
   - Weekly: 2 snapshots
   - Monthly: 1 snapshot
5. **User Home**: Chọn Exclude (vì @home là subvolume riêng).
6. Nhấn **Create** để tạo snapshot đầu tiên.

## Exlude subvolume trong BTRFS mode

Trong BTRFS mode, Timeshift mặc định snapshot toàn bộ subvolume gốc.
Cần exclude các subvolume không cần snapshot:

| Subvolume | Mount point | Lý do exclude |
|---|---|---|
| `@home` | `/home` | Dữ liệu cá nhân, thay đổi liên tục → snapshot tốn dung lượng |
| `@pkg` | `/var/cache/pacman/pkg` | Cache gói, không cần rollback |
| `@swap` | `/swap` | Có nodatacow, không snapshot được |

Timeshift tự động exclude các subvolume riêng nếu được mount riêng.
Kiểm tra trong `timeshift-gtk` → Settings → Exclude.

## Schedule (lịch snapshot)

Thiết lập hợp lý cho desktop cá nhân:

| Loại | Số lượng giữ | Giải thích |
|---|---|---|
| Daily | 3 | Snapshot 3 ngày gần nhất |
| Weekly | 2 | Snapshot 2 tuần gần nhất |
| Monthly | 1 | Snapshot 1 tháng gần nhất |

Tổng cộng: 6 snapshot, dung lượng vài GB đến vài chục GB tùy mức độ thay đổi.

## Tạo snapshot thủ công

```bash
sudo timeshift --create --comments "truoc-khi-update-kernel"
```

## Liệt kê snapshot

```bash
sudo timeshift --list
```

Output:

```
Num     Name                                Tags
0    > 2026-06-25_14-30-01                  O
1    > 2026-06-24_08-00-01                  D
2    > 2026-06-18_08-00-01                  W
```

Tags: `O` (Older), `D` (Daily), `W` (Weekly), `M` (Monthly).

## Xóa snapshot

```bash
# Theo tên
sudo timeshift --delete --snapshot '2026-06-18_08-00-01'

# Xóa snapshot cũ nhất
sudo timeshift --delete-all
```

## File cấu hình

`/etc/timeshift/timeshift.json`:

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

Có thể edit thủ công nhưng nên dùng GUI/CLI của Timeshift.

## GUI vs CLI

| Tác vụ | GUI | CLI |
|---|---|---|
| Setup lần đầu | `sudo timeshift-gtk` | `sudo timeshift --first-run` |
| Tạo snapshot | Nút Create | `sudo timeshift --create --comments "..."` |
| Liệt kê | Xem danh sách | `sudo timeshift --list` |
| Restore | Chọn + Restore | `sudo timeshift --restore --snapshot "..."` |
| Xóa | Chọn + Delete | `sudo timeshift --delete --snapshot "..."` |

GUI dễ dùng hơn cho người mới. CLI nhanh hơn khi quen.

## Troubleshooting

### "BTRFS not detected"

```bash
# Kiểm tra filesystem
lsblk -f /dev/nvme0n1p2

# Phải là btrfs
```

### Timeshift không thấy snapshot

```bash
# Kiểm tra thư mục
ls -la /.snapshots

# Nếu snapshot ở /timeshift/
ls -la /timeshift/
```

### Dung lượng đầy do snapshot

```bash
# Xóa snapshot cũ
sudo timeshift --delete --snapshot '2026-06-01_20-00-01'

# Xóa tất cả snapshot cũ (giữ 2 gần nhất)
sudo timeshift --delete-all
```

### "Snapshot creation failed"

```bash
# Kiểm tra dung lượng còn trống
df -h /

# Kiểm tra đủ quyền ghi
ls -la /.snapshots
```

## Best practices

1. **Exclude @home**: Dữ liệu cá nhân không cần snapshot thường xuyên.
2. **Exclude @pkg**: Cache pacman không cần rollback.
3. **Exclude @swap**: Swap có nodatacow, không snapshot được.
4. **Giới hạn số lượng snapshot**: Không giữ quá nhiều → tốn dung lượng.
5. **Snapshot trước update lớn**: Tạo thủ công trước kernel update.
6. **Kiểm tra định kỳ**: `df -h /` và `sudo timeshift --list`.
7. **Snapshot không thay thế backup**: Vẫn cần backup ra ổ ngoài.

## Tổng kết

- Timeshift tự động tạo snapshot BTRFS theo lịch.
- BTRFS mode dùng CoW snapshot (nhanh, tiết kiệm).
- Exclude @home, @pkg, @swap để tránh lãng phí dung lượng.
- Có thể tạo snapshot thủ công trước update.
- GUI (`timeshift-gtk`) cho người mới, CLI cho người quen.
- Snapshot là lớp bảo vệ đầu tiên — vẫn cần backup ra ổ ngoài.
