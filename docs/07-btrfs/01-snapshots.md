# BTRFS Snapshots

## Mục tiêu

Hiểu và tạo snapshot BTRFS thủ công.

## Kiến thức nền

### Snapshot là gì?

Snapshot là ảnh chụp trạng thái của một subvolume tại một thời điểm.
Nó không phải bản sao dữ liệu — snapshot dùng CoW (Copy-on-Write) để
chỉ lưu sự khác biệt so với bản gốc.

```
Ban đầu: [A][B][C][D]        (dữ liệu gốc)
Snapshot: [A][B]              (chỉ trỏ đến A, B, chưa copy)
Sau khi sửa C → C':
Gốc:     [A][B][C'][D]
Snapshot: [A][B][C]           (C được copy trước khi ghi đè)
```

### Snapshot chiếm bao nhiêu dung lượng?

- **Lúc tạo**: Gần như 0 byte (chỉ metadata).
- **Sau khi thay đổi dữ liệu**: Chỉ lưu các block bị thay đổi.
- **Càng nhiều thay đổi**: Snapshot càng lớn.

### Tại sao dùng snapshot?

- Rollback sau update lỗi.
- Rollback sau cấu hình sai.
- Rollback sau khi xóa nhầm file.

## Các lệnh snapshot cơ bản

### Tạo snapshot

```bash
# Cú pháp
btrfs subvolume snapshot -r <nguồn> <đích>

# Snapshot subvolume @ (root)
sudo btrfs subvolume snapshot -r / /.snapshots/@_$(date +%Y%m%d_%H%M%S)

# Snapshot subvolume @home
sudo btrfs subvolume snapshot -r /home /.snapshots/@home_$(date +%Y%m%d_%H%M%S)
```

Giải thích:
- `-r`: Read-only snapshot (không thể sửa, an toàn).
- `/.snapshots`: Thư mục chứa tất cả snapshot.
- `@_20250619_120000`: Tên snapshot = subvolume + timestamp.

### Liệt kê snapshot

```bash
# Liệt kê subvolume
sudo btrfs subvolume list /

# Lọc snapshot
sudo btrfs subvolume list -s /
```

### Xóa snapshot

```bash
sudo btrfs subvolume delete /.snapshots/@_20250601_120000
```

## Thư mục chứa snapshot

Tạo thư mục snapshot:

```bash
sudo mkdir -p /.snapshots
```

Snapshot có thể lưu ở bất kỳ đâu trên cùng filesystem BTRFS.
Thường lưu trong `/.snapshots` hoặc `/.snapshots`.

## Script snapshot nhanh

```bash
vim /usr/local/bin/snap
```

Nội dung:

```bash
#!/bin/bash
# Quick snapshot tool
SNAPSHOT_DIR="/.snapshots"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

case "$1" in
    root)
        sudo btrfs subvolume snapshot -r / "$SNAPSHOT_DIR/@_$TIMESTAMP"
        echo "Created: @_$TIMESTAMP"
        ;;
    home)
        sudo btrfs subvolume snapshot -r /home "$SNAPSHOT_DIR/@home_$TIMESTAMP"
        echo "Created: @home_$TIMESTAMP"
        ;;
    list)
        sudo btrfs subvolume list -s /
        ;;
    *)
        echo "Usage: snap {root|home|list}"
        ;;
esac
```

```bash
sudo chmod +x /usr/local/bin/snap
```

Dùng:

```bash
snap root         # Snapshot @
snap home         # Snapshot @home
snap list         # Liệt kê snapshot
```

## Tạo snapshot trước update

```bash
#!/bin/bash
# Trước khi chạy pacman -Syu
snap root
snap home
sudo pacman -Syu
```

## Lưu ý

- Snapshot không thay thế backup (dữ liệu trên cùng ổ cứng).
- Nếu ổ cứng hỏng, snapshot cũng mất.
- Snapshot chiếm dung lượng dần dần.
- Cần kiểm tra dung lượng thường xuyên.
- Read-only snapshot an toàn hơn (không bị sửa).

## Tổng kết

- Snapshot là ảnh chụp subvolume, chiếm ít dung lượng ban đầu.
- Tạo snapshot trước mọi thay đổi lớn.
- Snapshot nằm trên cùng ổ BTRFS.
- Không thay thế backup ra ổ ngoài.
- Script snap giúp tạo snapshot nhanh.
