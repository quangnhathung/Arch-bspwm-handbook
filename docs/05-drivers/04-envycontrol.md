# Hybrid Graphics — Intel + NVIDIA

## Mục tiêu

Hiểu và cấu hình hybrid graphics: Intel UHD + NVIDIA RTX 4050 hoạt động
cùng nhau trên cùng một máy.

## Kiến thức nền

### Hybrid Graphics là gì?

Laptop có hai GPU:
- **Integrated (iGPU)**: Intel UHD Graphics — tiết kiệm pin, đủ cho desktop và video.
- **Discrete (dGPU)**: NVIDIA RTX 4050 — mạnh, dùng cho render, game (sau này).

Hai GPU kết nối với nhau và với màn hình qua các đường:

```
Intel iGPU ────┤ Màn hình laptop (eDP)
NVIDIA dGPU ───┤
```

Trên máy Lenovo LOQ, màn hình laptop được kết nối trực tiếp với Intel iGPU.
NVIDIA render ra framebuffer rồi copy qua Intel để hiển thị (kiến trúc Optimus).

### Cách hoạt động

```
Ứng dụng → Nvidia GPU (render) → framebuffer → Intel GPU (display) → màn hình
```

### Các chế độ hoạt động

| Chế độ | iGPU | dGPU | Pin | Hiệu năng |
|---|---|---|---|---|
| Intel Only | Hoạt động | Tắt | Tốt nhất | Thấp (dùng Intel) |
| NVIDIA Only | Tắt | Hoạt động | Thấp nhất | Cao nhất |
| Hybrid | Hoạt động | Hoạt động | Trung bình | Linh hoạt |
| On-Demand | Hoạt động | Ngủ, thức khi cần | Tốt | Cao khi cần |

## Các bước thực hiện

### Bước 1: Cài driver (đã làm)

- Intel driver: đã cài ở bài intel.md.
- NVIDIA driver: đã cài ở bài nvidia.md.

### Bước 2: Xác nhận cả hai GPU

```bash
lspci | grep -E "VGA|3D"
```

Output:

```
00:02.0 VGA compatible controller: Intel Corporation Alder Lake-P Integrated Graphics Controller
01:00.0 3D controller: NVIDIA Corporation AD107M [GeForce RTX 4050 Max-Q] (rev a1)
```

### Bước 3: Cấu hình PRIME (NVIDIA render offload)

Tạo file Xorg để cho phép render offload:

```bash
vim /etc/X11/xorg.conf.d/10-nvidia-prime.conf
```

Nội dung:

```
Section "ServerLayout"
    Identifier "layout"
    Option "AllowEmptyInitialConfiguration"
EndSection

Section "Device"
    Identifier "intel"
    Driver "modesetting"
    BusID  "PCI:0:2:0"
    Option "AccelMethod" "none"
EndSection

Section "Device"
    Identifier "nvidia"
    Driver "nvidia"
    BusID  "PCI:1:0:0"
    Option "AllowEmptyInitialConfiguration"
    Option "AllowExternalGpus"
    Option "PrimaryGPU" "no"
EndSection

Section "Screen"
    Identifier "screen"
    Device "intel"
EndSection
```

Lưu ý: `BusID` phải đúng với output của `lspci`.
- Intel: `00:02.0` → `PCI:0:2:0`
- NVIDIA: `01:00.0` → `PCI:1:0:0`

Định dạng BusID: `PCI:domain:bus:device:function`, thường là `PCI:0:2:0`.

### Bước 4: Cấu hình biến môi trường

Thêm vào `/etc/environment` hoặc `~/.bashrc`:

```bash
# Dùng Intel làm primary display
export __GLX_VENDOR_LIBRARY_NAME=mesa
export DRI_PRIME=0

# Khi muốn chạy ứng dụng trên NVIDIA
# __NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia
```

### Bước 5: Kiểm tra chế độ hiện tại

```bash
# Xem GPU nào đang active cho display
glxinfo | grep "OpenGL renderer"
```

Nếu đang dùng Intel:

```
OpenGL renderer string: Mesa Intel(R) Graphics (ADL GT2)
```

Nếu đang dùng NVIDIA:

```
OpenGL renderer string: NVIDIA GeForce RTX 4050 Laptop GPU/PCIe/SSE2
```

### Bước 6: Chạy ứng dụng trên NVIDIA (render offload)

```bash
# Render offload method
__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia alacritty

# Hoặc
prime-run alacritty
```

`prime-run` là script từ gói `nvidia-prime`:

```bash
pacman -S nvidia-prime
```

Sau đó chỉ cần:

```bash
prime-run alacritty
```

## Kiểm tra ứng dụng đang dùng GPU nào

```bash
# Trong ứng dụng đó (ví dụ glxgears)
prime-run glxgears

# Xem process sử dụng NVIDIA
nvidia-smi
```

## Chọn chế độ mặc định

### Chạy toàn bộ desktop trên NVIDIA

```bash
# Sửa Xorg config để PrimaryGPU=yes
vim /etc/X11/xorg.conf.d/10-nvidia.conf
# Option "PrimaryGPU" "yes"
```

### Chạy toàn bộ desktop trên Intel

```bash
# Đảm bảo không có file Xorg nào set PrimaryGPU cho NVIDIA
# Xóa hoặc comment dòng đó
```

## Troubleshooting

### Màn hình đen khi chạy NVIDIA

- BusID sai → kiểm tra `lspci` và sửa.
- Thiếu kernel parameter → kiểm tra `nvidia-drm.modeset=1`.

### Render offload không hoạt động

```bash
# Kiểm tra biến môi trường
echo $__NV_PRIME_RENDER_OFFLOAD

# Nếu không set → set thủ công
export __NV_PRIME_RENDER_OFFLOAD=1
export __GLX_VENDOR_LIBRARY_NAME=nvidia
```

### Ứng dụng không dùng được GPU rời

- Kiểm tra ứng dụng có hỗ trợ GPU không.
- Dùng `nvidia-smi` để kiểm tra process.

## Tổng kết

- Hybrid graphics hoạt động với Intel (display) + NVIDIA (render).
- Render offload cho phép chọn GPU cho từng ứng dụng.
- Có thể chạy toàn bộ desktop trên Intel hoặc NVIDIA.
- prime-run là cách dễ nhất để chạy ứng dụng trên NVIDIA.
