# NVIDIA Issues

## Symptoms

- Màn hình đen hoặc treo sau khi boot với NVIDIA.
- `nvidia-smi` không thấy GPU.
- Screen tearing.
- Hiệu năng đồ họa kém.
- Nhiệt độ cao / quạt chạy liên tục (do NVIDIA không tắt).

## Cause

1. **Kernel module không được load**.
2. **Kernel parameter `nvidia-drm.modeset` thiếu hoặc sai**.
3. **Xung đột giữa Intel và NVIDIA**.
4. **NVIDIA không tắt khi dùng Intel mode**.
5. **NVIDIA driver không tương thích với kernel mới**.

## Diagnosis

```bash
# Kiểm tra module
lsmod | grep nvidia

# Kiểm tra kernel parameter
cat /proc/cmdline

# Kiểm tra Xorg log
cat /var/log/Xorg.0.log | grep -i nvidia

# Kiểm tra GPU
lspci | grep -i nvidia

# Kiểm tra nhiệt độ
nvidia-smi --query-gpu=temperature.gpu --format=csv,noheader
```

## Fix

### Fix 1: Kernel module không load

```bash
# Load thủ công
sudo modprobe nvidia
sudo modprobe nvidia_modeset
sudo modprobe nvidia_uvm
sudo modprobe nvidia_drm

# Thêm vào mkinitcpio
vim /etc/mkinitcpio.conf
# MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm)
mkinitcpio -P
```

### Fix 2: Kernel parameter

```bash
vim /etc/default/grub
# GRUB_CMDLINE_LINUX_DEFAULT="... nvidia-drm.modeset=1 ..."
grub-mkconfig -o /boot/grub/grub.cfg
```

### Fix 3: Screen tearing

```bash
# Tạo /etc/X11/xorg.conf.d/10-nvidia.conf
Section "OutputClass"
    Identifier "nvidia"
    MatchDriver "nvidia-drm"
    Driver "nvidia"
    Option "AllowEmptyInitialConfiguration"
    Option "PrimaryGPU" "yes"
    Option "SLI" "Off"
    Option "MetaModes" "nvidia-auto-select +0+0 {ForceFullCompositionPipeline=On}"
EndSection

# Picom
vim ~/.config/picom/picom.conf
# unredir-if-possible = false
```

### Fix 4: NVIDIA không tắt ở Intel mode

```bash
# Kiểm tra
nvidia-smi
# Nếu có process → NVIDIA đang hoạt động

# Dùng envycontrol để tắt
sudo envycontrol -s intel
sudo reboot

# Hoặc tắt thủ công
sudo tee /proc/acpi/bbswitch <(echo OFF)
```

### Fix 5: Driver không tương thích

```bash
# Cài nvidia-dkms (tự rebuild với kernel mới)
sudo pacman -S nvidia-dkms

# Hoặc downgrade driver
sudo pacman -U /var/cache/pacman/pkg/nvidia-xxx.pkg.tar.zst
```

## Prevention

1. **Luôn dùng `nvidia-drm.modeset=1`**.
2. **Cấu hình ForceFullCompositionPipeline** để chống tearing.
3. **Dùng nvidia-dkms** nếu kernel update thường xuyên.
4. **Kiểm tra nvidia-smi** sau mỗi update.
5. **Snapshot trước update NVIDIA driver**.
