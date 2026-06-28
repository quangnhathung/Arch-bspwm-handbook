# Customization — Tùy chỉnh bspwm

Ngày: 28/06/2026

## Các thành phần có thể tùy chỉnh

| Thành phần | File cấu hình | Công cụ |
|------------|---------------|---------|
| Window manager | `~/.config/bspwm/bspwmrc` | `bspc config` |
| Keybinding | `~/.config/sxhkd/sxhkdrc` | sxhkd |
| Bar | `~/.config/polybar/config.ini` | Polybar (Dynamic Island) |
| Launcher | `~/.config/rofi/config.rasi` | Rofi (glassmorphism) |
| Compositor | `~/.config/picom/picom.conf` | Picom (picom-ftlabs-git) |
| Wallpaper | `~/.local/bin/wallpaper.sh` | feh (custom script) |
| Terminal | `~/.config/alacritty/alacritty.toml` | Alacritty |
| Notification | `~/.config/dunst/dunstrc` | Dunst |
| Lock screen | — | `betterlockscreen` (AUR) |
| GTK Theme | `~/.config/gtk-3.0/settings.ini` | GTK (Arc-Dark + Papirus) |
| Polybar modules | `~/.config/polybar/modules/*.ini` | 7 module files |
| Polybar scripts | `~/.config/polybar/scripts/gpu.sh`, `mic.sh` | custom |
| Brightness script | `~/.local/bin/brightness.sh` | custom (NVIDIA backlight) |
| Volume script | `~/.local/bin/volume.sh` | custom (wpctl + dunst notify) |
| Wallpaper script | `~/.local/bin/wallpaper.sh` | custom (prev/next/random) |
| Power menu script | `~/.local/bin/powermenu.sh` | custom (Rofi grid) |

---

## bspwm config (cấu hình thực tế)

Các thiết lập trong `bspwmrc`:

### Window gap

```bash
bspc config window_gap 3   # Khoảng cách hẹp 3px (tiết kiệm diện tích)
```

### Border

```bash
bspc config border_width 1                        # Viền mỏng 1px

# Màu viền Catppuccin Mocha
bspc config focused_border_color   "#89B4FA"      # Blue (cửa sổ focus)
bspc config normal_border_color    "#45475A"      # Surface1 (không focus)
bspc config active_border_color    "#F38BA8"      # Red (urgent)
```

### Padding = 0 (Polybar Dynamic Island floating)

```bash
# Tất cả = 0 vì Polybar dùng override-redirect, không chiếm không gian
bspc config top_padding    0
bspc config bottom_padding 0
bspc config left_padding   0
bspc config right_padding  0
```

### Mouse behavior

```bash
bspc config focus_follows_pointer true    # Chuột chỉ vào đâu, focus vào đó
bspc config pointer_modifier     mod4     # Super + chuột trái = kéo cửa sổ float
```

### Split ratio

```bash
bspc config split_ratio 0.65   # Node mới 35%, node cũ 65%
bspc config automatic_scheme alternate
```

---

## Polybar (Dynamic Island)

File: `~/.config/polybar/config.ini`

### Cấu trúc thư mục

```
~/.config/polybar/
├── config.ini              # Bar config (floating, 78% width, radius 20)
├── launch.sh               # Kill cũ → launch → xdotool raise → ẩn
├── peek.sh                 # Hiện bar trong 8 giây (Super + Shift + b)
├── modules/
│   ├── bspwm.ini           # Dot pager (● ẩn empty workspace)
│   ├── audio.ini           # PulseAudio volume + ramp icon
│   ├── battery.ini         # Pin BAT1 + sạc
│   ├── date.ini            # HH:MM | dd/mm
│   ├── user.ini            # whoami
│   ├── power.ini           # Icon  → gọi powermenu
│   ├── wifi.ini            # SSID + ramp signal
│   └── hw_mic.ini          # CPU + GPU script + mic-status script
└── scripts/
    ├── gpu.sh              # Intel iGPU MHz + NVIDIA %
    └── mic.sh              # Phát hiện app đang ghi âm
```

### Tính năng chính

- **Floating hoàn toàn:** `override-redirect = true` → bar không chiếm không gian
- **Peek mechanism:** Ẩn mặc định → gọi peek.sh → hiện 8 giây → tự ẩn
- **Dot pager:** Workspace hiển thị dạng chấm tròn, empty workspace ẩn
- **GPU monitoring:** Script tự dò Intel iGPU + NVIDIA qua nvidia-smi
- **Mic detection:** Ẩn khi không có app dùng mic, đỏ khi có

### Module bspwm (dot pager)

```ini
[module/bspwm]
type = internal/bspwm
ws-icon-default = ●
label-focused = ●
label-focused-foreground = #fa89b4
label-occupied = ●
label-occupied-foreground = #e1e6f5
label-urgent = ●
label-urgent-foreground = #F38BA8
label-empty =                    # Ẩn workspace trống
```

### Multi-monitor

Xem bài `05-multi-monitor.md`.

---

## Rofi

File: `~/.config/rofi/config.rasi`

### Cấu hình thực tế

```css
configuration {
    modi: "drun";
    show-icons: true;
    icon-theme: "Papirus";
    matching: "fuzzy";
    sort: true;
    sorting-method: "fzf";
    hover-select: true;
    kb-cancel: "Escape,Super+space";
}

* {
    bg-overlay:   #11111b96;   /* Lớp kính mờ */
    bg-input:     #1e1e2eB3;   /* Input trong suốt */
    fg-main:      #cdd6f4;     /* Catppuccin text */
    border-color: #bad2fa;     /* Viền xanh */
}

window {
    width: 750px;
    border-radius: 12px;
    background-image: url("/home/quangnhathung/images/Theme/rofi.jpeg", width);
}

inputbar {
    border-radius: 100px;      /* Hình viên thuốc */
    padding: 12px 20px;
    border: 2px solid #4b5563;
}
```

### Power menu

File: `~/.config/rofi/powermenu.rasi` — layout grid 5 cột:

```css
window {
    width: 650px;
    location: center;
    background-color: #1e1e2e;
    border: 2px solid #89b4fa;
    border-radius: 12px;
}

listview {
    columns: 5;
    lines: 1;
    spacing: 15px;
}

element selected.normal {
    background-color: #89b4fa;
    text-color: #11111b;
}
```

Script: `~/.local/bin/powermenu.sh` (gọi từ Polybar module power, click-left):

```bash
#!/bin/bash
OPTIONS="Shutdown\nReboot\nLock\nLogout"
CHOICE=$(echo -e "$OPTIONS" | rofi -dmenu -p "Power Menu" \
    -theme ~/.config/rofi/powermenu.rasi)

case "$CHOICE" in
    Shutdown) systemctl poweroff ;;
    Reboot)   systemctl reboot ;;
    Lock)     betterlockscreen -l ;;
    Logout)   bspc quit ;;
esac
```

---

## Picom (picom-ftlabs-git — spring physics)

File: `~/.config/picom/picom.conf`

### Fork đang dùng

Cấu hình thực tế dùng **`picom-ftlabs-git`** (AUR) — fork có **spring physics animation**:

```bash
yay -S picom-ftlabs-git
```

Khác biệt với `picom-ibhagwan-git`: dùng hệ thống lò xo vật lý (stiffness, dampening,
mass) thay vì easing thông thường, tạo hiệu ứng nảy tự nhiên.

### Cấu hình thực tế

```bash
# Backend
backend = "glx";
vsync = true;
unredir-if-possible = false;   # Quan trọng cho NVIDIA

# Shadow (macOS style)
shadow = true;
shadow-radius = 28;
shadow-opacity = 0.18;
shadow-offset-y = 8;
shadow-exclude = [ "class_g = 'Rofi'", "class_g = 'Dunst'" ];

# Opacity per-app (focused/unfocused)
opacity-rule = [
    "80:class_g = 'Alacritty' && !focused",
    "100:class_g = 'Alacritty' && focused"
];

# Spring physics animation
animations = true;
animation-stiffness = 300.0;
animation-dampening = 22.0;
animation-mass = 1.0;
animation-for-open-window = "zoom";
animation-for-unmap-window = "zoom";
animation-for-workspace-switch-in = "zoom";

# Rounded corner
corner-radius = 13;
round-borders = 1;             # Ép viền bspwm theo góc bo
```

---

## Theme tổng thể — Color scheme

### Các scheme phổ biến

| Scheme | Đặc điểm | Hex gốc |
|--------|----------|---------|
| **Catppuccin Mocha** | Pastel, ấm, phổ biến nhất | `#1E1E2E`, `#CDD6F4`, `#89B4FA` |
| **Nord** | Xanh lam lạnh, tối giản | `#2E3440`, `#D8DEE9`, `#81A1C1` |
| **Dracula** | Tím đậm, tương phản cao | `#282A36`, `#F8F8F2`, `#BD93F9` |

### Cài đặt theme cho từng thành phần (config thực tế)

| Thành phần | Config hiện tại |
|------------|----------------|
| GTK | `Arc-Dark` + `Papirus` icons |
| Alacritty | Catppuccin Mocha (tự định nghĩa màu trong `alacritty.toml`) |
| Polybar | Catppuccin Mocha (màu thủ công trong `config.ini` + modules) |
| Rofi | Catppuccin Mocha (glassmorphism, ảnh nền, pill input) |
| Picom | Catppuccin Mocha (shadow màu đen mặc định) |
| Dunst | Catppuccin North (`#2E3440`, `#81A1C1`, `#5E81AC`) |
| Tmux | Catppuccin Mocha (plugin `catppuccin/tmux`, flavour mocha) |

**Quan trọng:** Chọn một scheme và **dùng đồng bộ** cho tất cả thành phần.
Catppuccin Mocha là theme chính, đồng bộ qua: bspwm borders, polybar, rofi,
alacritty, dunst, tmux.

---

## Lock screen

### i3lock-color (đơn giản, có sẵn trên AUR)

```bash
yay -S i3lock-color
```

Sử dụng:

```bash
i3lock-color --color=1E1E2E --inside-color=1E1E2E --ring-color=89B4FA \
    --keyhl-color=A6E3A1 --bshl-color=F38BA8 --line-color=00000000 \
    --separator-color=00000000 --verif-color=CDD6F4 --wrong-color=F38BA8 \
    --layout-color=CDD6F4 --time-color=CDD6F4 --date-color=CDD6F4
```

### betterlockscreen (AUR, nhiều tính năng hơn)

```bash
yay -S betterlockscreen
```

Hỗ trợ: chỉnh ảnh nền lock screen, blur, clock, notifications.

```bash
# Set ảnh nền
betterlockscreen -u ~/Pictures/wallpapers/lock.jpg

# Lock
betterlockscreen -l

# Lock với blur
betterlockscreen -l --blur 0.5
```

Phím tắt:

```
super + Shift + Escape
    betterlockscreen -l
```

### So sánh

| Tool | Gói | Độ phức tạp | Tính năng |
|------|-----|-------------|-----------|
| `i3lock-color` | AUR | Thấp | Đơn giản, nhanh, tùy chỉnh màu |
| `betterlockscreen` | AUR | Trung bình | Ảnh nền, blur, clock, media |

---

## Screenshot (Flameshot AppImage)

Dùng Flameshot **AppImage** (không cài package, portable):

```bash
# Tải từ flameshot.org
# Đặt tại ~/flameshot-13.3.AppImage
chmod +x ~/flameshot-13.3.AppImage
```

Keybinding trong sxhkdrc:

```
# Chụp vùng chọn + tự động lưu + copy clipboard
Print
	~/flameshot-13.3.AppImage gui --accept-on-select -p ~/images/Screenshots -c

# Mở Flameshot GUI để chú thích trước khi lưu
super + Print
	~/flameshot-13.3.AppImage gui
```

Tự động chạy cùng bspwm: `~/flameshot-13.3.AppImage &` trong bspwmrc (chạy ngầm
để giảm thời gian khởi động khi nhấn Print).

---

## Backup cấu hình

### Dùng Git

```bash
cd ~/.config
git init
git add bspwm sxhkd polybar rofi picom alacritty
git commit -m "Initial bspwm config"
```

### Backup tar

```bash
tar -czf bspwm-config-$(date +%Y%m%d).tar.gz \
    ~/.config/{bspwm,sxhkd,polybar,rofi,picom,alacritty}
```

---

## Autostart (cấu hình thực tế)

```bash
#!/usr/bin/env bash
# ~/.config/bspwm/bspwmrc

# Touchpad tap-to-click (auto-detect ID)
for id in $(xinput list | grep -i "touchpad" | grep -o 'id=[0-9]*' | cut -d= -f2); do
    xinput set-prop "$id" "libinput Tapping Enabled" 1
done

# Hotkey daemon
pgrep -x sxhkd > /dev/null || sxhkd &

# Cursor fix (tránh icon X đen)
xsetroot -cursor_name left_ptr &

# Policy kit
/usr/lib/polkit-gnome/polkit-gnome-authentication-agent-1 &

# Wallpaper (custom script có prev/next/random)
~/.local/bin/wallpaper.sh &

# Compositor
picom --config ~/.config/picom/picom.conf &

# Polybar Dynamic Island
~/.config/polybar/launch.sh &

# Notification
pgrep -x dunst > /dev/null || dunst &

# Flameshot AppImage (preload)
~/flameshot-13.3.AppImage &
```

> **Lưu ý:** Không dùng `nitrogen`, `nm-applet`, `blueman-applet`, `xfce4-power-manager`.
> Các chức năng được thay thế bởi: `wallpaper.sh` (wallpaper), Polybar modules (network, battery),
> `volume.sh` + Dunst (audio feedback).

---

## Power Management — TLP vs power-profiles-daemon

⚠️ **Xung đột:** Cả hai cùng quản lý CPU scaling governor. **Không chạy cùng lúc.**

| Giải pháp | Khi nào dùng |
|-----------|-------------|
| **TLP** | Cần kiểm soát sâu: disk, USB, PCIe, pin, ngưỡng sạc |
| **power-profiles-daemon** | Chỉ cần chuyển profile đơn giản, có Fn+Q support |

```bash
# Nếu dùng TLP
sudo systemctl disable --now power-profiles-daemon

# Nếu dùng power-profiles-daemon
sudo systemctl disable --now tlp
```

Xem chi tiết tại bài `00-keyboard-system.md` (mục Power Management).

---

## Best practices

1. **Chọn một color scheme và dùng đồng bộ** cho tất cả component.
2. **Backup trước khi thay đổi lớn** — dùng Git để quản lý.
3. **Thay đổi từ từ** — mỗi lần chỉnh một thứ, test trước khi chuyển sang thứ khác.
4. **Giữ bspwmrc gọn gàng** — tách autostart thành file riêng nếu cần.
5. **Đọc tài liệu chính thức** của từng component để hiểu sâu.
