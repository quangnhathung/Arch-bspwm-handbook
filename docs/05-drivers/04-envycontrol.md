# EnvyControl — Quản lý GPU

## Mục tiêu

Cài đặt EnvyControl để dễ dàng chuyển đổi giữa Intel và NVIDIA mà không cần cấu hình thủ công.

## Kiến thức nền

### EnvyControl là gì?

EnvyControl là công cụ CLI cho phép chuyển đổi chế độ đồ họa trên laptop NVIDIA Optimus. Nó tự động:

1. Tạo/sửa file Xorg config phù hợp
2. Cập nhật GRUB kernel parameters
3. Rebuild initramfs
4. Yêu cầu reboot để áp dụng

### Ba chế độ

| Chế độ | Mô tả |
|---|---|
| `intel` | Chỉ Intel iGPU — NVIDIA tắt hoàn toàn, pin tốt nhất |
| `nvidia` | Chỉ NVIDIA dGPU — Intel tắt, hiệu năng cao nhất |
| `hybrid` | Cả hai — Intel display, NVIDIA render offload |

### Rủi ro

- **Phải reboot** sau mỗi lần chuyển
- EnvyControl ghi đè Xorg config → mất cấu hình thủ công trước đó
- Chọn sai chế độ → màn hình đen → cần recovery
- Build từ AUR source mỗi lần cài/gỡ (không có prebuilt binary)

### Lưu ý về envycontrol từ AUR

EnvyControl được cài từ AUR. Khi dùng `git clone + makepkg`, nó sẽ **build từ source** mỗi lần. Dùng `yay` cũng tương tự (tự động clone + build).

## Các bước thực hiện

### Bước 1: Cài EnvyControl

Có hai cách:

```bash
# Cách 1 — yay (khuyến nghị)
yay -S envycontrol

# Cách 2 — thủ công
git clone https://aur.archlinux.org/envycontrol.git
cd envycontrol
makepkg -si
```

### Bước 2: Kiểm tra trạng thái hiện tại

```bash
envycontrol --query
```

Output:

```
Current graphics mode: hybrid
```

### Bước 3: Chuyển sang Intel mode

```bash
sudo envycontrol -s intel
sudo reboot
```

**Kết quả**:

- NVIDIA tắt hoàn toàn → pin tốt nhất
- Hiệu năng 3D kém (dùng Intel)
- Nhiệt độ thấp

### Bước 4: Chuyển sang NVIDIA mode

```bash
sudo envycontrol -s nvidia
sudo reboot
```

**Kết quả**:

- Intel tắt, NVIDIA làm mọi thứ
- Hiệu năng cao nhất
- Pin nhanh hết

### Bước 5: Chuyển sang Hybrid mode (khuyến nghị)

```bash
sudo envycontrol -s hybrid
sudo reboot
```

**Kết quả**:

- Cả hai GPU hoạt động
- Intel display + NVIDIA render offload
- Cân bằng hiệu năng và pin

### Bước 6: Kiểm tra sau khi chuyển

```bash
envycontrol --query
glxinfo | grep "OpenGL renderer"
nvidia-smi
```

## Chi tiết từng chế độ

### Intel mode

```
GPU:      Chỉ Intel
Pin:      ★★★★★ (5/5)
Nhiệt:    Thấp
Hiệu năng: Thấp
Dùng khi: Lướt web, xem phim, văn phòng, di chuyển
```

### NVIDIA mode

```
GPU:      Chỉ NVIDIA
Pin:      ★☆☆☆☆ (1/5)
Nhiệt:    Cao
Hiệu năng: Cao nhất
Dùng khi: Render nặng, game
```

### Hybrid mode (khuyến nghị)

```
GPU:      Intel display + NVIDIA render offload
Pin:      ★★☆☆☆ (2/5)
Nhiệt:    Trung bình
Hiệu năng: Theo nhu cầu (prime-run)
Dùng khi: Sử dụng hàng ngày, thi thoảng cần GPU
```

## Log

```bash
cat /var/log/envycontrol.log
```

## Gỡ EnvyControl

```bash
# Restore Xorg config gốc
sudo envycontrol --restore-xorg

# Gỡ package
sudo pacman -R envycontrol
```

## Troubleshooting

### Màn hình đen sau khi chuyển

**Nguyên nhân**: EnvyControl cấu hình sai Xorg hoặc kernel params.

**Cách xử lý**:

1. Khi boot GRUB, nhấn `E` để sửa dòng kernel
2. Thêm `nomodeset` hoặc `systemd.unit=multi-user.target`
3. Boot vào hệ thống (single-user mode)
4. Chuyển về hybrid:

```bash
sudo envycontrol -s hybrid
sudo reboot
```

5. Nếu vẫn đen, gỡ envycontrol và cấu hình thủ công theo bài 03-hybrid-graphics.md

### EnvyControl báo lỗi module

```bash
# Load module thủ công
modprobe nvidia
modprobe nvidia_modeset
modprobe nvidia_uvm
modprobe nvidia_drm
```

## Khuyến nghị cho Lenovo LOQ 15IAX9

| Tình huống | Chế độ |
|---|---|
| Hàng ngày (desktop, browsing, code) | `hybrid` |
| Di chuyển (tiết kiệm pin) | `intel` |
| Render nặng, CUDA, game | `nvidia` |

Hybrid là chế độ linh hoạt nhất — không cần reboot để chạy ứng dụng trên NVIDIA (dùng `prime-run`).

## Tổng kết

- EnvyControl cài từ AUR — build source mỗi lần
- Ba chế độ: intel, nvidia, hybrid
- Một lệnh duy nhất `envycontrol -s <mode>` + reboot
- Hybrid mode khuyến nghị cho sử dụng hàng ngày
- Recovery: boot với `nomodeset`, chuyển về hybrid
