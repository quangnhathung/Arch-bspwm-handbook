# GRUB Bootloader

## Mục tiêu

Cài đặt GRUB làm bootloader cho UEFI và cấu hình kernel parameters cho Lenovo LOQ 15IAX9.

## Điều kiện tiên quyết

- Đã chroot vào hệ thống mới.
- Đã cấu hình locale, network, user.
- ESP đã mount đúng (`/efi`).

## Kiến thức nền

### Bootloader là gì?

Bootloader là chương trình nhỏ chạy đầu tiên khi máy khởi động. Nó load kernel
Linux từ ổ cứng vào RAM và chuyển quyền điều khiển cho kernel.

### GRUB (Grand Unified Bootloader)

GRUB 2 là bootloader phổ biến nhất trên Linux. Hỗ trợ:
- UEFI và Legacy BIOS.
- BTRFS, ext4, XFS, ZFS.
- LVM, encryption (LUKS).
- Kernel parameters.
- Dual boot (os-prober).

### EFI System Partition (ESP)

ESP là partition FAT32 chứa bootloader. UEFI firmware đọc ESP, tìm file `.efi`
trong thư mục `EFI/` để chạy.

```
ESP (/efi)
└── EFI/
    └── GRUB/
        ├── grubx64.efi        (GRUB EFI binary)
        └── grub.cfg           (GRUB embedded config)
```

### grub-install

1. Copy `grubx64.efi` vào `ESP/EFI/GRUB/`.
2. Đăng ký boot entry với firmware UEFI (dùng `efibootmgr`).

### grub-mkconfig

1. Quét kernel trong `/boot`.
2. Load microcode (`intel-ucode.img`).
3. Quét OS khác (nếu có os-prober).
4. Sinh `/boot/grub/grub.cfg`.

### mkinitcpio

Công cụ tạo initramfs (initial RAM filesystem). Kernel modules cần thiết để
boot hệ thống được load qua mkinitcpio. Với NVIDIA, cần thêm module `nvidia`,
`nvidia_modeset`, `nvidia_uvm`, `nvidia_drm` vào initramfs.

### nvidia-dkms và kernel update

Khi dùng `nvidia-open-dkms` (khuyến nghị cho RTX 4050):
- Driver NVIDIA được biên dịch lại **tự động** mỗi khi kernel được cập nhật.
- DKMS (Dynamic Kernel Module Support) đảm bảo driver luôn tương thích.
- Không cần cài lại driver thủ công sau `pacman -Syu`.
- Gói `nvidia-open` (phiên bản đóng gói sẵn) không cần DKMS nhưng chỉ tương
  thích với kernel hiện tại. `nvidia-open-dkms` linh hoạt hơn.

### grub-btrfs

Gói `grub-btrfs` tự động thêm các bản snapshot BTRFS vào GRUB menu. Cho phép
boot trực tiếp vào snapshot để rollback nếu hệ thống lỗi.

---

## Kernel Parameters cho Lenovo LOQ

Các tham số truyền cho kernel khi boot:

| Parameter | Bắt buộc | Ý nghĩa |
|---|---|---|
| `loglevel=3` | Nên | Chỉ hiện log từ mức error trở lên |
| `quiet` | Nên | Ẩn kernel messages khi boot |
| `nvidia-drm.modeset=1` | **Bắt buộc** | DRM modesetting cho NVIDIA GPU |
| `nowatchdog` | Khuyến nghị | Tắt NMI watchdog, tiết kiệm pin |

### Tại sao cần `nvidia-drm.modeset=1`?

DRM (Direct Rendering Manager) modesetting là cơ chế để kernel quản lý display
output. Cần thiết cho:
- Chống screen tearing.
- Hoạt động đúng của NVIDIA driver (`nvidia-open`).
- Hỗ trợ Wayland trên NVIDIA.
- PRIME render offload (Intel + NVIDIA hybrid graphics).

Nếu thiếu tham số này, NVIDIA driver có thể không khởi tạo đúng dẫn đến màn
hình đen khi load driver.

---

## Cách A: Cài thủ công

### Bước 1: Cài gói GRUB

```bash
pacman -S grub efibootmgr
```

- `grub`: Bootloader GRUB 2.
- `efibootmgr`: Công cụ quản lý EFI boot entries.

### Bước 2: Cài GRUB vào ESP

```bash
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB
```

Giải thích:
- `--target=x86_64-efi`: Cài cho UEFI 64-bit.
- `--efi-directory=/efi`: Thư mục ESP đã mount.
- `--bootloader-id=GRUB`: Tên hiển thị trong UEFI boot menu (F12).

Output mong đợi:
```
Installing for x86_64-efi platform.
Installation finished. No error reported.
```

### Bước 3: Cấu hình kernel parameters

Sửa file `/etc/default/grub`:

```bash
vim /etc/default/grub
```

Tìm dòng:
```
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet"
```

Sửa thành:
```
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet nvidia-drm.modeset=1 nowatchdog"
```

**Giải thích tham số:**
- `loglevel=3`: Chỉ hiện log từ mức 3 (error) trở lên.
- `quiet`: Ẩn kernel messages, boot nhanh và sạch hơn.
- `nvidia-drm.modeset=1`: Kích hoạt DRM modesetting cho NVIDIA GPU.
- `nowatchdog`: Tắt NMI watchdog (tiết kiệm năng lượng, giảm CPU usage).

### Bước 4: Cấu hình mkinitcpio cho NVIDIA

Với `nvidia-open`, cần thêm modules và hooks vào initramfs để NVIDIA driver
được load sớm khi boot.

Sửa `/etc/mkinitcpio.conf`:

```bash
vim /etc/mkinitcpio.conf
```

**Thêm NVIDIA modules:**

Tìm dòng `MODULES=()` và sửa thành:
```
MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm)
```

**Kiểm tra hooks:**

Dòng `HOOKS=` cần có `keyboard` và `kms`:
```
HOOKS=(base udev keyboard modconf kms keymap consolefont block filesystems fsck)
```

Hook `kms` (Kernel Mode Setting) tự động thêm module đồ họa vào initramfs.

Tạo lại initramfs:

```bash
mkinitcpio -P
```

Output:
```
==> Building image from preset: /etc/mkinitcpio.d/linux.preset: 'default'
==> Initcpio image: /boot/initramfs-linux.img
==> WARNING: Possibly missing firmware for module: nvidia
```

> Cảnh báo "missing firmware for module: nvidia" là bình thường. NVIDIA GPU
> không cần firmware.

### Bước 5: Sinh GRUB config

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

Output mong đợi:
```
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-linux
Found initrd image: /boot/intel-ucode.img /boot/initramfs-linux.img
Warning: os-prober will not be used to detect other bootable partitions ...
done
```

Kiểm tra kernel parameters trong grub.cfg:

```bash
grep "nvidia-drm.modeset" /boot/grub/grub.cfg
```

### Bước 6: Kiểm tra boot entry

```bash
efibootmgr -v
```

Output mẫu:
```
BootCurrent: 0001
Timeout: 0 seconds
BootOrder: 0001, 0000
Boot0000* GRUB    HD(1,GPT,xxx,0x800,0x80000)/File(\EFI\GRUB\grubx64.efi)
```

Kiểm tra file GRUB trong ESP:

```bash
ls -la /efi/EFI/GRUB/grubx64.efi
```

### Bước 7: (Tùy chọn) Cài grub-btrfs

`grub-btrfs` tự động thêm các snapshot BTRFS vào GRUB menu:

```bash
pacman -S grub-btrfs
systemctl enable grub-btrfsd
```

Sau mỗi lần tạo snapshot, chạy `grub-mkconfig -o /boot/grub/grub.cfg` để cập
nhật GRUB menu.

### Bước 8: Kiểm tra tổng thể trước reboot

```bash
# Kiểm tra kernel image
ls /boot/vmlinuz-linux
ls /boot/initramfs-linux.img
ls /boot/intel-ucode.img

# Kiểm tra GRUB config
head -30 /boot/grub/grub.cfg

# Kiểm tra NVIDIA modules trong initramfs
lsinitcpio /boot/initramfs-linux.img | grep nvidia

# Kiểm tra boot entry
efibootmgr -v
```

---

## Cách B: Dùng Archinstall

### Trong quá trình archinstall

Archinstall cài GRUB tự động nếu bạn chọn GRUB trong menu `Bootloader`.

**Cấu hình kernel parameters trong archinstall:**

Khi đến mục `Bootloader`:
1. Chọn `Grub` (mặc định).
2. Mục `Kernel parameters`: Nhập thêm:
   ```
   nvidia-drm.modeset=1 nowatchdog
   ```
   (Không ghi đè `loglevel=3 quiet` mặc định — archinstall tự thêm).

### Kiểm tra sau archinstall

Sau khi archinstall hoàn tất, kiểm tra:

```bash
arch-chroot /mnt

# Kiểm tra kernel parameters
cat /etc/default/grub | grep GRUB_CMDLINE_LINUX_DEFAULT

# Kiểm tra mkinitcpio
cat /etc/mkinitcpio.conf | grep MODULES

# Kiểm tra GRUB config
grep "nvidia-drm" /boot/grub/grub.cfg
```

### Nếu archinstall chưa cấu hình đúng

Cấu hình thủ công file `/etc/default/grub` và `/etc/mkinitcpio.conf` như cách A,
sau đó chạy:

```bash
mkinitcpio -P
grub-mkconfig -o /boot/grub/grub.cfg
```

---

## Xử lý lỗi

### "grub-install: error: cannot find EFI directory"

ESP chưa mount đúng:

```bash
# Kiểm tra ESP đã mount
mount | grep efi

# Mount ESP
mount /dev/nvme0n1p1 /efi
```

### "Failed to register the EFI boot entry"

Một số máy Lenovo không cho ghi NVRAM. Dùng `--no-nvram`:

```bash
grub-install --target=x86_64-efi --efi-directory=/efi \
  --bootloader-id=GRUB --no-nvram
```

Sau đó thêm boot entry thủ công khi reboot (vào BIOS/UEFI → Add boot option)
hoặc dùng efibootmgr từ live USB:

```bash
efibootmgr -c -d /dev/nvme0n1 -p 1 -L GRUB -l \EFI\GRUB\grubx64.efi
```

### GRUB không thấy kernel

```bash
# Kiểm tra /boot
ls /boot/

# Nếu /boot trống → subvolume @ chưa mount đúng
# Mount lại subvolume @ và chạy grub-mkconfig
```

### "mkinitcpio: command not found"

```bash
# Cài mkinitcpio (thường có sẵn trong base)
pacman -S mkinitcpio
```

### NVIDIA module không load

Sau reboot, nếu NVIDIA không hoạt động:

```bash
# Kiểm tra module
lsmod | grep nvidia

# Nếu không có, load thủ công
modprobe nvidia
modprobe nvidia_drm

# Kiểm tra log
dmesg | grep nvidia
```

---

## Tổng kết

| Hạng mục | Giá trị |
|---|---|
| Bootloader | GRUB 2 (UEFI) |
| ESP mount | `/efi` |
| Boot entry | `GRUB` |
| Kernel params | `loglevel=3 quiet nvidia-drm.modeset=1 nowatchdog` |
| NVIDIA modules | `nvidia nvidia_modeset nvidia_uvm nvidia_drm` |
| initramfs | Đã rebuild với NVIDIA modules |
| grub-btrfs | Đã cài (tùy chọn) |

GRUB đã được cài vào ESP và đăng ký với firmware UEFI. Sẵn sàng reboot vào
Arch Linux mới cài.

## Sau reboot

```bash
# Kiểm tra GRUB menu
systemctl reboot

# Chọn GRUB trong UEFI boot menu (F12)
# Chọn "Arch Linux" trong GRUB menu
# Login với user đã tạo
```
