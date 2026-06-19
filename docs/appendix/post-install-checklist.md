# Post-Install Checklist

## Mục tiêu

Danh sách kiểm tra sau khi cài đặt để đảm bảo hệ thống hoạt động đúng.

## Sau khi reboot

### 1. Kiểm tra kết nối mạng

```bash
ping -c 3 archlinux.org
ip addr show
```

Nếu chưa có Wi-Fi → cắm LAN hoặc USB tethering.
Sau đó cài driver Wi-Fi Realtek:

```bash
yay -S rtl8852be-dkms
sudo modprobe 8852be
nmcli device wifi list
nmcli device wifi connect "SSID" password "password"
```

### 2. Cập nhật hệ thống

```bash
sudo pacman -Syu
```

### 3. Cài yay (AUR helper)

```bash
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

### 4. Cài Xorg và bspwm

```bash
sudo pacman -S xorg xorg-server xorg-init xorg-xrandr
sudo pacman -S bspwm sxhkd picom polybar rofi alacritty nitrogen
```

### 5. Cấu hình desktop

- Copy file cấu hình từ docs/04-desktop/.
- `chmod +x ~/.config/bspwm/bspwmrc`.
- Tạo `~/.xinitrc` với nội dung `exec bspwm`.

### 6. Kiểm tra desktop

```bash
startx
```

Nếu thành công → thấy bspwm với Polybar.

### 7. Cài driver đồ họa

```bash
# Intel
sudo pacman -S mesa lib32-mesa vulkan-intel lib32-vulkan-intel intel-media-driver

# NVIDIA
sudo pacman -S nvidia nvidia-utils nvidia-settings
```

### 8. Kiểm tra GPU

```bash
glxinfo | grep "OpenGL renderer"
nvidia-smi
```

### 9. Cài PipeWire (audio)

```bash
sudo pacman -S pipewire pipewire-alsa pipewire-pulse wireplumber pavucontrol
systemctl --user enable --now pipewire pipewire-pulse wireplumber
```

### 10. Cài Bluetooth

```bash
sudo pacman -S bluez bluez-utils blueman
sudo systemctl enable --now bluetooth
```

### 11. Cài font

```bash
sudo pacman -S ttf-firacode-nerd noto-fonts noto-fonts-emoji ttf-font-awesome
```

### 12. Cài Timeshift

```bash
sudo pacman -S timeshift
sudo timeshift --first-run
```

### 13. Tạo snapshot đầu tiên

```bash
sudo timeshift --create --comments "sau-cai-dat"
```

### 14. Kiểm tra và cấu hình GRUB

```bash
cat /proc/cmdline
# Phải có nvidia-drm.modeset=1
```

### 15. Đổi password mặc định

```bash
passwd            # root
passwd archuser   # user
```

## Các kiểm tra cuối

| Check | Lệnh | Kết quả mong đợi |
|---|---|---|
| Network | `ip addr` | wlan0 hoặc eth0 có IP |
| Wi-Fi | `nmcli device status` | wlan0 connected |
| Audio | `speaker-test -l1 -c2` | Có tiếng |
| GPU Intel | `glxinfo \| grep renderer` | Mesa Intel |
| GPU NVIDIA | `nvidia-smi` | Thấy GPU |
| Bluetooth | `bluetoothctl show` | Controller available |
| BTRFS | `btrfs filesystem usage /` | Filesystem OK |
| Timeshift | `timeshift --list` | Có snapshot |
| GRUB | `efibootmgr -v` | GRUB boot entry |
| bspwm | `pgrep -x bspwm` | bspwm running |

## Chưa hoàn thành?

Quay lại tài liệu tương ứng:

- **Network**: docs/05-drivers/wifi.md
- **Desktop**: docs/04-desktop/
- **Audio**: docs/05-drivers/audio.md
- **NVIDIA**: docs/05-drivers/nvidia.md
- **Snapshot**: docs/07-btrfs/
