# NVIDIA — GeForce RTX 4050

## Mục tiêu

Cài đặt và cấu hình driver NVIDIA cho GeForce RTX 4050 Laptop GPU.

## Kiến thức nền

### NVIDIA Optimus

NVIDIA Optimus là công nghệ chuyển đổi giữa Intel iGPU và NVIDIA dGPU.
Ứng dụng nhẹ dùng Intel để tiết kiệm pin, ứng dụng nặng có thể chạy trên NVIDIA.

Trên Linux, Optimus có thể được quản lý qua:

1. **NVIDIA Prime**: Chạy toàn bộ desktop trên NVIDIA hoặc Intel, chuyển đổi bằng
   envycontrol hoặc nvidia-prime.
2. **NVIDIA On-Demand**: Ứng dụng tự chọn GPU (cần cấu hình).
3. **Render offload**: NVIDIA làm render server, Intel làm display.

### Modesetting

`nvidia-drm.modeset=1` là kernel parameter bắt buộc để:
- Chống tearing.
- Hỗ trợ DRM (Direct Rendering Manager).
- Cho phép Wayland hoạt động với NVIDIA (trong tương lai).

### Nouveau vs Proprietary

| Driver | Loại | 3D Performance | CUDA | Hỗ trợ RTX 4050 |
|---|---|---|---|---|
| Nouveau | Mã nguồn mở | Kém | Không | Chưa tốt |
| NVIDIA (proprietary) | Đóng | Tối ưu | Có | Tốt |

Chúng ta dùng driver NVIDIA chính thức (proprietary).

## Các bước thực hiện

### Bước 1: Cài driver NVIDIA

```bash
pacman -S nvidia nvidia-utils nvidia-settings
```

| Gói | Vai trò |
|---|---|
| `nvidia` | Kernel module NVIDIA |
| `nvidia-utils` | Thư viện và công cụ (nvidia-smi) |
| `nvidia-settings` | GUI để cấu hình NVIDIA |

### Bước 2: Cấu hình mkinitcpio

Để NVIDIA module được load sớm khi boot:

```bash
vim /etc/mkinitcpio.conf
```

Tìm dòng `MODULES=(...)` và thêm `nvidia nvidia_modeset nvidia_uvm nvidia_drm`:

```
MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm)
```

Sau đó rebuild initramfs:

```bash
mkinitcpio -P
```

### Bước 3: Cấu hình GRUB (đã làm ở bước cài)

Đảm bảo `/etc/default/grub` có:

```
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet nvidia-drm.modeset=1 nowatchdog"
```

Sinh lại GRUB config:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

### Bước 4: Tạo file Xorg cho NVIDIA

```bash
vim /etc/X11/xorg.conf.d/10-nvidia.conf
```

Nội dung:

```
Section "OutputClass"
    Identifier "nvidia"
    MatchDriver "nvidia-drm"
    Driver "nvidia"
    Option "AllowEmptyInitialConfiguration"
    Option "PrimaryGPU" "yes"
    Option "SLI" "Off"
    Option "MetaModes" "nvidia-auto-select +0+0 {ForceFullCompositionPipeline=On}"
EndSection
```

Giải thích:
- `PrimaryGPU "yes"`: Dùng NVIDIA làm GPU chính.
- `ForceFullCompositionPipeline=On`: Chống tearing triệt để.
- `SLI "Off"`: Tắt SLI (máy laptop chỉ có 1 GPU NVIDIA).

### Bước 5: Cấu hình Picom cho NVIDIA

Trong `~/.config/picom/picom.conf`:

```ini
vsync = true;
unredir-if-possible = false;
```

`unredir-if-possible = false` rất quan trọng: khi true, picom tắt compositor
khi cửa sổ fullscreen → screen tearing trên NVIDIA.

### Bước 6: Kiểm tra hoạt động

#### nvidia-smi

```bash
nvidia-smi
```

Output:

```
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 550.xx.xx    Driver Version: 550.xx.xx    CUDA Version: 12.x    |
|-------------------------------+----------------------+----------------------+
| GPU  Name            TCC/WDDM | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ...  Off  | 00000000:01:00.0  On |                  N/A |
|  0%   45C    P0    25W /  95W |    123MiB /  6144MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
```

#### glxinfo

```bash
pacman -S mesa-utils
glxinfo | grep "OpenGL renderer"
```

Output:

```
OpenGL renderer string: NVIDIA GeForce RTX 4050 Laptop GPU/PCIe/SSE2
```

### Bước 7: Cấu hình PRIME (chọn GPU)

```bash
# Xem GPU nào đang active
cat /sys/kernel/debug/dri/0/name

# Đặt GPU mặc định cho ứng dụng
# Chạy ứng dụng trên NVIDIA:
__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia alacritty

# Kiểm tra ứng dụng đang dùng GPU nào:
glxheads
```

Để dễ dùng, tạo alias:

```bash
echo 'alias nvrun="__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia"' >> ~/.bashrc
echo 'alias intel="__NV_PRIME_RENDER_OFFLOAD=0"' >> ~/.bashrc
```

### Bước 8: Cấu hình NVIDIA persistence (tùy chọn)

```bash
systemctl enable nvidia-persistenced
```

Giúp NVIDIA module không bị unload khi không dùng, tăng tốc độ phản hồi.

## Xác nhận không còn tearing

```bash
# Test bằng glxgears (nếu có)
vblank_mode=0 glxgears

# Nếu không bị xé hình khi chạy → OK
```

## Troubleshooting

### nvidia-smi không thấy GPU

```bash
# Kiểm tra module
lsmod | grep nvidia
# Nếu không thấy → load thủ công
modprobe nvidia
modprobe nvidia_modeset
modprobe nvidia_uvm
modprobe nvidia_drm
```

### Lỗi "NVIDIA kernel module is missing"

```bash
# Fallback to nvidia-dkms
pacman -S nvidia-dkms
# DKMS tự động rebuild module mỗi khi kernel update
```

### Screen tearing vẫn còn

1. Kiểm tra `ForceFullCompositionPipeline=On` trong Xorg config.
2. Kiểm tra picom `unredir-if-possible = false`.
3. Thêm `nvidia-drm.modeset=1` trong kernel params.

### NVIDIA không hoạt động sau kernel update

```bash
# Nếu dùng nvidia (non-DKMS)
pacman -S nvidia
mkinitcpio -P
reboot

# Nếu dùng nvidia-dkms, DKMS sẽ tự rebuild
```

## Tổng kết

- NVIDIA driver đã được cài với kernel module.
- Kernel parameters đã cấu hợp GRUB.
- Xorg config với ForceFullCompositionPipeline chống tearing.
- nvidia-smi hoạt động và thấy GPU.
- PRIME render offload có thể chọn GPU cho từng ứng dụng.
