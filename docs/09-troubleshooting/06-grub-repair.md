# GRUB không boot

*Áp dụng cho Lenovo LOQ 15IAX9 — BTRFS + UEFI — Kernel 7.x — 25/06/2026*

## Triệu chứng (Symptoms)

- `grub rescue>` prompt thay vì menu GRUB.
- "No such partition" hoặc "File not found".
- Màn hình đen ngay sau khi chọn GRUB entry.
- Boot thẳng vào Windows (nếu dual-boot) — không thấy Arch.
- Sau restore snapshot → GRUB không tìm thấy kernel.

## Nguyên nhân (Causes)

1. **GRUB bị hỏng do cập nhật kernel không rebuild** — `grub.cfg` cũ trỏ sai đường dẫn.
2. **EFI System Partition (ESP) bị format hoặc lỗi** — mất file `.efi`.
3. **`/etc/fstab` sai UUID** — không mount được root BTRFS.
4. **BTRFS subvolume `@` bị thay đổi / xóa** — GRUB không tìm thấy `/boot`.
5. **Sau restore snapshot** — cấu hình GRUB cũ trong snapshot không khớp với partition hiện tại.

## Chẩn đoán (Diagnosis)

```bash
# Boot từ USB Arch → mount hệ thống
mount /dev/nvme0n1p2 /mnt
mount -o subvol=@ /dev/nvme0n1p2 /mnt

# Mount các subvolume khác
mount -o subvol=@home /dev/nvme0n1p2 /mnt/home
mount -o subvol=@log /dev/nvme0n1p2 /mnt/var/log

# Mount ESP
mount /dev/nvme0n1p1 /mnt/efi

# Chroot
arch-chroot /mnt

# Kiểm tra GRUB config
cat /boot/grub/grub.cfg | head -30

# Kiểm tra kernel
ls /boot/vmlinuz-linux

# Kiểm tra EFI file
ls /efi/EFI/GRUB/grubx64.efi

# Kiểm tra boot entries
efibootmgr -v

# Kiểm tra fstab
cat /etc/fstab
blkid /dev/nvme0n1p1 /dev/nvme0n1p2
```

## Khắc phục (Fix)

### Fix 1: Reinstall GRUB (phổ biến nhất)

```bash
# Trong chroot
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB --recheck

# Sinh config mới
grub-mkconfig -o /boot/grub/grub.cfg

# Kiểm tra
efibootmgr -v
```

### Fix 2: Sửa fstab sai UUID

```bash
# Lấy UUID đúng
blkid /dev/nvme0n1p2  # root BTRFS
blkid /dev/nvme0n1p1  # ESP

# Sửa /etc/fstab
vim /etc/fstab

# Đảm bảo UUID trùng với blkid
# Ví dụ:
# UUID=xxxx-xxxx  /efi  vfat  umask=0077  0  1
# UUID=yyyy-yyyy  /     btrfs subvol=@,compress=zstd  0  0
```

### Fix 3: Boot thủ công từ grub rescue>

Gõ các lệnh sau tại `grub rescue>`:

```bash
# Liệt kê các partition
ls

# Tìm root (thử từng partition)
ls (hd0,gpt2)/
ls (hd0,gpt2)/@/boot/grub

# Set root và load normal module
set root=(hd0,gpt2)
set prefix=(hd0,gpt2)/@/boot/grub
insmod normal
normal
```

Sau khi boot được → vào hệ thống, chạy:
```bash
sudo grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB --recheck
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

### Fix 4: Sau restore snapshot

Sau khi restore snapshot BTRFS (manual hoặc Timeshift):

```bash
# Trong chroot
mkinitcpio -P
grub-mkconfig -o /boot/grub/grub.cfg
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB --recheck
```

### Fix 5: Thêm boot entry bằng efibootmgr

```bash
# Nếu GRUB cài rồi nhưng NVRAM entry bị mất
efibootmgr -c -d /dev/nvme0n1 -p 1 -L "GRUB" -l '\EFI\GRUB\grubx64.efi'
```

## Phòng ngừa (Prevention)

1. **Tạo snapshot BTRFS trước khi cập nhật GRUB.**
2. **Sao lưu `/etc/default/grub` ra USB hoặc repo riêng.**
3. **Kiểm tra `/etc/fstab` sau mỗi lần thay đổi partition.**
4. **Không format ESP (`nvme0n1p1`) trừ khi biết chính xác mình đang làm gì.**
5. **Sau restore snapshot, luôn rebuild GRUB:**
```bash
grub-mkconfig -o /boot/grub/grub.cfg
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB --recheck
```
