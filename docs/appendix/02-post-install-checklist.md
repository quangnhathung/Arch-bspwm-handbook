# Post-Install Checklist — Lenovo LOQ 15IAX9

**Ngày: 25/06/2026 — Kernel 7.x**

Danh sách kiểm tra sau khi cài Arch Linux. Làm theo thứ tự.

---

### 1. Kết nối mạng

```bash
ping -c 3 archlinux.org
ip addr show
```

Nếu chưa có Wi-Fi (card Realtek RTL8852BE):

```bash
# Cài driver từ USB / LAN tạm thời
sudo pacman -S --needed git base-devel dkms
git clone https://aur.archlinux.org/rtl8852be-dkms.git
cd rtl8852be-dkms
makepkg -si
sudo modprobe 8852be
nmcli device wifi list
nmcli device wifi connect "SSID" password "password"
```

### 2. Cập nhật hệ thống

```bash
sudo pacman -Syu
```

### 3. Cài yay (AUR helper)

`yay` biên dịch từ mã nguồn (chậm hơn). Nếu muốn nhẹ hơn, dùng `yay-bin` (pre-compiled).

```bash
# Bản build từ nguồn (chậm):
sudo pacman -S --needed git base-devel
git clone https://aur.archlinux.org/yay.git
cd yay && makepkg -si

# HOẶC bản pre-compiled (nhanh hơn):
# git clone https://aur.archlinux.org/yay-bin.git
# cd yay-bin && makepkg -si
```

### 4. Cài Xorg + bspwm stack

```bash
sudo pacman -S xorg xorg-server xorg-xinit xorg-xrandr
sudo pacman -S bspwm sxhkd picom polybar rofi alacritty nitrogen
```

### 5. Cấu hình desktop

```bash
mkdir -p ~/.config/{bspwm,sxhkd,picom,polybar,rofi,alacritty}
# Copy config từ repo hoặc tự viết
chmod +x ~/.config/bspwm/bspwmrc
```

Tạo `~/.xinitrc`:

```bash
echo "exec bspwm" > ~/.xinitrc
```

### 6. Kiểm tra desktop

```bash
startx
```

Thành công → bspwm hiện ra với Polybar.

### 7. Cài driver đồ hoạ

RTX 4050 dùng **nvidia-open** (không phải `nvidia`).

```bash
# Intel (Arc tích hợp)
sudo pacman -S mesa lib32-mesa vulkan-intel lib32-vulkan-intel intel-media-driver

# NVIDIA (dùng nvidia-open)
sudo pacman -S nvidia-open nvidia-utils nvidia-settings opencl-nvidia
```

Kiểm tra:

```bash
glxinfo | grep "OpenGL renderer"
nvidia-smi
```

### 8. Cài PipeWire (âm thanh)

```bash
sudo pacman -S pipewire pipewire-alsa pipewire-pulse wireplumber pavucontrol
systemctl --user enable --now pipewire pipewire-pulse wireplumber
```

Kiểm tra:

```bash
speaker-test -l1 -c2
pactl info
```

### 9. Cài Bluetooth

```bash
sudo pacman -S bluez bluez-utils blueman
sudo systemctl enable --now bluetooth
```

Kiểm tra:

```bash
bluetoothctl show
```

### 10. Cài font

```bash
sudo pacman -S ttf-firacode-nerd noto-fonts noto-fonts-emoji ttf-font-awesome
```

### 11. Cài Timeshift + snapshot đầu tiên

```bash
sudo pacman -S timeshift
sudo timeshift --first-run
sudo timeshift --create --comments "sau-cai-dat-$(date +%Y%m%d)"
```

### 12. Cài thêm tiện ích

```bash
# paccache (đã cài qua pacman-contrib nếu dùng install script)
# Nếu chưa có:
sudo pacman -S pacman-contrib

# Công cụ hữu ích
sudo pacman -S htop btop nvtop lm_sensors ntfs-3g exfat-utils
```

### 13. Đổi mật khẩu mặc định

```bash
passwd            # root
passwd archuser   # user
```

### 14. Kiểm tra GRUB

```bash
cat /proc/cmdline
# Phải có: nvidia-drm.modeset=1
efibootmgr -v
```

---

## Bảng kiểm tra tổng hợp

| Hạng mục | Lệnh kiểm tra | Kết quả mong đợi |
|---|---|---|
| Mạng | `ip addr` | `wlan0` hoặc `eth0` có IP |
| Wi-Fi | `nmcli device status` | `wlan0` connected |
| Âm thanh | `speaker-test -l1 -c2` | Có tiếng |
| GPU Intel | `glxinfo \| grep renderer` | Mesa Intel |
| GPU NVIDIA | `nvidia-smi` | Thấy RTX 4050 |
| Bluetooth | `bluetoothctl show` | Controller available |
| BTRFS | `sudo btrfs filesystem usage /` | Filesystem OK |
| Timeshift | `sudo timeshift --list` | Có snapshot |
| GRUB | `efibootmgr -v` | GRUB boot entry |
| bspwm | `pgrep -x bspwm` | bspwm running |
| PipeWire | `pactl info` | Server Name: PulseAudio (on PipeWire) |

## Nếu gặp vấn đề

- **Wi-Fi**: Kiểm tra `rfkill list`, `sudo modprobe 8852be`
- **Desktop**: `cat ~/.xinitrc`, kiểm tra `chmod +x bspwmrc`
- **Âm thanh**: `pactl info`, `systemctl --user status pipewire`
- **NVIDIA**: `dmesg | grep nvidia`, kiểm tra `nvidia-open` đã cài
- **Snapshot**: `sudo timeshift --list`, kiểm tra subvolume @
