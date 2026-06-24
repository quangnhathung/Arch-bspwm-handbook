# Mount hệ thống

## Mục tiêu

Format EFI System Partition (ESP), mount các subvolume BTRFS vào đúng vị trí trước khi cài base system.

## Điều kiện tiên quyết

- Partition BTRFS (`/dev/nvme0n1p2`) đã format và có subvolume.
- EFI System Partition (`/dev/nvme0n1p1`) đã tạo và chưa format.

## Kiến thức nền về mount options

### compress=zstd

Nén dữ liệu trong suốt. Zstd là chuẩn nén hiện đại:
- Tỉ lệ nén cao hơn zlib (gzip) nhưng tốc độ nhanh hơn.
- Level mặc định (3) cân bằng giữa tốc độ và tỉ lệ nén.
- Tiết kiệm 20-40% dung lượng tùy loại file.

### noatime

Khi đọc file, Linux mặc định ghi access time (thời gian truy cập cuối).
- `noatime` tắt hoàn toàn việc ghi access time.
- Giảm đáng kể số lần ghi vào NVMe.
- Hầu hết ứng dụng không cần access time.

### space_cache=v2

BTRFS cache thông tin về không gian trống. v2 là phiên bản mới:
- Hiệu suất cao hơn khi filesystem đầy.
- Giảm CPU usage khi cấp phát block mới.

### nodatacow

Tắt Copy-on-Write cho subvolume. Cần cho swapfile vì:
- Swap không tương thích với CoW (kernel từ chối).
- CoW trên swapfile gây phân mảnh và giảm hiệu suất.
- Subvolume `@swap` dùng `nodatacow`.

---

## Cách A: Cài thủ công

### Bước 1: Format EFI System Partition

```bash
mkfs.fat -F32 /dev/nvme0n1p1
```

Giải thích:
- `mkfs.fat`: Tạo FAT32 filesystem.
- `-F32`: Chỉ định FAT32 (FAT16/FAT12 cũ không dùng được cho UEFI).

Nếu gặp lỗi `mkfs.fat: command not found`:
```bash
pacman -S dosfstools
```

### Bước 2: Mount root subvolume (`@`)

```bash
mount -o compress=zstd,noatime,space_cache=v2,autodefrag,subvol=@ \
  /dev/nvme0n1p2 /mnt
```

### Bước 3: Tạo thư mục mount points

```bash
mkdir -p /mnt/{home,var/log,var/cache/pacman/pkg,swap,efi}
```

### Bước 4: Mount các subvolume còn lại

```bash
mount -o compress=zstd,noatime,space_cache=v2,autodefrag,subvol=@home \
  /dev/nvme0n1p2 /mnt/home

mount -o compress=zstd,noatime,space_cache=v2,autodefrag,subvol=@log \
  /dev/nvme0n1p2 /mnt/var/log

mount -o compress=zstd,noatime,space_cache=v2,autodefrag,subvol=@pkg \
  /dev/nvme0n1p2 /mnt/var/cache/pacman/pkg

mount -o nodatacow,noatime,space_cache=v2,subvol=@swap \
  /dev/nvme0n1p2 /mnt/swap
```

Lưu ý: `@swap` không có `compress=zstd` (vì `nodatacow` và `compress` không cần
cho swapfile) và không có `autodefrag`.

### Bước 5: Mount EFI System Partition

```bash
mount /dev/nvme0n1p1 /mnt/efi
```

### Bước 6: Kiểm tra kết quả

```bash
lsblk
```

Output mong đợi:
```
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
nvme0n1     259:0    0 476.9G  0 disk
├─nvme0n1p1 259:1    0     1G  0 part /mnt/efi
└─nvme0n1p2 259:2    0 475.9G  0 part /mnt
                                        /mnt/home
                                        /mnt/var/log
                                        /mnt/var/cache/pacman/pkg
                                        /mnt/swap
```

Kiểm tra subvolume tree:
```bash
btrfs subvolume list /mnt
```

Kiểm tra mount options:
```bash
mount | grep nvme0n1
```

Output mẫu:
```
/dev/nvme0n1p2 on /mnt type btrfs (rw,noatime,compress=zstd:3,space_cache=v2,autodefrag,subvol=@)
/dev/nvme0n1p2 on /mnt/home type btrfs (rw,noatime,compress=zstd:3,space_cache=v2,autodefrag,subvol=@home)
/dev/nvme0n1p2 on /mnt/var/log type btrfs (rw,noatime,compress=zstd:3,space_cache=v2,autodefrag,subvol=@log)
/dev/nvme0n1p2 on /mnt/var/cache/pacman/pkg type btrfs (rw,noatime,compress=zstd:3,space_cache=v2,autodefrag,subvol=@pkg)
/dev/nvme0n1p2 on /mnt/swap type btrfs (rw,noatime,space_cache=v2,nodatacow,subvol=@swap)
/dev/nvme0n1p1 on /mnt/efi type vfat (rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro)
```

---

## Cách B: Dùng Archinstall

Khi dùng `archinstall` với `Manual partitioning`:

### Bước 1: Chạy archinstall

```bash
archinstall
```

### Bước 2: Cấu hình mount

Trong menu `Disk configuration` → `Manual partitioning`:

1. Chọn partition `/dev/nvme0n1p1`:
   - Filesystem: `fat32`.
   - Mountpoint: `/boot` (Archinstall mặc định) hoặc `/efi`.
   - Nếu dùng `/boot`, GRUB sẽ cài trực tiếp vào `/boot`.
   - Nếu dùng `/efi`, cần tạo thư mục `/boot/efi` hoặc dùng `/efi`.

2. Chọn partition `/dev/nvme0n1p2`:
   - Filesystem: `btrfs`.
   - Mountpoint: `/`.
   - Subvolumes: thêm từng subvolume nếu được hỏi:
     - `@` → `/`
     - `@home` → `/home`
     - `@log` → `/var/log`
     - `@pkg` → `/var/cache/pacman/pkg`
     - `@swap` → `/swap`

### Bước 3: Nếu archinstall không hỗ trợ subvolume tùy chỉnh

Archinstall có thể chỉ tạo một subvolume mặc định. Trong trường hợp đó:
- Chọn `btrfs` và mount `/` trực tiếp (không subvolume tùy chỉnh).
- Sau khi cài xong, boot vào hệ thống và tự tạo subvolume + sửa fstab.

### Bước 4: Hoặc dùng `archinstall --config` với cấu hình có sẵn

Nếu đã có file cấu hình JSON:
```bash
archinstall --config archinstall-config.json
```

File cấu hình mẫu chứa subvolume config (xem appendix).

---

## Cấu trúc mount hoàn chỉnh

```
/mnt
├── (BTRFS subvolume @)          → root filesystem
├── home/                        → @home (BTRFS subvolume)
├── var/
│   ├── log/                     → @log (BTRFS subvolume)
│   └── cache/
│       └── pacman/
│           └── pkg/             → @pkg (BTRFS subvolume)
├── swap/                        → @swap (BTRFS, nodatacow)
└── efi/                         → ESP (FAT32)
```

## Xác minh mount options

```bash
mount | grep "/mnt"
```

Kiểm tra từng dòng:
- `@`, `@home`, `@log`, `@pkg`: phải có `compress=zstd`, `noatime`, `space_cache=v2`.
- `@swap`: phải có `nodatacow`, không có `compress`.
- `/efi` (ESP): FAT32, không có compress.

## Xử lý lỗi

### "mount: /mnt: /dev/nvme0n1p2 already mounted"

```bash
umount -R /mnt
# Sau đó mount lại từ đầu
```

### "wrong fs type, bad option, bad superblock"

- Chưa format BTRFS: `mkfs.btrfs -f /dev/nvme0n1p2`.
- Option sai: kiểm tra lại cú pháp `subvol=@` (không có space, không có dấu nháy).

### "FAT32 not supported"

```bash
pacman -S dosfstools
# Sau đó chạy lại mkfs.fat
```

### "invalid option: compress"

- Kernel quá cũ không hỗ trợ zstd. Arch ISO luôn có kernel mới — reboot live
  environment nếu gặp.

## Tổng kết

| Partition | Mountpoint | Filesystem | Options đặc biệt |
|---|---|---|---|
| `/dev/nvme0n1p1` | `/mnt/efi` | FAT32 | — |
| `/dev/nvme0n1p2` (`@`) | `/mnt` | BTRFS | `compress=zstd,noatime` |
| `/dev/nvme0n1p2` (`@home`) | `/mnt/home` | BTRFS | `compress=zstd,noatime` |
| `/dev/nvme0n1p2` (`@log`) | `/mnt/var/log` | BTRFS | `compress=zstd,noatime` |
| `/dev/nvme0n1p2` (`@pkg`) | `/mnt/var/cache/pacman/pkg` | BTRFS | `compress=zstd,noatime` |
| `/dev/nvme0n1p2` (`@swap`) | `/mnt/swap` | BTRFS | `nodatacow,noatime` |

Hệ thống đã sẵn sàng để cài base system bằng pacstrap.
