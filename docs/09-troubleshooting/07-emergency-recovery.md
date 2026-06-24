# Khôi phục khẩn cấp từ USB

*Áp dụng cho Lenovo LOQ 15IAX9 — BTRFS + NVIDIA + UEFI — Kernel 7.x — 25/06/2026*

Hướng dẫn từ A→Z để cứu hệ thống từ USB live khi không thể boot.

---

## 1. Boot USB Arch Live

1. Cắm USB Arch.
2. Boot → F12 (LOQ 15IAX9) → chọn USB.
3. Chọn "Arch Linux install medium".

## 2. Mount hệ thống BTRFS

```bash
# Xác định partition
lsblk -f
# LOQ 15IAX9: nvme0n1p1 = ESP, nvme0n1p2 = BTRFS

# Mount BTRFS gốc
mount /dev/nvme0n1p2 /mnt

# Mount subvolume @ (root)
mount -o subvol=@ /dev/nvme0n1p2 /mnt

# Mount các subvolume khác (nếu có)
mount -o subvol=@home /dev/nvme0n1p2 /mnt/home || mkdir -p /mnt/home
mount -o subvol=@log /dev/nvme0n1p2 /mnt/var/log || mkdir -p /mnt/var/log
mount -o subvol=@pkg /dev/nvme0n1p2 /mnt/var/cache/pacman/pkg || mkdir -p /mnt/var/cache/pacman/pkg

# Mount ESP
mount /dev/nvme0n1p1 /mnt/efi
```

## 3. Chroot

```bash
arch-chroot /mnt
```

## 4. Chẩn đoán

```bash
# Kernel
ls /boot/vmlinuz-linux
ls /boot/initramfs-linux.img

# fstab
cat /etc/fstab

# Log
journalctl -p 3 -xb --no-pager | tail -40

# Disk space
df -h

# BTRFS health
btrfs filesystem usage /
btrfs device stats /
```

## 5. Sửa lỗi thường gặp

### GRUB hỏng

```bash
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB --recheck
grub-mkconfig -o /boot/grub/grub.cfg
```

### Kernel hỏng

```bash
pacman -S linux
mkinitcpio -P
grub-mkconfig -o /boot/grub/grub.cfg
```

### fstab sai

```bash
blkid /dev/nvme0n1p1 /dev/nvme0n1p2
vim /etc/fstab
# Sửa UUID cho khớp
```

### NVIDIA lỗi (blacklist để boot)

```bash
echo -e "blacklist nvidia\nblacklist nvidia_modeset\nblacklist nvidia_uvm\nblacklist nvidia_drm" > /etc/modprobe.d/blacklist-nvidia.conf
mkinitcpio -P
```

> Nếu cần cài lại driver: `pacman -S nvidia-open-dkms nvidia-utils`

### BTRFS lỗi

```bash
# Kiểm tra
btrfs device stats /

# Quét sửa lỗi
btrfs scrub start /

# Kiểm tra sau scrub
btrfs device stats /
```

## 6. Restore từ snapshot

### Dùng Timeshift (nếu có)

```bash
timeshift --list
timeshift --restore --snapshot 'TEN-SNAPSHOT'
```

### Thủ công (rename subvolume)

```bash
# Xác định snapshot tốt
ls /mnt/.snapshots/
# ...

# Đổi tên @ cũ (hỏng) thành @_broken
cd /mnt
mv @ @_broken

# Copy snapshot tốt thành @ mới
btrfs subvolume snapshot .snapshots/@_GOOD_SNAPSHOT @
# Hoặc: cp -a --reflink=always .snapshots/@_GOOD_SNAPSHOT @
```

## 7. Rebuild initramfs và GRUB

```bash
# Trong chroot
mkinitcpio -P
grub-mkconfig -o /boot/grub/grub.cfg
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB --recheck
```

## 8. Reboot

```bash
exit
umount -R /mnt
reboot
```

---

## Các tình huống đặc biệt

### Mất hoàn toàn GRUB (ESP bị format)

```bash
# Tạo lại ESP
mkfs.fat -F32 /dev/nvme0n1p1

# Mount
mount /dev/nvme0n1p1 /mnt/efi

# Trong chroot
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

### BTRFS subvolume @ bị xóa

```bash
# Tạo mới từ snapshot
cd /mnt
btrfs subvolume snapshot .snapshots/@_GOOD_SNAPSHOT @

# Nếu không có snapshot → cài lại từ đầu
```

### Kernel panic loop (lỗi module)

```bash
# Boot vào kernel cũ từ GRUB Advanced Options
# Hoặc dùng linux-lts
pacman -S linux-lts
grub-mkconfig -o /boot/grub/grub.cfg
```

---

## Kit khẩn cấp (nên có trên USB)

| Thành phần | Ghi chú |
|---|---|
| Arch ISO mới nhất | Luôn cập nhật, kernel 7.x + |
| `config_bk.sh` | Script backup nhanh bspwmrc, sxhkdrc, grub, fstab |
| `pkglist_official.txt` | `pacman -Qqn > pkglist_official.txt` |
| `pkglist_aur.txt` | `pacman -Qqm > pkglist_aur.txt` |
| Snapshot BTRFS gần nhất | Timeshift hoặc thủ công |
| USB dự phòng | Luôn mang theo |

## Phòng ngừa (Prevention)

1. **Timeshift snapshot tự động hàng ngày** — giữ tối thiểu 5 snapshot gần.
2. **Backup config ra USB / ổ ngoài định kỳ.**
3. **Luôn có USB Arch ở trong túi laptop LOQ 15IAX9.**
4. **Không panic — hầu hết lỗi đều sửa được từ USB.**
5. **Dùng git để quản lý config** (`~/.config` repo riêng).