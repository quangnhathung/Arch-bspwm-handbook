# GRUB Bootloader

## Mục tiêu

Cài đặt GRUB làm bootloader cho UEFI và cấu hình kernel parameters.

## Kiến thức nền

### Bootloader là gì?

Bootloader là chương trình nhỏ chạy đầu tiên khi máy khởi động. Nó load kernel
Linux từ ổ cứng vào RAM và chuyển quyền điều khiển cho kernel.

### GRUB (Grand Unified Bootloader)

GRUB là bootloader phổ biến nhất trên Linux. GRUB 2 (phiên bản hiện tại) hỗ trợ:

- UEFI và Legacy BIOS
- BTRFS, ext4, XFS, ZFS
- LVM, encryption
- Boot từ network (PXE)
- Kernel parameters

### EFI System Partition (ESP)

ESP là partition FAT32 chứa bootloader. Khi máy bật, UEFI firmware đọc ESP,
tìm file `.efi` trong `EFI/` và chạy nó.

```
ESP (/efi)
└── EFI/
    └── GRUB/
        └── grubx64.efi    (GRUB binary)
```

### grub-install làm gì?

1. Copy file `grubx64.efi` vào `ESP/EFI/GRUB/`.
2. Tạo file cấu hình GRUB cơ bản trong ESP.
3. Đăng ký GRUB với firmware UEFI (thêm vào boot menu).

### grub-mkconfig làm gì?

1. Quét các kernel có trong `/boot`.
2. Quét các OS khác (nếu có dual boot).
3. Sinh file `/boot/grub/grub.cfg` chứa menu boot.

## Các bước thực hiện

### Bước 1: Cài gói GRUB

```bash
pacman -S grub efibootmgr
```

- `grub`: Bootloader.
- `efibootmgr`: Công cụ quản lý EFI boot entries (cần cho grub-install).

### Bước 2: Cài GRUB vào ESP

```bash
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB
```

Giải thích:
- `--target=x86_64-efi`: Cài cho UEFI 64-bit.
- `--efi-directory=/efi`: Thư mục ESP đã mount.
- `--bootloader-id=GRUB`: Tên hiển thị trong UEFI boot menu.

### Bước 3: Cấu hình kernel parameters

Kernel parameters là các option truyền cho kernel khi boot. Chúng ta cần:

- `nvidia-drm.modeset=1`: Cho NVIDIA DRM modesetting (chống tearing, cần cho Wayland/nvidia).
- `nowatchdog`: Tắt watchdog timer (tiết kiệm pin, giảm CPU usage).

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

**Giải thích từng parameter:**

| Parameter | Ý nghĩa |
|---|---|
| `loglevel=3` | Chỉ hiện log từ mức 3 (error) trở lên — bớt rác |
| `quiet` | Ẩn kernel messages khi boot |
| `nvidia-drm.modeset=1` | Kích hoạt DRM modesetting cho NVIDIA GPU |
| `nowatchdog` | Tắt NMI watchdog (tiết kiệm năng lượng) |

#### NVIDIA parameter quan trọng

`nvidia-drm.modeset=1` bắt buộc cho:

- Chống screen tearing.
- Hoạt động đúng của NVIDIA driver.
- Tương lai có thể chạy Wayland trên NVIDIA.

### Bước 4: Sinh GRUB config

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

Kết quả mong đợi:

```
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-linux
Found initrd image: /boot/intel-ucode.img /boot/initramfs-linux.img
Warning: os-prober will not be used to detect other bootable partitions ...
done
```

Nếu không thấy `Found linux image`, kiểm tra `ls /boot/`.

### Bước 5: Kiểm tra

```bash
# Kiểm tra file GRUB trong ESP
ls /efi/EFI/GRUB/grubx64.efi

# Kiểm tra boot entry
efibootmgr -v
```

Output của `efibootmgr` sẽ có dòng:

```
Boot0001* GRUB    HD(1,GPT,xxx,0x800,0x80000)/File(\EFI\GRUB\grubx64.efi)
```

## Các kernel parameters có thể cần thêm

### Cho Intel GPU

```
i915.enable_psr=0     # Tắt Panel Self Refresh (nếu gặp lỗi màn hình)
i915.enable_fbc=1     # Bật Frame Buffer Compression
```

### Cho laptop

```
acpi_osi=             # Một số laptop cần để ACPI hoạt động đúng
acpi_backlight=vendor # Cho phép điều chỉnh độ sáng bằng phím chức năng
```

### Khi cần debug

```
systemd.log_level=debug
systemd.log_target=kmsg
```

## Nếu có lỗi

### "grub-install: error: cannot find EFI directory"

- ESP chưa mount đúng: `ls /efi/EFI` phải tồn tại.
- Mount ESP: `mount /dev/nvme0n1p1 /efi`.

### "Failed to register the EFI boot entry"

- Dùng `--no-nvram` nếu máy không cho ghi NVRAM:

```bash
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB --no-nvram
```

- Boot entry vẫn có thể thêm bằng tay: `efibootmgr -c -d /dev/nvme0n1 -p 1 -L GRUB -l \EFI\GRUB\grubx64.efi`.

### GRUB không thấy kernel

- `/boot` phải có `vmlinuz-linux` và `initramfs-linux.img`.
- Nếu dùng BTRFS, đảm bảo subvolume `@` được mount đúng.

## Tổng kết

- GRUB đã được cài vào ESP và đăng ký với firmware UEFI.
- Kernel parameters đã được cấu hình cho NVIDIA và laptop.
- Sẵn sàng reboot vào Arch Linux mới cài.
