# Hybrid Graphics — Intel + NVIDIA

## Mục tiêu

Hiểu và cấu hình hybrid graphics: Intel UHD Graphics + NVIDIA RTX 4050 hoạt động cùng nhau trên Lenovo LOQ 15IAX9.

## Kiến thức nền

### Hybrid Graphics là gì?

Laptop có hai GPU:

- **Integrated (iGPU)**: Intel UHD Graphics — tiết kiệm pin, đủ cho desktop và video
- **Discrete (dGPU)**: NVIDIA RTX 4050 — mạnh, dùng cho render, game

Trên Lenovo LOQ 15IAX9, màn hình laptop (eDP) được kết nối trực tiếp với Intel iGPU. NVIDIA render ra framebuffer rồi copy qua Intel để hiển thị (kiến trúc Optimus truyền thống).

```
Intel iGPU ──── Màn hình laptop (eDP)
NVIDIA dGPU ───→ Framebuffer → Intel iGPU → Màn hình
```

### Các chế độ hoạt động

| Chế độ | iGPU | dGPU | Pin | Hiệu năng |
|---|---|---|---|---|
| Intel Only | Hoạt động | Tắt | Tốt nhất | Thấp (Intel) |
| NVIDIA Only | Tắt | Hoạt động | Thấp nhất | Cao nhất |
| Hybrid | Hoạt động | Hoạt động | Trung bình | Linh hoạt (render offload) |

## Các bước thực hiện

### Bước 1: Xác nhận cả hai GPU

```bash
lspci | grep -E "VGA|3D"
```

Output:

```
00:02.0 VGA compatible controller: Intel Corporation Alder Lake-P Integrated Graphics Controller
01:00.0 3D controller: NVIDIA Corporation AD107M [GeForce RTX 4050 Max-Q] (rev a1)
```

### Bước 2: Cấu hình Xorg cho hybrid

Tạo file Xorg cho phép render offload:

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

Lưu ý BusID — phải đúng với output của `lspci`:

- Intel: `00:02.0` → `PCI:0:2:0`
- NVIDIA: `01:00.0` → `PCI:1:0:0`

Định dạng BusID: `PCI:domain:bus:device:function`. Nếu `lspci` hiển thị khác (vd `0000:01:00.0`), chuyển đổi: `0000:01:00.0` → `PCI:1:0:0`.

### Bước 3: Kiểm tra chế độ hiện tại

```bash
glxinfo | grep "OpenGL renderer"
```

Intel:

```
OpenGL renderer string: Mesa Intel(R) Graphics (ADL GT2)
```

NVIDIA:

```
OpenGL renderer string: NVIDIA GeForce RTX 4050 Laptop GPU/PCIe/SSE2
```

### Bước 4: Cấu hình biến môi trường

Thiết lập mặc định trong `~/.config/environment.d/gpu.conf` (cách hiện đại, không dùng `/etc/environment`):

```ini
# Mặc định dùng Intel
__GLX_VENDOR_LIBRARY_NAME=mesa
DRI_PRIME=0
```

Khi muốn chạy ứng dụng trên NVIDIA:

```bash
__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia ứng_dụng
```

### Bước 5: prime-run script

Tạo script `/usr/local/bin/prime-run` (thủ công) hoặc cài từ `nvidia-prime`:

```bash
# Cài từ extra repo
pacman -S nvidia-prime

# Nội dung prime-run (tương đương):
# __NV_PRIME_RENDER_OFFLOAD=1 __VK_LAYER_NV_optimus=NVIDIA_only __GLX_VENDOR_LIBRARY_NAME=nvidia "$@"
```

Sử dụng:

```bash
prime-run alacritty
prime-run blender
prime-run steam
```

Nếu không cài `nvidia-prime`, tạo alias trong `~/.bashrc`:

```bash
alias prime-run='__NV_PRIME_RENDER_OFFLOAD=1 __VK_LAYER_NV_optimus=NVIDIA_only __GLX_VENDOR_LIBRARY_NAME=nvidia'
```

## Kiểm tra ứng dụng đang dùng GPU nào

```bash
# Khi chạy ứng dụng
prime-run glxgears

# Kiểm tra process trên NVIDIA
nvidia-smi

# Hoặc trong ứng dụng, chạy:
glxinfo | grep "OpenGL renderer"
```

## Chọn chế độ mặc định

### Chạy toàn bộ desktop trên NVIDIA

```bash
vim /etc/X11/xorg.conf.d/10-nvidia.conf
# Option "PrimaryGPU" "yes"
```

### Chạy toàn bộ desktop trên Intel

```bash
# Xóa hoặc comment dòng PrimaryGPU trong file NVIDIA
# Đảm bảo Screen section chỉ vào Intel device
```

## Troubleshooting

### Màn hình đen khi dùng NVIDIA

- **BusID sai**: Kiểm tra `lspci` và sửa BusID trong Xorg config
- **Thiếu kernel parameter**: Kiểm tra `nvidia-drm.modeset=1` trong GRUB
- **Boot recovery**: Thêm `nomodeset` vào GRUB khi boot

### Render offload không hoạt động

```bash
set | grep NVIDIA

__NV_PRIME_RENDER_OFFLOAD=1
__GLX_VENDOR_LIBRARY_NAME=nvidia
```

Nếu biến môi trường không được set → set thủ công trước khi chạy ứng dụng.

### Ứng dụng không dùng được NVIDIA

- Kiểm tra ứng dụng có hỗ trợ GPU không (vd `glxinfo` → OpenGL renderer)
- Dùng `nvidia-smi` để kiểm tra process
- Vulkan apps: cần `__VK_LAYER_NV_optimus=NVIDIA_only`

### lspci không thấy NVIDIA

```bash
# Kiểm tra trong BIOS → GPU có bị disable không
# Kiểm tra power state
cat /sys/bus/pci/devices/0000:01:00.0/power/control
```

## Tổng kết

- Hybrid graphics: Intel hiển thị + NVIDIA render offload
- Xorg config với BusID chính xác cho cả hai GPU
- `prime-run` để chạy ứng dụng trên NVIDIA
- Không dùng `/etc/environment` — dùng `~/.config/environment.d/*.conf`
- Kiểm tra GPU active bằng `glxinfo` và `nvidia-smi`
