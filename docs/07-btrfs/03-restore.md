# Restore từ Snapshot

## Mục tiêu

Khôi phục hệ thống từ snapshot BTRFS khi gặp sự cố.

## Kiến thức nền

### Restore là gì?

Restore là quá trình đưa subvolume về trạng thái của một snapshot.
Điều này undo tất cả thay đổi kể từ khi snapshot được tạo.

### Cách restore hoạt động

BTRFS không cho phép sửa read-only snapshot.
Thay vào đó, chúng ta đổi tên — không copy dữ liệu (tức thì):

1. Đổi tên subvolume hiện tại thành `@_broken`.
2. Đổi tên snapshot thành `@`.
3. Rebuild initramfs và GRUB.

**Không mất thêm dung lượng** vì chỉ rename.

### Khi nào cần restore?

- Hệ thống không boot được sau update.
- Cấu hình sai làm hỏng màn hình đăng nhập.
- Xóa nhầm file hệ thống quan trọng.
- Driver đồ họa lỗi.

## Các bước restore thủ công

### Bước 1: Boot từ Arch Live USB

Boot từ USB cài Arch. Chọn "Arch Linux install medium".

### Bước 2: Mount partition BTRFS

```bash
# Xác định partition BTRFS
lsblk -f

# Mount (không dùng subvol= option)
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
ID 258 gen 125 top level 5 path .snapshots/@_20250625_143000
ID 259 gen 126 top level 5 path .snapshots/@_20250618_080000
```

### Bước 4: Chọn snapshot để restore

Ví dụ restore về `@_20250625_143000`.

### Bước 5: Rename subvolume hiện tại

```bash
# Backup subvolume cũ
mv /mnt/@ /mnt/@_broken

# Rename snapshot thành @
mv /mnt/.snapshots/@_20250625_143000 /mnt/@
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

# Mount EFI
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

## Restore bằng Timeshift từ Live USB

### Cài Timeshift trên live USB

```bash
pacman -Sy timeshift
```

### Restore

```bash
# Liệt kê snapshot
sudo timeshift --list

# Restore
sudo timeshift --restore --snapshot '2026-06-25_14-30-01'
```

Timeshift tự động rename và rebuild. Sau đó reboot.

### Restore từ GRUB (nếu Timeshift tích hợp)

1. Boot → Advanced Options for Arch Linux.
2. Chọn kernel với "timeshift" trong tên.
3. Menu Timeshift hiện ra → chọn snapshot → Enter.

## Restore @home riêng

Nếu chỉ cần restore dữ liệu cá nhân:

```bash
# Từ live USB
mount /dev/nvme0n1p2 /mnt

# Backup @home cũ
mv /mnt/@home /mnt/@home_broken

# Restore từ snapshot
mv /mnt/.snapshots/@home_20250625_143000 /mnt/@home

# Không cần rebuild GRUB — @home không ảnh hưởng boot
reboot
```

## Dọn dẹp sau restore

```bash
# Xóa subvolume hỏng (sau khi chắc chắn hệ thống ổn)
sudo btrfs subvolume delete /mnt/@_broken

# Trong hệ thống đang chạy
sudo btrfs subvolume delete /.snapshots/@_broken
# hoặc
mv /.snapshots/@_broken /tmp/
```

## WARNING: Data loss

**Restore snapshot sẽ xóa tất cả thay đổi kể từ khi snapshot được tạo.**

- File mới tạo sau snapshot → **biến mất**.
- Cấu hình thay đổi sau snapshot → **mất**.
- Gói cài sau snapshot → **mất** (phải cài lại).

Cách giảm thiểu:
- Snapshot @home riêng → không bị ảnh hưởng khi restore @.
- Backup dữ liệu quan trọng trước khi restore.
- Giữ `@_broken` vài ngày trước khi xóa.

## Troubleshooting

### "Device or resource busy"

```bash
# Unmount tất cả
umount -R /mnt
sleep 1
mount /dev/nvme0n1p2 /mnt
```

### Không boot được sau restore

```bash
# Kiểm tra trong chroot
mount -o subvol=@ /dev/nvme0n1p2 /mnt
arch-chroot /mnt

# Kiểm tra fstab
cat /etc/fstab

# Rebuild lại
mkinitcpio -P
grub-mkconfig -o /boot/grub/grub.cfg
```

### Snapshot không tìm thấy

```bash
btrfs subvolume list -s /mnt
```

Kiểm tra snapshot có ở đường dẫn khác (vd `/timeshift/`).

### UUID thay đổi?

Không — vì snapshot trên cùng partition, UUID không đổi.

## Lưu ý quan trọng

1. **Snapshot không phải backup**: Nếu ổ cứng hỏng, snapshot cũng mất.
2. **Restore @ không ảnh hưởng @home**: Nếu mount riêng.
3. **Luôn rebuild initramfs và GRUB** sau khi restore @.
4. **Giữ @_broken** trước khi xóa — đề phòng restore sai.
5. **Snapshot càng cũ, càng mất nhiều dữ liệu**.

## Tổng kết

- Restore BTRFS = rename subvolume (nhanh, không copy).
- Boot live USB → mount → rename → rebuild → reboot.
- Timeshift có thể restore từ CLI hoặc GRUB.
- Restore @home riêng nếu cần.
- Luôn giữ @_broken trước khi xóa.
- Cảnh báo: mất dữ liệu từ sau snapshot.
