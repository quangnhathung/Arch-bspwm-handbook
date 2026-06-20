# EnvyControl — Quản lý GPU

## Mục tiêu

Cài đặt EnvyControl để dễ dàng chuyển đổi giữa Intel và NVIDIA.

## Kiến thức nền

### EnvyControl là gì?

EnvyControl là công cụ CLI cho phép chuyển đổi chế độ đồ họa trên laptop
NVIDIA Optimus. Nó hỗ trợ:

- **Intel mode**: Chỉ dùng Intel iGPU, NVIDIA tắt hoàn toàn.
- **NVIDIA mode**: Chỉ dùng NVIDIA dGPU, Intel tắt.
- **Hybrid mode**: Cả hai đều hoạt động, NVIDIA render offload.

### Tại sao dùng EnvyControl?

Thay vì cấu hình thủ công Xorg, kernel params, EnvyControl tự động hóa
việc chuyển đổi. Một lệnh duy nhất chuyển chế độ, EnvyControl:

1. Sửa file Xorg config.
2. Cập nhật GRUB.
3. Rebuild initramfs.
4. Reboot (bắt buộc sau khi chuyển).

### Rủi ro

- **Phải reboot sau mỗi lần chuyển** (không thể chuyển nóng).
- Nếu chọn sai chế độ → màn hình đen → cần recovery.
- EnvyControl ghi đè Xorg config → nếu đã cấu hình thủ công, có thể bị mất.

## Các bước thực hiện

### Bước 1: Cài EnvyControl từ AUR

```bash
pacman -S --needed base-devel git
git clone https://aur.archlinux.org/envycontrol.git
cd envycontrol
makepkg -si
```

Hoặc nếu đã có yay:

```bash
yay -S envycontrol
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
```

Sau đó reboot:

```bash
sudo reboot
```

**Kết quả**:
- NVIDIA tắt hoàn toàn → pin tốt nhất.
- Hiệu năng đồ họa 3D kém hơn (dùng Intel).
- Nhiệt độ thấp hơn.

### Bước 4: Chuyển sang NVIDIA mode

```bash
sudo envycontrol -s nvidia
sudo reboot
```

**Kết quả**:
- Intel tắt.
- NVIDIA làm mọi thứ.
- Hiệu năng cao nhất.
- Pin nhanh hết hơn.

### Bước 5: Chuyển sang Hybrid mode (khuyên dùng)

```bash
sudo envycontrol -s hybrid
sudo reboot
```

**Kết quả**:
- Cả hai GPU hoạt động.
- Intel làm display, NVIDIA render offload.
- Cân bằng giữa hiệu năng và pin.

### Bước 6: Kiểm tra sau khi chuyển

```bash
# Kiểm tra GPU active
envycontrol --query

# Kiểm tra OpenGL renderer
glxinfo | grep "OpenGL renderer"

# Kiểm tra NVIDIA có active không
nvidia-smi
```

## Chi tiết từng chế độ

### Intel mode

```
GPU: Chỉ Intel
Pin: ++++ (5/5)
Nhiệt: Thấp
Hiệu năng 3D: Thấp
Dùng khi: Lướt web, xem phim, văn phòng, lúc di chuyển
```

### NVIDIA mode

```
GPU: Chỉ NVIDIA
Pin: + (1/5)
Nhiệt: Cao
Hiệu năng 3D: Cao nhất
Dùng khi: Cần GPU mạnh (render, game — sau này)
```

### Hybrid mode

```
GPU: Intel (display) + NVIDIA (render offload)
Pin: ++ (2/5)
Nhiệt: Trung bình
Hiệu năng 3D: Theo nhu cầu
Dùng khi: Sử dụng hàng ngày, thi thoảng cần GPU
```

## Log của EnvyControl

```bash
cat /var/log/envycontrol.log
```

## Gỡ EnvyControl

Nếu muốn quay lại cấu hình thủ công:

```bash
# Kiểm tra các file đã backup
ls /etc/envycontrol/

# Restore Xorg config gốc
sudo envycontrol --restore-xorg

# Hoặc gỡ hoàn toàn
sudo pacman -R envycontrol
```

## Troubleshooting

### Màn hình đen sau khi chuyển chế độ

**Symptoms**: Sau reboot, màn hình đen, không vào được X.

**Cause**: EnvyControl đã cấu hình sai Xorg hoặc kernel params.

**Fix**:
1. Boot vào single-user mode (thêm `systemd.unit=multi-user.target` vào GRUB).
2. Chuyển về hybrid:

```bash
sudo envycontrol -s hybrid
sudo reboot
```

3. Nếu vẫn đen, gỡ envycontrol và cấu hình thủ công.

### EnvyControl báo lỗi "NVIDIA module not loaded"

```bash
# Load module thủ công
modprobe nvidia
modprobe nvidia_modeset
modprobe nvidia_uvm
modprobe nvidia_drm
```

## Khuyên dùng

Với máy Lenovo LOQ 15IAX9:

- **Mặc định**: Hybrid mode (dùng hàng ngày).
- **Khi di chuyển**: Intel mode (tiết kiệm pin).
- **Khi cần hiệu năng**: NVIDIA mode (render nặng, game).

Tuy nhiên, hybrid mode thường là lựa chọn tốt nhất vì không cần reboot
khi muốn chạy ứng dụng trên NVIDIA (dùng `prime-run`).

## Tổng kết

- EnvyControl đã được cài từ AUR.
- Ba chế độ: intel, nvidia, hybrid.
- Chuyển đổi bằng một lệnh + reboot.
- Hybrid mode khuyên dùng cho sử dụng hàng ngày.
- Có rủi ro màn hình đen nếu chuyển sai chế độ.
