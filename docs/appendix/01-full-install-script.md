# Full Install Script

## Mục tiêu

Script tự động hóa toàn bộ quá trình cài đặt Arch Linux trên Lenovo LOQ 15IAX9.

## Cảnh báo

Script này sẽ **xóa SẠCH toàn bộ dữ liệu trên ổ `/dev/nvme0n1`**.
Chạy script này đồng nghĩa với việc bạn đồng ý mất toàn bộ dữ liệu cũ.

**Chỉ chạy khi đã boot vào Arch live environment và đã có kết nối Internet.**

## Script

Tạo file `install-arch.sh`:

```bash
#!/bin/bash
#
# install-arch.sh — Full Arch Linux installation for Lenovo LOQ 15IAX9
# Usage: bash install-arch.sh
#

set -e

# ===== Configuration =====
DISK="/dev/nvme0n1"
HOSTNAME="loq-arch"
USERNAME="archuser"
TIMEZONE="Asia/Ho_Chi_Minh"
PASSWORD="archlinux"
LOCALE="en_US.UTF-8"

# ===== Warning =====
echo "=========================================="
echo " WARNING: This will COMPLETELY WIPE $DISK"
echo "=========================================="
echo ""
echo "All data will be destroyed."
echo "Press Ctrl+C to cancel (5 seconds)..."
sleep 5

# ===== Disk Partitioning =====
echo ">>> Partitioning disk..."
sgdisk --zap-all "$DISK"
sgdisk --clear "$DISK"
sgdisk --new=1:0:+1G --typecode=1:ef00 "$DISK"
sgdisk --new=2:0:0 --typecode=2:8300 "$DISK"
partprobe "$DISK"
sleep 2

# ===== Format ESP =====
echo ">>> Formatting ESP..."
mkfs.fat -F32 "${DISK}p1"

# ===== Create BTRFS =====
echo ">>> Creating BTRFS filesystem..."
mkfs.btrfs -f "${DISK}p2"

# ===== Create Subvolumes =====
echo ">>> Creating BTRFS subvolumes..."
mount "${DISK}p2" /mnt
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@log
btrfs subvolume create /mnt/@pkg
btrfs subvolume create /mnt/@swap
umount /mnt

# ===== Mount Subvolumes =====
echo ">>> Mounting subvolumes..."
mount -o compress=zstd,noatime,space_cache=v2,autodefrag,subvol=@ "${DISK}p2" /mnt
mkdir -p /mnt/{home,var/log,var/cache/pacman/pkg,swap,efi}
mount -o compress=zstd,noatime,space_cache=v2,autodefrag,subvol=@home "${DISK}p2" /mnt/home
mount -o compress=zstd,noatime,space_cache=v2,autodefrag,subvol=@log "${DISK}p2" /mnt/var/log
mount -o compress=zstd,noatime,space_cache=v2,autodefrag,subvol=@pkg "${DISK}p2" /mnt/var/cache/pacman/pkg
mount -o nodatacow,noatime,space_cache=v2,subvol=@swap "${DISK}p2" /mnt/swap
mount "${DISK}p1" /mnt/efi

# ===== Install Base System =====
echo ">>> Installing base system..."
pacman -Sy --noconfirm archlinux-keyring
pacstrap -K /mnt base linux linux-firmware intel-ucode sof-firmware vim sudo networkmanager git base-devel btrfs-progs

# ===== Generate Fstab =====
echo ">>> Generating fstab..."
genfstab -U /mnt >> /mnt/etc/fstab

# ===== Chroot Commands =====
echo ">>> Configuring system (chroot)..."

arch-chroot /mnt /bin/bash <<'CHROOT_COMMANDS'
    # Timezone
    ln -sf /usr/share/zoneinfo/Asia/Ho_Chi_Minh /etc/localtime
    hwclock --systohc

    # Locale
    sed -i 's/^#en_US.UTF-8/en_US.UTF-8/' /etc/locale.gen
    locale-gen
    echo "LANG=en_US.UTF-8" > /etc/locale.conf

    # Hostname
    echo "loq-arch" > /etc/hostname
    cat > /etc/hosts <<EOF
127.0.0.1   localhost
::1         localhost
127.0.1.1   loq-arch.localdomain loq-arch
EOF

    # User
    echo "root:archlinux" | chpasswd
    useradd -m -G wheel -s /bin/bash archuser
    echo "archuser:archlinux" | chpasswd
    echo "%wheel ALL=(ALL:ALL) ALL" >> /etc/sudoers

    # Network
    systemctl enable NetworkManager

    # GRUB
    pacman -S --noconfirm grub efibootmgr
    sed -i 's/GRUB_CMDLINE_LINUX_DEFAULT=".*"/GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet nvidia-drm.modeset=1 nowatchdog"/' /etc/default/grub
    grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB
    grub-mkconfig -o /boot/grub/grub.cfg

    # Initramfs
    sed -i 's/^MODULES=(.*)/MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm)/' /etc/mkinitcpio.conf
    mkinitcpio -P
CHROOT_COMMANDS

# ===== Unmount =====
echo ">>> Unmounting..."
umount -R /mnt

echo "=========================================="
echo " Installation complete!"
echo " Remove USB and reboot."
echo "=========================================="
echo ""
echo "User:     archuser"
echo "Password: archlinux"
echo "Root:     archlinux"
echo ""
echo "After reboot, run:"
echo "  1. nmcli device wifi connect <SSID> password <pass>"
echo "  2. sudo pacman -S xorg bspwm sxhkd ..."
echo "  3. Follow docs/04-desktop/ for desktop setup"
```

## Cách dùng

```bash
# Boot vào Arch live environment
# Có kết nối Internet
# Chạy script
bash install-arch.sh
```

## Sau khi script chạy xong

1. Reboot: `reboot`
2. Login với `archuser` / `archlinux`.
3. Chạy các bước tiếp theo theo tài liệu.

## Lưu ý

- Script đặt password mặc định. **Đổi password ngay sau khi cài xong**.
- Script không cài Xorg/bspwm. Cần làm thủ công.
- Script cài NVIDIA module trong initramfs.
- Nếu Wi-Fi không hoạt động, cần cài driver Realtek riêng sau khi reboot.
