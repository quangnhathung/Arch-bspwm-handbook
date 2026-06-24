# NVIDIA Issues

*Áp dụng cho Lenovo LOQ 15IAX9 — RTX 30/40 series — nvidia-open — Kernel 7.x — 25/06/2026*

## Triệu chứng (Symptoms)

- Màn hình đen hoặc treo ngay sau boot khi NVIDIA được kích hoạt.
- `nvidia-smi` không thấy GPU — hiện "NVIDIA-SMI has failed because it couldn't communicate".
- Screen tearing trong trình duyệt, video, game.
- Nhiệt độ cao / quạt chạy liên tục dù ở chế độ Intel (NVIDIA không tắt).
- Hiệu năng đồ họa kém, thấp hơn mong đợi.

## Nguyên nhân (Causes)

1. **Module NVIDIA chưa được load** — `nvidia_open`, `nvidia_modeset`, `nvidia_uvm`, `nvidia_drm`.
2. **Thiếu kernel parameter `nvidia-drm.modeset=1`** — cần để kích hoạt DRM (modesetting).
3. **Xung đột giữa Intel (i915) và NVIDIA (nvidia-open)** — sai thứ tự module trong mkinitcpio.
4. **NVIDIA không tắt ở Intel mode** — `nvidia-smi` vẫn show GPU active, hao pin.
5. **Cấu hình Xorg thiếu ForceFullCompositionPipeline** — gây screen tearing.
6. **Dùng `nvidia` thay vì `nvidia-open`** — kernel 7.x khuyên dùng `nvidia-open-dkms` cho RTX 30/40.

## Chẩn đoán (Diagnosis)

```bash
# 1. Module NVIDIA
lsmod | grep nvidia

# 2. Kernel parameters
cat /proc/cmdline

# 3. Xorg log
cat /var/log/Xorg.0.log | grep -iE "nvidia|fail|error"

# 4. nvidia-smi
nvidia-smi

# 5. GPU nhiệt độ
nvidia-smi --query-gpu=temperature.gpu --format=csv,noheader

# 6. Driver version
nvidia-smi | grep "Driver Version"
```

## Khắc phục (Fix)

### Fix 1: Load kernel modules

```bash
# Load thủ công (test)
sudo modprobe nvidia_open
sudo modprobe nvidia_modeset
sudo modprobe nvidia_uvm
sudo modprobe nvidia_drm

# Nếu OK → thêm vào mkinitcpio
echo 'MODULES=(nvidia_open nvidia_modeset nvidia_uvm nvidia_drm)' | sudo tee /etc/mkinitcpio.conf.d/nvidia.conf
sudo mkinitcpio -P
```

> **Lưu ý:** dùng `nvidia_open` (cho `nvidia-open-dkms`), không phải `nvidia`.

### Fix 2: Kernel parameter

```bash
# Sửa /etc/default/grub
sudo sed -i 's/GRUB_CMDLINE_LINUX_DEFAULT="[^"]*/& nvidia-drm.modeset=1/' /etc/default/grub

# Hoặc mở thủ công
sudo vim /etc/default/grub
# Thêm: nvidia-drm.modeset=1

sudo grub-mkconfig -o /boot/grub/grub.cfg
```

### Fix 3: Screen tearing — ForceFullCompositionPipeline

Tạo `/etc/X11/xorg.conf.d/10-nvidia.conf`:

```conf
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

Nếu dùng picom (fork ibhagwan/picom), tắt unredir:

```bash
vim ~/.config/picom/picom.conf
# unredir-if-possible = false
```

> **picom animation/blur chỉ có trên fork ibhagwan/picom**, không phải picom gốc.

### Fix 4: NVIDIA không tắt ở Intel mode

```bash
# Kiểm tra GPU đang hoạt động
nvidia-smi
# Nếu có process → NVIDIA đang chạy

# Chuyển sang Intel bằng envycontrol
sudo envycontrol -s intel
sudo reboot

# Kiểm tra lại
nvidia-smi  # "No devices were found" → NVIDIA đã tắt

# Nếu vẫn không tắt → kiểm tra bbswitch
sudo tee /proc/acpi/bbswitch <<< OFF
```

### Fix 5: Cài nvidia-open-dkms (khuyên dùng cho RTX 30/40)

```bash
# Xóa driver cũ
sudo pacman -Rnsc nvidia nvidia-utils nvidia-dkms

# Cài nvidia-open-dkms
sudo pacman -S nvidia-open-dkms nvidia-utils nvidia-settings

# Sửa module name trong mkinitcpio
sed -i 's/nvidia /nvidia_open /g' /etc/mkinitcpio.conf.d/nvidia.conf
sudo mkinitcpio -P
```

## Phòng ngừa (Prevention)

1. **Luôn dùng `nvidia-open-dkms` thay vì `nvidia` trên RTX 30/40 series** — tương thích kernel 7.x.
2. **Luôn set `nvidia-drm.modeset=1` trong GRUB.**
3. **Cấu hình ForceFullCompositionPipeline ngay khi cài NVIDIA.**
4. **Snapshot BTRFS trước mỗi lần cập nhật NVIDIA driver.**
5. **Kiểm tra `nvidia-smi` sau mỗi lần update kernel.**
