# NVIDIA — GeForce RTX 4050

## Mục tiêu

Cài đặt và cấu hình driver NVIDIA cho GeForce RTX 4050 Laptop GPU (Ada Lovelace) trên Lenovo LOQ 15IAX9.

## Kiến thức nền

### NVIDIA Optimus

NVIDIA Optimus là công nghệ chuyển đổi giữa Intel iGPU và NVIDIA dGPU. Ứng dụng nhẹ dùng Intel để tiết kiệm pin, ứng dụng nặng chạy trên NVIDIA.

Trên Linux, Optimus được quản lý qua:

1. **NVIDIA Prime**: Render offload — Intel hiển thị, NVIDIA render khi cần
2. **EnvyControl**: Chuyển đổi giữa Intel-only, NVIDIA-only, hybrid
3. **Render offload thủ công**: Dùng biến môi trường `__NV_PRIME_RENDER_OFFLOAD`

### nvidia-open vs nvidia

| Driver | Kiến trúc | Khuyến nghị | Ghi chú |
|---|---|---|---|
| `nvidia-open` | Mã nguồn mở (kernel module) | **RTX 30/40 series** | Module chính thức từ NVIDIA, tương thích Turing+ |
| `nvidia` | Mã nguồn đóng | RTX 20 series trở xuống | Legacy, vẫn dùng được nhưng không khuyến nghị cho RTX 4050 |

RTX 4050 là Ada Lovelace (kiến trúc mới nhất) → **dùng `nvidia-open`**.

### Modesetting

`nvidia-drm.modeset=1` là kernel parameter bắt buộc để:

- Chống tearing
- Hỗ trợ DRM (Direct Rendering Manager)
- Tương thích Wayland (trong tương lai)

## Các bước thực hiện

### Bước 1: Cài driver NVIDIA

```bash
pacman -S nvidia-open nvidia-utils nvidia-settings
```

| Gói | Vai trò |
|---|---|
| `nvidia-open` | Kernel module NVIDIA mở (cho RTX 30/40 series) |
| `nvidia-utils` | Thư viện và công cụ CLI (`nvidia-smi`) |
| `nvidia-settings` | GUI cấu hình NVIDIA |

> **Tùy chọn**: Dùng `nvidia-open-dkms` nếu muốn module tự động rebuild mỗi khi kernel được cập nhật (khuyến nghị cho kernel tùy chỉnh hoặc kernel LTS):
> ```bash
> pacman -S nvidia-open-dkms nvidia-utils nvidia-settings
> ```

### Bước 2: Cấu hình mkinitcpio

Thêm module NVIDIA vào initramfs để load sớm khi boot:

```bash
vim /etc/mkinitcpio.conf
```

Tìm dòng `MODULES=(...)` và thêm:

```
MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm)
```

Sau đó rebuild initramfs:

```bash
mkinitcpio -P
```

### Bước 3: Cấu hình GRUB

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

- `PrimaryGPU "yes"`: Dùng NVIDIA làm GPU chính
- `ForceFullCompositionPipeline=On`: Chống tearing triệt để
- `SLI "Off"`: Tắt SLI (laptop chỉ có 1 GPU NVIDIA)

### Bước 5: Cấu hình Picom cho NVIDIA

Trong `~/.config/picom/picom.conf`:

```
vsync = true;
unredir-if-possible = false;
```

`unredir-if-possible = false` rất quan trọng: khi true, picom tắt compositor lúc fullscreen → screen tearing trên NVIDIA.

### Bước 6: Kiểm tra hoạt động

#### nvidia-smi

```bash
nvidia-smi
```

Output:

```
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 610.xx.xx    Driver Version: 610.xx.xx    CUDA Version: 12.x    |
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
glxinfo | grep "OpenGL renderer"
```

Output:

```
OpenGL renderer string: NVIDIA GeForce RTX 4050 Laptop GPU/PCIe/SSE2
```

### Bước 7: Cấu hình PRIME render offload

```bash
# Xem GPU nào đang active
cat /sys/kernel/debug/dri/0/name
```

Chạy ứng dụng trên NVIDIA:

```bash
__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia alacritty
```

Để dễ dùng, tạo alias trong `~/.bashrc`:

```bash
alias nvrun='__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia'
alias intel='__NV_PRIME_RENDER_OFFLOAD=0'
```

Nếu đã cài `nvidia-prime` (từ extra repo), có thể dùng `prime-run`:

```bash
pacman -S nvidia-prime
prime-run alacritty
```

`nvidia-prime` cung cấp script `prime-run` tương đương:

```bash
#!/bin/bash
__NV_PRIME_RENDER_OFFLOAD=1 __VK_LAYER_NV_optimus=NVIDIA_only __GLX_VENDOR_LIBRARY_NAME=nvidia "$@"
```

### Bước 8: Cấu hình NVIDIA persistence (tùy chọn)

```bash
systemctl enable nvidia-persistenced
```

Giúp NVIDIA module không bị unload khi không dùng, tăng tốc độ phản hồi khi cần dùng GPU.

## Xác nhận không còn tearing

```bash
# Test bằng glxgears
vblank_mode=0 glxgears
# Nếu không bị xé hình → OK
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
# Nếu dùng nvidia-open → chuyển sang nvidia-open-dkms
pacman -S nvidia-open-dkms
# DKMS tự động rebuild module mỗi khi kernel update
```

### Screen tearing vẫn còn

1. Kiểm tra `ForceFullCompositionPipeline=On` trong Xorg config
2. Kiểm tra picom: `unredir-if-possible = false`
3. Kiểm tra `nvidia-drm.modeset=1` trong kernel params

### NVIDIA không hoạt động sau kernel update

```bash
# Nếu dùng nvidia-open (non-DKMS)
pacman -S nvidia-open
mkinitcpio -P
reboot

# Nếu dùng nvidia-open-dkms → DKMS tự rebuild, chỉ cần mkinitcpio -P
mkinitcpio -P
reboot
```

### Màn hình đen sau reboot

- Kiểm tra GRUB: `nvidia-drm.modeset=1` có còn không
- Boot với `nomodeset` trong GRUB (nhấn E khi boot, thêm nomodeset)
- Vào system, sửa lại config, rebuild initramfs

## Tổng kết

- Dùng `nvidia-open` cho RTX 4050 (Ada Lovelace)
- Kernel modules + GRUB params đã cấu hình
- Xorg config với ForceFullCompositionPipeline chống tearing
- nvidia-smi hoạt động, thấy GPU RTX 4050
- PRIME render offload: chạy ứng dụng trên NVIDIA bằng `prime-run` hoặc env vars
- Khuyến nghị `nvidia-open-dkms` để tự động rebuild theo kernel
