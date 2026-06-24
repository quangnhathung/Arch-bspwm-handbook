# BTRFS Snapshots

## Mục tiêu

Hiểu và tạo snapshot BTRFS thủ công — công cụ rollback nhanh nhất.

## Kiến thức nền

### Snapshot là gì?

Snapshot là ảnh chụp trạng thái của một subvolume tại một thời điểm.
Nó dùng CoW (Copy-on-Write) để chỉ lưu sự khác biệt so với bản gốc.

```
Ban đầu: [A][B][C][D]        (dữ liệu gốc)
Snapshot: [A][B]              (chỉ trỏ đến A, B, chưa copy)
Sau khi sửa C → C':
Gốc:     [A][B][C'][D]
Snapshot: [A][B][C]           (C được copy trước khi ghi đè)
```

### Đặc điểm

- **Tạo tức thì**: Dù dữ liệu lớn, snapshot tạo trong vài giây.
- **Tiết kiệm**: Ban đầu gần như 0 byte (chỉ metadata).
- **Chiếm dung lượng dần**: Khi dữ liệu gốc thay đổi, các block cũ được giữ lại.
- **Chỉ trên cùng filesystem**: Snapshot nằm trên cùng ổ BTRFS.

### Tại sao dùng snapshot?

- Rollback sau update lỗi.
- Rollback sau cấu hình sai.
- Rollback sau xóa nhầm file.
- Phục hồi nhanh hơn cài lại từ đầu.

## Cấu trúc subvolume

Hệ thống Arch-BSPWM sử dụng các subvolume:

```
@           → /
@home       → /home
@log        → /var/log
@pkg        → /var/cache/pacman/pkg
@swap       → /swap
```

Snapshot lưu tại `/` → dùng `/.snapshots/`.

## Snapshot naming convention

Đặt tên theo pattern: `@_YYYYMMDD_HHMMSS`

Ví dụ: `@_20250625_143000` — snapshot của `@` ngày 25/06/2026 lúc 14:30.

```
@_20250625_143000    → snapshot của @
@home_20250625_143000 → snapshot của @home
```

## Các lệnh snapshot cơ bản

### Tạo snapshot read-only

```bash
# Cú pháp
btrfs subvolume snapshot -r <nguồn> <đích>

# Snapshot @ (root filesystem)
sudo btrfs subvolume snapshot -r / /.snapshots/@_$(date +%Y%m%d_%H%M%S)

# Snapshot @home
sudo btrfs subvolume snapshot -r /home /.snapshots/@home_$(date +%Y%m%d_%H%M%S)
```

- `-r`: Read-only — không thể sửa, an toàn cho backup.
- `/.snapshots/`: Thư mục chứa snapshot.

### Tạo writable snapshot

Bỏ `-r` nếu muốn snapshot có thể chỉnh sửa (dùng để clone).

### Liệt kê snapshot

```bash
# Liệt kê tất cả subvolume
sudo btrfs subvolume list /

# Chỉ liệt kê snapshot
sudo btrfs subvolume list -s /

# Chi tiết
sudo btrfs subvolume list -a /
```

Output:

```
ID 261 gen 128 top level 5 path .snapshots/@_20250625_143000
ID 262 gen 129 top level 5 path .snapshots/@home_20250625_143000
```

### Xóa snapshot

```bash
sudo btrfs subvolume delete /.snapshots/@_20250601_120000
```

Snapshot xóa ngay lập tức, giải phóng dung lượng.

## Thư mục snapshot

```bash
# Tạo thư mục chứa snapshot
sudo mkdir -p /.snapshots
```

Snapshot nên lưu ở `/.snapshots` (gốc của BTRFS partition).
Không lưu snapshot bên trong `@` vì sẽ bị xóa khi restore.

## Script snapshot nhanh

`/usr/local/bin/snap`:

```bash
#!/bin/bash
# Quick snapshot tool
# Ngày: 25/06/2026
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

## Tự động snapshot trước update

Thêm vào shell alias trong `~/.bashrc`:

```bash
alias update='snap root && sudo pacman -Syu'
```

Hoặc script:

```bash
#!/bin/bash
snap root
yay -Syu
```

## Giới hạn của snapshot

- **Snapshot không phải backup**: Dữ liệu trên cùng ổ cứng — nếu ổ hỏng, mất tất cả.
- **Snapshot chiếm dung lượng dần**: Cần kiểm tra định kỳ.
- **Không thay thế backup ngoài**: Luôn có backup ra ổ ngoài hoặc cloud.

## Lưu ý

- Snapshot read-only (`-r`) an toàn hơn — không bị sửa đổi.
- Xóa snapshot cũ định kỳ để tránh đầy ổ.
- Kiểm tra dung lượng: `sudo btrfs filesystem usage /`.
- Snapshot @home riêng biệt với @ — có thể rollback @ mà không ảnh hưởng home.

## Tổng kết

- Snapshot BTRFS là CoW — tạo nhanh, tiết kiệm dung lượng.
- Đặt tên theo convention `@_YYYYMMDD_HHMMSS`.
- Lưu trong `/.snapshots`.
- Script `snap` giúp tạo snapshot nhanh.
- Snapshot không thay thế backup ra ổ ngoài.
