# Restore từ Snapshot

## Mục tiêu

Khôi phục hệ thống từ snapshot BTRFS khi gặp sự cố.

## Kiến thức nền

### Restore là gì?

Restore là quá trình đưa subvolume về trạng thái của một snapshot.
Điều này sẽ undo tất cả thay đổi kể từ khi snapshot được tạo.

### Cách restore hoạt động

BTRFS không cho phép sửa read-only snapshot.
Thay vào đó, chúng ta:

1. Tạo snapshot từ snapshot cũ thành subvolume mới (hoặc ghi đè).
2. Hoặc đổi tên subvolume hiện tại thành backup, và rename snapshot thành subvolume mới.

### Có 2 cách restore

| Cách | Mô tả | Khi nào dùng |
|---|---|---|
| **Rollback** | Đổi tên subvolume + snapshot | Khi cần restore nhanh từ live system |
| **Boot từ snapshot** | Boot kernel từ snapshot | Khi hệ thống không boot được |

Chúng ta focus vào rollback từ live environment (khi boot từ USB).

## Các bước thực hiện

### Bước 1: Boot vào Arch live USB

Boot từ USB Arch, mount filesystem.

### Bước 2: Mount BTRFS partition

```bash
mount /dev/nvme0n1p2 /mnt
```

### Bước 3: Liệt kê snapshot

```bash
btrfs subvolume list /mnt
```

Output:

```
ID 256 gen 123 top level 5 path @
ID 257 gen 124 top level 5 path @home
ID 258 gen 125 top level 5 path @log
ID 259 gen 126 top level 5 path @pkg
ID 260 gen 127 top level 5 path @swap
ID 261 gen 128 top level 5 path .snapshots/@_20250601_120000
ID 262 gen 129 top level 5 path .snapshots/@_20250615_120000
```

### Bước 4: Chọn snapshot để restore

Giả sử cần restore về snapshot `@_20250615_120000`.

### Bước 5: Đổi tên subvolume hiện tại

```bash
# Backup subvolume hiện tại
mv /mnt/@ /mnt/@_broken

# Đổi tên snapshot thành @
mv /mnt/.snapshots/@_20250615_120000 /mnt/@
```

### Bước 6: Mount và chroot

```bash
# Mount subvolume mới
mount -o subvol=@ /dev/nvme0n1p2 /mnt

# Mount các subvolume khác
mount -o subvol=@home /dev/nvme0n1p2 /mnt/home
mount -o subvol=@log /dev/nvme0n1p2 /mnt/var/log
mount -o subvol=@pkg /dev/nvme0n1p2 /mnt/var/cache/pacman/pkg
mount -o subvol=@swap /dev/nvme0n1p2 /mnt/swap
mount /dev/nvme0n1p1 /mnt/efi

# Chroot
arch-chroot /mnt
```

### Bước 7: Rebuild initramfs và GRUB

```bash
# Rebuild initramfs
mkinitcpio -P

# Rebuild GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

### Bước 8: Reboot

```bash
exit
umount -R /mnt
reboot
```

## Restore bằng Timeshift

Nếu Timeshift hoạt động, có thể restore từ boot menu:

1. Boot vào GRUB → Advanced Options.
2. Chọn kernel với `timeshift` trong tên.
3. Timeshift sẽ hiện menu chọn snapshot.
4. Chọn snapshot → Enter.

Hoặc từ live system với Timeshift CLI:

```bash
# Boot live USB
# Cài Timeshift nếu chưa có
pacman -S timeshift

# Restore snapshot
sudo timeshift --restore --snapshot '2025-06-15_12-00-00'
```

## Restore @home (dữ liệu cá nhân)

Nếu chỉ cần restore home:

```bash
# Từ live USB
mount /dev/nvme0n1p2 /mnt

# Backup @home cũ
mv /mnt/@home /mnt/@home_broken

# Restore từ snapshot
mv /mnt/.snapshots/@home_20250601_120000 /mnt/@home
```

## Xóa snapshot cũ

Sau khi restore thành công:

```bash
# Xóa subvolume hỏng
btrfs subvolume delete /mnt/@_broken

# Xóa snapshot cũ không cần
btrfs subvolume delete /mnt/.snapshots/@_20250601_120000
```

## Troubleshooting

### Snapshot không tìm thấy

```bash
btrfs subvolume list -s /mnt
```

Nếu không thấy, lưu ý snapshot có đuôi `-r` (read-only) hoặc nằm ở đường dẫn khác.

### Không mount được sau khi restore

- Kiểm tra fstab (UUID có thay đổi không? → không, vì trên cùng partition).
- Kiểm tra subvol= option trong fstab.

### "Device or resource busy"

```bash
# Unmount tất cả
umount -R /mnt
sleep 1
mount /dev/nvme0n1p2 /mnt
```

## Lưu ý quan trọng

- Restore snapshot sẽ xóa tất cả thay đổi kể từ khi snapshot được tạo.
- File mới tạo sau snapshot sẽ biến mất.
- Dữ liệu trong @home không bị ảnh hưởng nếu bạn chỉ restore @.
- Luôn backup @home riêng nếu cần.
- Sau restore, cần rebuild initramfs và GRUB.

## Tổng kết

- Restore bằng rename subvolume (nhanh và an toàn).
- Có thể restore từ Timeshift (GUI hoặc CLI).
- Luôn backup subvolume cũ trước khi restore (đề phòng).
- Rebuild initramfs và GRUB sau khi restore @.
- Restore @home riêng nếu cần.
