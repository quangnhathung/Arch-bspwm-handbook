# Khôi phục khẩn cấp

## Mục tiêu

Hướng dẫn khôi phục hệ thống từ USB live khi không thể boot.

## Các bước khôi phục cơ bản

### Bước 1: Boot USB Arch Live

- Cắm USB Arch.
- Boot từ USB (F12).
- Chọn Arch Linux install medium.

### Bước 2: Mount hệ thống

```bash
# Tìm partition BTRFS
lsblk

# Mount BTRFS partition
mount /dev/nvme0n1p2 /mnt

# Mount subvolume @ (root)
mount -o subvol=@ /dev/nvme0n1p2 /mnt

# Mount các subvolume khác
mount -o subvol=@home /dev/nvme0n1p2 /mnt/home || mkdir -p /mnt/home
mount -o subvol=@log /dev/nvme0n1p2 /mnt/var/log || mkdir -p /mnt/var/log
mount -o subvol=@pkg /dev/nvme0n1p2 /mnt/var/cache/pacman/pkg || mkdir -p /mnt/var/cache/pacman/pkg
mount -o subvol=@swap /dev/nvme0n1p2 /mnt/swap || mkdir -p /mnt/swap

# Mount ESP
mount /dev/nvme0n1p1 /mnt/efi
```

### Bước 3: Chroot

```bash
arch-chroot /mnt
```

### Bước 4: Xác định vấn đề

```bash
# Kiểm tra kernel
ls /boot/vmlinuz-linux
ls /boot/initramfs-linux.img

# Kiểm tra fstab
cat /etc/fstab

# Kiểm tra log
journalctl -p 3 -xb --no-pager | tail -30

# Kiểm tra disk
df -h
btrfs filesystem usage /
```

### Bước 5: Sửa lỗi

Tùy theo lỗi:

#### GRUB hỏng

```bash
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB --recheck
grub-mkconfig -o /boot/grub/grub.cfg
```

#### Kernel hỏng

```bash
pacman -S linux
mkinitcpio -P
```

#### fstab sai

```bash
vim /etc/fstab
# Sửa UUID hoặc mount options
```

#### NVIDIA lỗi

```bash
# Blacklist NVIDIA
echo -e "blacklist nvidia\nblacklist nvidia_modeset\nblacklist nvidia_uvm\nblacklist nvidia_drm" > /etc/modprobe.d/blacklist-nvidia.conf
mkinitcpio -P
```

#### BTRFS filesystem lỗi

```bash
# Kiểm tra
btrfs device stats /

# Sửa lỗi (cẩn thận)
btrfs scrub start /
```

### Bước 6: Restore từ snapshot

Nếu có Timeshift snapshot (xem docs/07-btrfs/restore.md):

```bash
# Liệt kê snapshot
timeshift --list

# Restore
timeshift --restore --snapshot 'TÊN-SNAPSHOT'
```

Hoặc thủ công:

```bash
cd /mnt
mv @ @_broken
mv .snapshots/@_GOOD_SNAPSHOT @
```

### Bước 7: Rebuild initramfs và GRUB

```bash
mkinitcpio -P
grub-mkconfig -o /boot/grub/grub.cfg
```

### Bước 8: Reboot

```bash
exit
umount -R /mnt
reboot
```

## Các tình huống khẩn cấp

### Mất hoàn toàn GRUB (ESP bị format)

```bash
# Tạo lại ESP
mkfs.fat -F32 /dev/nvme0n1p1

# Mount
mount /dev/nvme0n1p1 /mnt/efi

# Cài GRUB
arch-chroot /mnt
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

### BTRFS subvolume @ bị xóa

```bash
# Tạo lại từ snapshot
cd /mnt
btrfs subvolume create @

# Hoặc restore từ snapshot
btrfs subvolume snapshot .snapshots/@_GOOD_SNAPSHOT @
```

### Kernel panic liên tục

```bash
# Boot vào kernel cũ từ GRUB advanced options
# Hoặc chọn fallback initramfs
# Hoặc dùng kernel linux-lts
pacman -S linux-lts
grub-mkconfig -o /boot/grub/grub.cfg
```

## Kit khẩn cấp nên có trên USB

1. **Arch ISO** luôn cập nhật.
2. **Backup config** (bspwmrc, sxhkdrc, grub, fstab).
3. **Danh sách gói** (pkglist-official.txt, pkglist-aur.txt).
4. **Script restore nhanh** (tự viết).

## Prevention

1. **Timeshift snapshot tự động hàng ngày**.
2. **Backup config ra ổ ngoài**.
3. **Luôn có USB Arch bên mình**.
4. **Đừng panic** — hầu hết lỗi đều sửa được từ USB.
5. **Ghi chép lại những gì đã làm** trước khi hỏng.
