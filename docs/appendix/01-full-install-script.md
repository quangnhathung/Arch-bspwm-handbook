# Script Cài Đặt Hoàn Chỉnh — Lenovo LOQ 15IAX9

## Mục tiêu

Tự động hoá toàn bộ quá trình cài đặt Arch Linux trên Lenovo LOQ 15IAX9 (Intel Core Ultra 5 125H, RTX 4050).

## Cảnh báo

Script này sẽ **xoá SẠCH toàn bộ dữ liệu trên ổ `/dev/nvme0n1`**.
Chạy script đồng nghĩa với việc bạn chấp nhận mất toàn bộ dữ liệu cũ.

**Chỉ chạy khi đã boot vào Arch live environment và đã có kết nối Internet.**

## Script

Tạo file `install-arch.sh`:

```bash
#!/bin/bash
#
# install-arch.sh — Cài đặt Arch Linux tự động cho Lenovo LOQ 15IAX9
# Ngày: 25/06/2026 — Kernel 7.x
# Cách dùng: bash install-arch.sh
#
# WARNING: Xoá SẠCH toàn bộ đĩa /dev/nvme0n1
#

set -e

# ===== Cấu hình =====
DISK="/dev/nvme0n1"
HOSTNAME="loq-arch"
USERNAME="archuser"
TIMEZONE="Asia/Ho_Chi_Minh"
PASSWORD="archlinux"
LOCALE="en_US.UTF-8"
ESP_SIZE="1G"

# ===== Cảnh báo =====
echo "=========================================="
echo "  CẢNH BÁO: Sẽ XOÁ SẠCH $DISK"
echo "=========================================="
echo ""
echo "Toàn bộ dữ liệu trên $DISK sẽ bị phá huỷ."
echo "Nhấn Ctrl+C để huỷ (5 giây)..."
sleep 5

# ===== Phân vùng đĩa =====
echo ">>> Đang phân vùng đĩa..."
sgdisk --zap-all "$DISK"
sgdisk --clear "$DISK"
sgdisk --new=1:0:+${ESP_SIZE} --typecode=1:ef00 "$DISK"
sgdisk --new=2:0:0     --typecode=2:8300 "$DISK"
partprobe "$DISK"
sleep 2

# ===== Định dạng ESP =====
echo ">>> Đang định dạng ESP (FAT32)..."
mkfs.fat -F32 "${DISK}p1"

# ===== Tạo BTRFS =====
echo ">>> Đang tạo filesystem BTRFS..."
mkfs.btrfs -f "${DISK}p2"

# ===== Tạo subvolume =====
echo ">>> Đang tạo BTRFS subvolumes..."
mount "${DISK}p2" /mnt
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@log
btrfs subvolume create /mnt/@pkg
btrfs subvolume create /mnt/@swap
umount /mnt

# ===== Mount subvolumes =====
echo ">>> Đang mount subvolumes..."
mount -o compress=zstd:3,noatime,space_cache=v2,autodefrag,subvol=@ "${DISK}p2" /mnt
mkdir -p /mnt/{home,var/log,var/cache/pacman/pkg,swap,efi}
mount -o compress=zstd:3,noatime,space_cache=v2,autodefrag,subvol=@home "${DISK}p2" /mnt/home
mount -o compress=zstd:3,noatime,space_cache=v2,autodefrag,subvol=@log "${DISK}p2" /mnt/var/log
mount -o compress=zstd:3,noatime,space_cache=v2,autodefrag,subvol=@pkg "${DISK}p2" /mnt/var/cache/pacman/pkg
mount -o nodatacow,noatime,space_cache=v2,subvol=@swap "${DISK}p2" /mnt/swap
mount "${DISK}p1" /mnt/efi

# ===== Cài base system =====
echo ">>> Đang cài base system..."
pacman -Sy --noconfirm archlinux-keyring
pacstrap -K /mnt base linux linux-firmware intel-ucode sof-firmware \
  vim sudo networkmanager git base-devel btrfs-progs grub efibootmgr \
  pacman-contrib

# ===== Sinh fstab =====
echo ">>> Đang sinh fstab..."
genfstab -U /mnt >> /mnt/etc/fstab

# ===== Chroot cấu hình =====
echo ">>> Đang cấu hình hệ thống (chroot)..."

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

    # GRUB — kernel params cho NVIDIA
    sed -i 's/GRUB_CMDLINE_LINUX_DEFAULT=".*"/GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet nvidia-drm.modeset=1 nowatchdog"/' /etc/default/grub
    grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB
    grub-mkconfig -o /boot/grub/grub.cfg

    # Initramfs — thêm module NVIDIA open
    sed -i 's/^MODULES=(.*)/MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm)/' /etc/mkinitcpio.conf
    mkinitcpio -P
CHROOT_COMMANDS

# ===== Unmount =====
echo ">>> Đang unmount..."
umount -R /mnt

echo "=========================================="
echo "  Cài đặt hoàn tất!"
echo "  Rút USB và khởi động lại."
echo "=========================================="
echo ""
echo "User:     archuser"
echo "Password: archlinux"
echo "Root:     archlinux"
echo ""
echo "Sau khi reboot, chạy các bước sau:"
echo "  1. Kết nối Wi-Fi: nmcli device wifi connect <SSID> password <pass>"
echo "  2. Cập nhật: sudo pacman -Syu"
echo "  3. Cài yay: xem docs/appendix/02-post-install-checklist.md"
echo "  4. Cài desktop: xem docs/appendix/02-post-install-checklist.md"
```

## Cách dùng

```bash
# Boot từ USB Arch live environment
# Đảm bảo có Internet
# Chạy script
bash install-arch.sh
```

## Sau khi script chạy xong

1. `reboot`
2. Đăng nhập với `archuser` / `archlinux`.
3. Làm theo **Post-Install Checklist** (`02-post-install-checklist.md`).

## Lưu ý quan trọng

- Script đặt mật khẩu mặc định. **Đổi ngay sau khi cài xong**.
- Script **không** cài Xorg/bspwm. Cần làm thủ công sau.
- Script dùng **nvidia-open** (module open source cho RTX 4050).
- Nếu Wi-Fi không hoạt động, cài driver Realtek `rtl8852be-dkms` qua AUR sau khi reboot.
- Script cài `pacman-contrib` để có lệnh `paccache`.
- Ngày phát hành: 25/06/2026 — nhân Linux 7.x.
