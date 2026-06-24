# Customization — Tùy chỉnh bspwm

Ngày: 25/06/2026

## Các thành phần có thể tùy chỉnh

| Thành phần | File cấu hình | Công cụ |
|------------|---------------|---------|
| Window manager | `~/.config/bspwm/bspwmrc` | `bspc config` |
| Keybinding | `~/.config/sxhkd/sxhkdrc` | sxhkd |
| Bar | `~/.config/polybar/config.ini` | Polybar |
| Launcher | `~/.config/rofi/config.rasi` | Rofi |
| Compositor | `~/.config/picom/picom.conf` | Picom |
| Wallpaper | `~/.config/nitrogen/bg-saved.cfg` | Nitrogen / feh |
| Terminal | `~/.config/alacritty/alacritty.toml` | Alacritty |
| Lock screen | — | `i3lock-color` / `betterlockscreen` (AUR) |
| GTK Theme | `~/.config/gtk-3.0/settings.ini` | GTK |

---

## bspwm config

Các thiết lập trong `bspwmrc`:

### Window gap

```bash
# Khoảng cách giữa các cửa sổ (px)
bspc config window_gap 8      # mặc định
bspc config window_gap 16     # thoáng hơn
bspc config window_gap 0      # tiết kiệm diện tích
```

### Border

```bash
# Độ dày viền
bspc config border_width 2

# Màu viền (Catppuccin Mocha)
bspc config focused_border_color   "#cba6f7"   # mauve
bspc config normal_border_color    "#45475a"   # surface1
bspc config presel_feedback_color  "#cba6f7"
```

### Padding

```bash
# Khoảng cách từ cạnh màn hình đến cửa sổ
bspc config top_padding    28    # = height của Polybar
bspc config bottom_padding 0
bspc config left_padding   0
bspc config right_padding  0
```

### Mouse behavior

```bash
# Focus khi di chuột qua cửa sổ
bspc config focus_follows_pointer true

# Con trỏ chuột nhảy theo focus
bspc config pointer_follows_focus false
```

### Split ratio

```bash
bspc config split_ratio 0.50
```

---

## Polybar

File: `~/.config/polybar/config.ini`

### Màu sắc (Catppuccin Mocha)

```ini
[colors]
background = #1E1E2E
background-alt = #313244
foreground = #CDD6F4
primary = #89B4FA
secondary = #A6E3A1
alert = #F38BA8
```

### Module bspwm

```ini
[module/bspwm]
type = internal/bspwm
label-focused = %name%
label-focused-foreground = #${colors.primary}
label-focused-background = #${colors.background-alt}
label-occupied = %name%
label-occupied-foreground = #${colors.foreground}
label-urgent = %name%
label-urgent-foreground = #${colors.alert}
label-empty = %name%
label-empty-foreground = #585B70
```

### Multi-monitor

Xem bài `05-multi-monitor.md`.

---

## Rofi

File: `~/.config/rofi/config.rasi`

```css
* {
    background-color: #1E1E2E;
    text-color: #CDD6F4;
}

window {
    width: 40%;
    border-color: #89B4FA;
    border: 2px;
}

listview {
    lines: 12;
}

element selected {
    background-color: #45475A;
    text-color: #89B4FA;
}

entry {
    background-color: #313244;
    text-color: #CDD6F4;
}
```

### Power menu script

Tạo `~/.config/rofi/powermenu.sh`:

```bash
#!/bin/bash
options="Shutdown\nReboot\nLock\nLogout"
selected=$(echo -e "$options" | rofi -dmenu -p "Power")

case "$selected" in
    Shutdown) systemctl poweroff ;;
    Reboot)   systemctl reboot ;;
    Lock)     betterlockscreen -l ;;
    Logout)   bspc quit ;;
esac
```

Phím tắt:

```
super + Shift + x
    ~/.config/rofi/powermenu.sh
```

---

## Picom

File: `~/.config/picom/picom.conf`

Picom là compositor cho X11, cung cấp shadow, opacity, fading, blur, animation.

### ⚠️ Lưu ý quan trọng về Picom

- **Official picom** (gói `picom` trong official repos) chỉ hỗ trợ shadow, opacity, fading cơ bản.
- **Tính năng animation và blur** (như `animations = true`, `blur-method = "kawase"`) **CHỈ CÓ** ở fork `ibhagwan/picom` (AUR: `picom-ibhagwan-git` hoặc `picom-git`).

Nếu bạn muốn animation và blur, cài fork:

```bash
# Từ AUR (yay hoặc paru)
yay -S picom-ibhagwan-git
```

### Cấu hình cơ bản

```bash
# Shadow
shadow = true;
shadow-radius = 8;
shadow-opacity = 0.3;
shadow-offset-x = -4;
shadow-offset-y = -4;

# Shadow exclude (không đổ bóng cho bar, dock)
shadow-exclude = [
    "class_g = 'Polybar'",
    "class_g = 'Rofi'",
    "class_g = 'dunst'"
];

# Fading
fading = true;
fade-in-step = 0.03;
fade-out-step = 0.03;

# Opacity
inactive-opacity = 0.95;

# Opacity exclude
opacity-rule = [
    "80:class_g = 'Alacritty' && focused",
    "90:class_g = 'Alacritty' && !focused"
];
```

#### Animation & Blur (chỉ với fork ibhagwan/picom)

```bash
# Blur
blur-method = "kawase";
blur-strength = 5;

# Animation
animations = true;
animation-for-open-window = "zoom";
animation-for-workswitch = "slide";
```

### Backend

```bash
backend = "glx";      # OpenGL (khuyên dùng)
# backend = "xrender"; # fallback nếu glx không hoạt động
# backend = "egl";     # thử nếu glx có vấn đề
```

---

## Theme tổng thể — Color scheme

### Các scheme phổ biến

| Scheme | Đặc điểm | Hex gốc |
|--------|----------|---------|
| **Catppuccin Mocha** | Pastel, ấm, phổ biến nhất | `#1E1E2E`, `#CDD6F4`, `#89B4FA` |
| **Nord** | Xanh lam lạnh, tối giản | `#2E3440`, `#D8DEE9`, `#81A1C1` |
| **Dracula** | Tím đậm, tương phản cao | `#282A36`, `#F8F8F2`, `#BD93F9` |

### Cài đặt theme cho từng thành phần

| Thành phần | Catppuccin | Nord | Dracula |
|------------|-----------|------|---------|
| GTK | `catppuccin-gtk-theme` (AUR) | `nordic-theme` | `dracula-gtk-theme` (AUR) |
| Alacritty | Copy từ catppuccin/alacritty | Copy từ arcticicestudio/nord-alacritty | Copy từ dracula/alacritty |
| Polybar | Tự chỉnh màu trong config.ini | Tự chỉnh | Tự chỉnh |
| Rofi | Dùng catppuccin rofi theme | Dùng nord rofi theme | Dùng dracula rofi theme |
| Picom | Chỉnh màu shadow | Chỉnh màu shadow | Chỉnh màu shadow |

**Quan trọng:** Chọn một scheme và **dùng đồng bộ** cho tất cả thành phần.

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

## Screenshot

### maim (có sẵn trong official repos)

```bash
sudo pacman -S maim
```


Tạo thư mục chứa ảnh chụp:

```bash
mkdir -p ~/Pictures/screenshots
```

Keybinding trong sxhkdrc:

```
# Toàn màn hình
Print
	maim -u ~/Pictures/screenshots/$(date +%Y%m%d-%H%M%S).png

# Chụp vùng chọn (select area)
super + Print
	maim -su ~/Pictures/screenshots/$(date +%Y%m%d-%H%M%S).png

# Chụp cửa sổ hiện tại
super + Ctrl + Print
	maim -i $(xdotool getactivewindow) ~/Pictures/screenshots/$(date +%Y%m%d-%H%M%S).png
```

### flameshot (thay thế, nhiều tính năng edit)

```bash
sudo pacman -S flameshot
```

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

## Autostart

Mọi chương trình tự động chạy đều đặt trong `bspwmrc`:

```bash
#!/bin/bash
# ~/.config/bspwm/bspwmrc

# Daemon
sxhkd &
/usr/local/bin/fnq-handler.sh &

# Bar
polybar main &

# Compositor
picom --config ~/.config/picom/picom.conf &

# Wallpaper
nitrogen --restore &
# hoặc feh --bg-scale ~/Pictures/wallpapers/current.jpg &

# Policy kit
/usr/lib/polkit-gnome/polkit-gnome-authentication-agent-1 &

# Notifications
dunst &

# Network applet (nếu dùng nm-applet)
nm-applet &
```

> **Lưu ý:** Giữ `bspwmrc` gọn gàng. Nếu có quá nhiều autostart, tách ra file riêng (ví dụ `~/.config/bspwm/autostart.sh`) và gọi từ `bspwmrc`.

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
