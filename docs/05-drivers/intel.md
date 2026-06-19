# Intel GPU — UHD Graphics

## Mục tiêu

Cấu hình Intel UHD Graphics (Alder Lake) cho hiệu suất tối ưu trên laptop.

## Kiến thức nền

### iGPU Intel

Intel UHD Graphics trên CPU i5-12450HX là GPU tích hợp (integrated),
dùng chung RAM với hệ thống. Nó phụ trách:

- Hiển thị desktop hàng ngày.
- Xem video (có hardware acceleration).
- Chạy các ứng dụng đồ họa nhẹ.

### Modesetting (KMS)

Kernel Mode Setting (KMS) là cơ chế kernel quản lý độ phân giải, màu sắc,
và các thông số màn hình. Intel driver trong kernel (`i915`) tự động kích hoạt
KMS.

### Tearing

Tearing (xé hình) xảy ra khi GPU render frame mới trong khi màn hình đang
refresh → hình ảnh bị xé ngang. Với Intel iGPU, tearing thường do thiếu
vsync hoặc compositor (picom đã giải quyết).

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
| `lib32-mesa` | OpenGL 32-bit (cho ứng dụng 32-bit) |
| `vulkan-intel` | Vulkan driver cho Intel (cần cho nhiều ứng dụng) |
| `lib32-vulkan-intel` | Vulkan 32-bit |
| `intel-media-driver` | Hardware video acceleration (VA-API) |

### Bước 3: Cấu hình kernel parameters (tùy chọn)

Thêm vào `/etc/default/grub` (dòng `GRUB_CMDLINE_LINUX_DEFAULT`):

```bash
i915.enable_psr=0 i915.enable_fbc=1
```

| Parameter | Ý nghĩa |
|---|---|
| `i915.enable_psr=0` | Tắt Panel Self Refresh (gây lỗi trên một số laptop) |
| `i915.enable_fbc=1` | Bật Frame Buffer Compression (tiết kiệm băng thông) |

Sau đó:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

### Bước 4: Kiểm tra hardware acceleration

```bash
# Kiểm tra VA-API
sudo pacman -S libva-utils
vainfo

# Output mẫu:
# vainfo: VA-API version: 1.20 (libva 2)
# vainfo: Driver version: Intel iHD driver for Intel(R) Gen Graphics - 24.x.x
# vainfo: Supported profile and entrypoints:
#       VAProfileNone                   : VAEntrypointVideoProc
#       VAProfileMPEG2Simple            : VAEntrypointVLD
#       VAProfileMPEG2Main              : VAEntrypointVLD
#       ...
```

Nếu `vainfo` không thấy driver → `intel-media-driver` chưa đúng.

```bash
# Đảm bảo dùng iHD driver (không phải i965)
export LIBVA_DRIVER_NAME=iHD
vainfo
```

### Bước 5: Kiểm tra OpenGL

```bash
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
# Cài brightnessctl
pacman -S brightnessctl

# Giảm độ sáng
brightnessctl set 50%

# Tăng
brightnessctl set +5%

# Giảm
brightnessctl set 5%-
```

Phím tắt đã cấu hình trong sxhkdrc:

```
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

Nếu có `intel_backlight` hoặc `amdgpu_bl0` → backlight driver đang hoạt động.

## Troubleshooting

### Màn hình nhấp nháy

- Tắt PSR: `i915.enable_psr=0` trong kernel params.
- Tắt FBC: `i915.enable_fbc=0`.

### Độ sáng không điều chỉnh được

```bash
# Thử acpi_backlight
acpi_listen
# Nhấn phím brightness → xem có event không
# Thêm kernel parameter:
acpi_backlight=vendor
```

### OpenGL không hoạt động

```bash
# Kiểm tra mesa
pacman -Q mesa
# Reinstall nếu cần
pacman -S mesa
```

## Tổng kết

- Intel GPU đã được cài driver Mesa + Vulkan.
- Hardware acceleration (VA-API) đã được kích hoạt.
- Kernel parameters đã được tối ưu cho laptop.
- Brightness hoạt động qua phím chức năng.
