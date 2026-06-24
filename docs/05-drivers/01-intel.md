# Intel GPU — UHD Graphics

## Mục tiêu

Cấu hình Intel UHD Graphics (Alder Lake) cho hiệu suất tối ưu trên Lenovo LOQ 15IAX9.

## Kiến thức nền

### iGPU Intel

Intel UHD Graphics trên CPU i5-12450HX là GPU tích hợp (integrated), dùng chung RAM với hệ thống. Nó phụ trách:

- Hiển thị desktop hàng ngày
- Xem video (có hardware acceleration)
- Chạy các ứng dụng đồ họa nhẹ

### Modesetting (KMS)

Kernel Mode Setting (KMS) là cơ chế kernel quản lý độ phân giải, màu sắc và các thông số màn hình. Intel driver trong kernel (`i915`) tự động kích hoạt KMS.

### Tearing

Tearing (xé hình) xảy ra khi GPU render frame mới trong khi màn hình đang refresh → hình ảnh bị xé ngang. Với Intel iGPU, tearing thường do thiếu vsync — compositor (picom) đã giải quyết vấn đề này.

## Các bước thực hiện

### Bước 1: Xác nhận Intel GPU

```bash
lspci | grep -E "VGA|3D|Display"
```

Output:

```
00:02.0 VGA compatible controller: Intel Corporation Alder Lake-P Integrated Graphics Controller (rev 0c)
```

### Bước 2: Cài driver Intel

```bash
pacman -S mesa lib32-mesa vulkan-intel lib32-vulkan-intel intel-media-driver
```

| Gói | Vai trò |
|---|---|
| `mesa` | OpenGL driver mã nguồn mở |
| `lib32-mesa` | OpenGL 32-bit (cho ứng dụng 32-bit như Steam) |
| `vulkan-intel` | Vulkan driver cho Intel (cần cho nhiều game và ứng dụng) |
| `lib32-vulkan-intel` | Vulkan 32-bit |
| `intel-media-driver` | Hardware video acceleration (VA-API) — iHD driver |

### Bước 3: Cấu hình kernel parameters (khuyến nghị)

Thêm vào `/etc/default/grub` (dòng `GRUB_CMDLINE_LINUX_DEFAULT`):

```bash
i915.enable_psr=0 i915.enable_fbc=1
```

| Parameter | Ý nghĩa |
|---|---|
| `i915.enable_psr=0` | Tắt Panel Self Refresh (gây flicker/nhấp nháy trên nhiều laptop) |
| `i915.enable_fbc=1` | Bật Frame Buffer Compression (tiết kiệm băng thông + điện năng) |

Sau đó:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

### Bước 4: Kiểm tra hardware acceleration

```bash
sudo pacman -S libva-utils
vainfo
```

Output mẫu:

```
vainfo: VA-API version: 1.20 (libva 2)
vainfo: Driver version: Intel iHD driver for Intel(R) Gen Graphics - 24.x.x
vainfo: Supported profile and entrypoints:
      VAProfileNone                   : VAEntrypointVideoProc
      VAProfileMPEG2Simple            : VAEntrypointVLD
      VAProfileMPEG2Main              : VAEntrypointVLD
      ...
```

Nếu `vainfo` không thấy driver iHD:

```bash
# Đảm bảo dùng iHD driver (không phải i965 cũ)
export LIBVA_DRIVER_NAME=iHD
vainfo
```

Để thiết lập vĩnh viễn, thêm vào `~/.bashrc` hoặc `~/.config/environment.d/va-api.conf`:

```ini
LIBVA_DRIVER_NAME=iHD
```

### Bước 5: Kiểm tra OpenGL

```bash
pacman -S mesa-utils
glxinfo | grep "OpenGL renderer"
```

Output:

```
OpenGL renderer string: Mesa Intel(R) Graphics (ADL GT2)
```

```bash
glxinfo | grep "OpenGL version"
```

Output:

```
OpenGL version string: 4.6 (Compatibility Profile) Mesa 24.x.x
```

## Cấu hình cho laptop

### Brightness

Điều chỉnh độ sáng màn hình:

```bash
pacman -S brightnessctl

# Giảm độ sáng
brightnessctl set 50%

# Tăng
brightnessctl set +5%

# Giảm
brightnessctl set 5%-
```

Phím tắt trong `~/.config/sxhkd/sxhkdrc`:

```bash
XF86MonBrightnessUp
    brightnessctl set +5%
XF86MonBrightnessDown
    brightnessctl set 5%-
```

### Backlight

Nếu phím brightness không hoạt động, kiểm tra:

```bash
ls /sys/class/backlight/
```

Nếu có `intel_backlight` → backlight driver đang hoạt động.

## Troubleshooting

### Màn hình nhấp nháy (flickering)

- Tắt PSR: `i915.enable_psr=0` trong kernel params (phổ biến nhất)
- Tắt FBC nếu cần: `i915.enable_fbc=0`

### Độ sáng không điều chỉnh được

```bash
# Kiểm tra phím tắt có gửi event không
acpi_listen
# Nhấn phím brightness → xem có event XF86MonBrightnessUp/Down không

# Nếu acpi không nhận, thêm kernel parameter:
acpi_backlight=vendor
```

### OpenGL không hoạt động

```bash
# Kiểm tra mesa đã cài
pacman -Q mesa
# Reinstall nếu cần
pacman -S mesa mesa-utils
```

### VA-API không hoạt động trong ứng dụng

```bash
# Kiểm tra với Chromium/Firefox
LIBVA_DRIVER_NAME=iHD chrome --use-gl=angle --use-angle=swiftshader
```

## Tổng kết

- Intel GPU đã được cài driver Mesa + Vulkan
- Hardware acceleration (VA-API) qua iHD driver
- Kernel parameters tối ưu: PSR tắt, FBC bật
- Brightness hoạt động qua phím chức năng + brightnessctl
