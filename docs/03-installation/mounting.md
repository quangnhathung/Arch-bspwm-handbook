# Mount hệ thống

## Mục tiêu

Mount các subvolume BTRFS và EFI System Partition vào đúng vị trí trước khi cài base.

## Điều kiện tiên quyết

- Partition BTRFS đã format và có subvolume.
- EFI System Partition đã format FAT32.

## Kiến thức nền

### Mount options

Chúng ta mount các subvolume với các option:

- **BTRFS subvolume**: Mount với `subvol=` để chỉ định subvolume.
- **Compress zstd**: Nén dữ liệu.
- **Noatime**: Tắt access time tracking.
- **Space_cache=v2**: Cache không gian trống.
- **Autodefrag**: Tự động chống phân mảnh.

Riêng subvolume `@swap` mount với `nodatacow` vì swap không tương thích CoW.

## Các bước thực hiện

### Bước 1: Format EFI System Partition

```bash
mkfs.fat -F32 /dev/nvme0n1p1
```

Giải thích:
- `mkfs.fat`: Tạo FAT32 filesystem.
- `-F32`: Chỉ định FAT32.
- `/dev/nvme0n1p1`: EFI System Partition (1GB).

### Bước 2: Mount root subvolume

```bash
mount -o compress=zstd,noatime,space_cache=v2,autodefrag,subvol=@ /dev/nvme0n1p2 /mnt
```

### Bước 3: Tạo thư mục mount cho các subvolume khác

```bash
mkdir -p /mnt/{home,var/log,var/cache/pacman/pkg,swap,efi}
```

### Bước 4: Mount các subvolume còn lại

```bash
mount -o compress=zstd,noatime,space_cache=v2,autodefrag,subvol=@home /dev/nvme0n1p2 /mnt/home
mount -o compress=zstd,noatime,space_cache=v2,autodefrag,subvol=@log /dev/nvme0n1p2 /mnt/var/log
mount -o compress=zstd,noatime,space_cache=v2,autodefrag,subvol=@pkg /dev/nvme0n1p2 /mnt/var/cache/pacman/pkg
mount -o nodatacow,noatime,space_cache=v2,subvol=@swap /dev/nvme0n1p2 /mnt/swap
```

### Bước 5: Mount EFI System Partition

```bash
mount /dev/nvme0n1p1 /mnt/efi
```

### Bước 6: Kiểm tra kết quả

```bash
lsblk
```

Kết quả mong đợi:

```
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
nvme0n1     259:0    0 476.9G  0 disk
├─nvme0n1p1 259:1    0     1G  0 part /mnt/efi
└─nvme0n1p2 259:2    0 475.9G  0 part /mnt
```

Kiểm tra thêm:

```bash
btrfs subvolume list /mnt
```

Output sẽ hiện 5 subvolume với ID tương ứng.

## Cấu trúc mount hoàn chỉnh

```
/mnt
├── (BTRFS subvolume @)        → root filesystem
├── home/                      → (BTRFS subvolume @home)
├── var/log/                   → (BTRFS subvolume @log)
├── var/cache/pacman/pkg/      → (BTRFS subvolume @pkg)
├── swap/                      → (BTRFS subvolume @swap, nodatacow)
└── efi/                       → (FAT32, EFI System Partition)
```

## Xác minh mount options

```bash
mount | grep nvme0n1
```

Output sẽ hiển thị các option đã mount. Kiểm tra xem `compress=zstd` và `noatime`
có xuất hiện không.

## Nếu có lỗi

### "mount: /mnt: /dev/nvme0n1p2 already mounted"

```bash
umount -R /mnt
# Sau đó mount lại từ đầu
```

### "wrong fs type, bad option, bad superblock"

- Chưa format BTRFS → chạy `mkfs.btrfs -f /dev/nvme0n1p2`.
- Partition bị hỏng → kiểm tra lại partition table.

### "FAT32 not supported"

```bash
pacman -S dosfstools
# Sau đó chạy lại mkfs.fat
```

## Tổng kết

- EFI partition đã format FAT32 và mount tại `/mnt/efi`.
- Các subvolume BTRFS đã mount đúng vị trí với option tối ưu.
- Sẵn sàng để cài base system bằng pacstrap.
