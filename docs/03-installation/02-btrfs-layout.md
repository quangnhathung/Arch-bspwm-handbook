# BTRFS Layout và Subvolume

## Mục tiêu

Định dạng partition BTRFS và tạo cấu trúc subvolume chuẩn cho hệ thống.

## Giới thiệu BTRFS

BTRFS (B-tree File System, thường đọc là "Butter FS") là filesystem hiện đại của
Linux với các tính năng:

- **Copy-on-Write (CoW)**: Dữ liệu không ghi đè trực tiếp, tạo bản sao khi ghi.
- **Subvolume**: Các thư mục logic có thể mount riêng, snapshot riêng.
- **Snapshot**: Ảnh chụp trạng thái tại một thời điểm, có thể rollback.
- **Compression**: Nén dữ liệu trong suốt (zstd).
- **Checksum**: Tự động kiểm tra lỗi dữ liệu.

## Tại sao dùng BTRFS thay vì ext4

| Tính năng | BTRFS | ext4 |
|---|---|---|
| Snapshot | Có (tức thì) | Không |
| Subvolume | Có | Không |
| Compression | Có (zstd) | Không |
| Checksum | Có | Có (metadata) |
| Rollback | Có | Không |
| Maturity | Rất ổn định | Rất ổn định |

Với desktop cá nhân, BTRFS cho phép snapshot trước khi cập nhật lớn → rollback
nếu có lỗi.

## Cấu trúc subvolume

Chúng ta tạo các subvolume riêng cho từng phần của hệ thống:

```
@                  → /            (root)
@home              → /home        (dữ liệu người dùng)
@log               → /var/log     (log hệ thống)
@pkg               → /var/cache/pacman/pkg (cache gói)
@swap              → /swap        (swapfile)
```

### Giải thích từng subvolume

#### `@` — Root filesystem

Chứa toàn bộ hệ thống trừ các thư mục được tách riêng.
Khi snapshot `@`, bạn snapshot toàn bộ hệ thống (trừ /home, /var/log, cache).

#### `@home` — Home directory

Chứa dữ liệu người dùng (Desktop, Documents, config, v.v.).
Tách riêng để:
- Snapshot root không ảnh hưởng đến dữ liệu cá nhân.
- Có thể cài lại hệ thống mà giữ /home.

#### `@log` — System logs

Chứa log hệ thống. Log thay đổi liên tục, không cần snapshot.
Tách riêng để snapshot root gọn hơn.

#### `@pkg` — Pacman cache

Chứa file `.pkg.tar.zst` đã tải. Cache có thể rất lớn (nhiều GB).
Tách riêng để dễ dọn dẹp và không ảnh hưởng snapshot.

#### `@swap` — Swapfile

Chứa file swap. Swapfile không nên nằm trong subvolume có snapshot
vì CoW và swap không tương thích.

## Mount options cho laptop

Các option tối ưu cho ổ NVMe + laptop:

| Option | Giá trị | Ý nghĩa |
|---|---|---|
| compress | zstd | Nén dữ liệu, tiết kiệm dung lượng, tăng tuổi thọ NVMe |
| noatime | — | Không ghi access time, giảm ghi ổ |
| space_cache | v2 | Cache không gian trống, hiệu suất cao hơn |
| autodefrag | — | Tự động chống phân mảnh |

## Các bước thực hiện

### Bước 1: Format partition BTRFS

```bash
mkfs.btrfs -f /dev/nvme0n1p2
```

Giải thích:
- `mkfs.btrfs`: Tạo filesystem BTRFS.
- `-f`: Force (ghi đè nếu đã có dữ liệu).
- `/dev/nvme0n1p2`: Partition đã tạo ở bước trước.

### Bước 2: Mount tạm để tạo subvolume

```bash
mount /dev/nvme0n1p2 /mnt
```

### Bước 3: Tạo các subvolume

```bash
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@log
btrfs subvolume create /mnt/@pkg
btrfs subvolume create /mnt/@swap
```

### Bước 4: Unmount

```bash
umount /mnt
```

### Bước 5: Kiểm tra

```bash
btrfs subvolume list /mnt
```

(Không cần mount lại, có thể check sau khi mount ở bước tiếp theo.)

## Cấu trúc cuối cùng

```
BTRFS partition (/dev/nvme0n1p2)
├── @               → sẽ mount tại /
├── @home           → sẽ mount tại /home
├── @log            → sẽ mount tại /var/log
├── @pkg            → sẽ mount tại /var/cache/pacman/pkg
└── @swap           → sẽ mount tại /swap
```

## Lưu ý về swap

Không tạo file swap ở bước này. File swap sẽ được tạo sau khi đã chroot
vào hệ thống mới, vì lúc đó mới có các công cụ cần thiết (`dd`, `mkswap`, `swapon`).
Subvolume `@swap` sẽ được mount với option `nodatacow` để tránh xung đột CoW.

## Tổng kết

- Partition BTRFS đã được format.
- Năm subvolume đã được tạo với cấu trúc rõ ràng.
- Sẵn sàng để mount đúng vị trí và cài base system.
