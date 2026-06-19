# GRUB không boot

## Symptoms

- Sau khi bật máy, thấy `grub rescue>` prompt.
- "No such partition" hoặc "File not found".
- GRUB menu không xuất hiện, boot thẳng vào Windows (nếu còn).
- Màn hình đen sau khi chọn GRUB.

## Cause

1. **GRUB bị hỏng do update kernel không đúng**.
2. **EFI System Partition bị format/lỗi**.
3. **fstab sai** — không mount được root.
4. **GRUB không tìm thấy kernel** do sai subvolume.
5. **BTRFS subvolume bị xóa hoặc thay đổi**.
6. **Sau khi restore snapshot** — GRUB config cũ không khớp.

## Diagnosis

```bash
# Boot từ USB Arch
# Mount hệ thống
mount /dev/nvme0n1p2 /mnt
mount -o subvol=@ /dev/nvme0n1p2 /mnt
mount /dev/nvme0n1p1 /mnt/efi

# Chroot
arch-chroot /mnt

# Kiểm tra GRUB config
cat /boot/grub/grub.cfg | head -20

# Kiểm tra kernel
ls /boot/vmlinuz-linux

# Kiểm tra EFI
ls /efi/EFI/GRUB/grubx64.efi

# Kiểm tra boot entries
efibootmgr -v
```

## Fix

### Fix 1: Reinstall GRUB

```bash
# Trong chroot
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB --recheck

# Sinh config mới
grub-mkconfig -o /boot/grub/grub.cfg
```

### Fix 2: Sửa fstab sai

```bash
# Kiểm tra UUID
blkid /dev/nvme0n1p2
blkid /dev/nvme0n1p1

# Sửa fstab
vim /etc/fstab

# Đảm bảo UUID đúng
```

### Fix 3: GRUB rescue mode

Khi ở `grub rescue>`:

```bash
# Tìm root
ls
# Sẽ hiện (hd0,gpt1) (hd0,gpt2) ...

# Set root và boot thủ công
set root=(hd0,gpt2)
set prefix=(hd0,gpt2)/@/boot/grub
insmod normal
normal
```

Sau khi boot được, chạy:

```bash
sudo grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB --recheck
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

### Fix 4: Sau restore snapshot

Sau khi restore snapshot BTRFS:

```bash
# Trong chroot
mkinitcpio -P
grub-mkconfig -o /boot/grub/grub.cfg
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB --recheck
```

### Fix 5: Thêm boot entry thủ công

```bash
efibootmgr -c -d /dev/nvme0n1 -p 1 -L "GRUB" -l "\EFI\GRUB\grubx64.efi"
```

## Prevention

1. **Tạo snapshot trước khi update GRUB**.
2. **Sao lưu `/etc/default/grub`**.
3. **Kiểm tra fstab** sau mỗi lần thay đổi partition.
4. **Không format ESP** (nvme0n1p1) trừ khi biết rõ.
5. **Sau restore snapshot, luôn rebuild GRUB**.
