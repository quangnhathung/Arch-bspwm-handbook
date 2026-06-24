# Modern bspwm Setup — "r/unixporn Style"

Ngày cập nhật: 25/06/2026

## Mục tiêu

Tổng hợp bộ công cụ phổ biến nhất trên r/unixporn để biến bspwm từ một
WM trần trụi thành desktop "modern" với: rounded corner, blur, status bar,
notification, launcher đẹp, terminal trong suốt, và phím tắt thuận tiện.

Bài này **không đi sâu vào config từng tool** (đã có các bài riêng) mà tập
trung vào: **tổng quan, cách chúng kết nối với nhau, và các tool còn thiếu**
trong handbook.

---

## 1. Tổng quan kiến trúc

```
Người dùng
    │
    ▼
sxhkd (phím tắt) ──► bspwm (cửa sổ)
    │                      │
    ▼                      ▼
rofi (launcher)      picom (hiệu ứng: blur, round, shadow)
alacritty (term)     polybar (thanh trạng thái)
dunst (thông báo)    feh/nitrogen (wallpaper)
flameshot (chụp ảnh)
    │
    ▼
Hệ thống nền (daemon, autostart trong bspwmrc):
    nm-applet   → Wi-Fi tray
    blueman-applet → Bluetooth tray
    pavucontrol   → Âm thanh
    polkit-gnome  → Xác thực quyền
    xfce4-power-manager → Pin / brightness
```

**Nguyên tắc:** bspwm không làm gì cả ngoài quản lý cửa sổ. Mọi thứ khác
đều là chương trình riêng, chạy độc lập, được gọi từ `bspwmrc` (autostart)
hoặc từ `sxhkdrc` (phím tắt).

---

## 2. Từng nhóm công cụ

### 2.1 Window Manager core — bspwm + sxhkd

| Tool | Vai trò | Đã có bài? |
|---|---|---|
| bspwm | Quản lý cửa sổ, chia cây nhị phân | ✅ 02-bspwm.md |
| sxhkd | Daemon phím tắt, không có thì không làm gì được | ✅ 03-sxhkd.md |

**Mẹo r/unixporn:**
- `window_gap` từ 4–12px để có khoảng cách đẹp giữa các cửa sổ
- `border_width` = 2, màu viền hợp với theme
- Dùng `bspc rule` để `state=floating` cho Rofi, Polybar, Dunst, Nitrogen

### 2.2 Compositor — picom

| Tool | Vai trò | Đã có bài? |
|---|---|---|
| picom | Shadow, blur, rounded corner, vsync, fade | ✅ 06-picom.md |

**Có 2 phiên bản:**
- `picom` (official pacman) — không blur, không animation
- `picom-ibhagwan-git` (AUR) — có blur + animation ⭐ r/unixporn

**Config "ăn tiền" cho r/unixporn:**

```ini
# Bắt buộc cho modern look:
shadow = true;
shadow-radius = 12;
shadow-opacity = 0.4;

# Blur (chỉ với ibhagwan fork):
blur-method = "kawase";
blur-size = 12;
blur-background = true;

# Rounded corner (phát hiện tự động):
detect-rounded-corners = true;

# Opacity cho terminal:
opacity-rule = [
    "90:class_g = 'Alacritty' && focused",
    "80:class_g = 'Alacritty' && !focused"
];
```

### 2.3 Status bar — Polybar

| Tool | Vai trò | Đã có bài? |
|---|---|---|
| polybar | Thanh workspace, CPU, RAM, network, pin, giờ | ✅ 04-polybar.md |

**Module phổ biến trên r/unixporn:**
| Module | Hiển thị |
|---|---|
| `bspwm` | Workspace có highlight |
| `pulseaudio` | Volume, click chuột phải → pavucontrol |
| `network` | Tên Wi-Fi + IP |
| `battery` | % pin (quan trọng với laptop) |
| `date` | Giờ:phút |
| `cpu` / `memory` | Tài nguyên hệ thống |
| `powermenu` | Menu tắt máy bằng Rofi |

**Font cần có:** `ttf-nerd-fonts-symbols` để hiển thị icon.

### 2.4 App launcher — Rofi

| Tool | Vai trò | Đã có bài? |
|---|---|---|
| rofi | Launcher, window switcher, power menu, calculator | ✅ 05-rofi.md |

**Modes phổ biến:**
- `rofi -show drun` — tìm app (Super + d)
- `rofi -show run` — chạy lệnh (Super + Shift + d)
- `rofi -show window` — chuyển cửa sổ (Super + Shift + r)
- `rofi-power-menu` (AUR) — shutdown/reboot/logout

### 2.5 Wallpaper — feh / Nitrogen

| Tool | Vai trò | Đã có bài? |
|---|---|---|
| nitrogen | GUI chọn wallpaper | ✅ 07-nitrogen.md |
| feh | CLI, cực nhẹ, set wallpaper | ❌ (cần bổ sung) |

**Cài feh:**
```bash
pacman -S feh
```

**Dùng feh trong bspwmrc** (thay nitrogen):
```bash
feh --bg-fill ~/Pictures/wallpapers/current.jpg &
```

**Mẹo:** Dùng `feh --randomize --bg-fill ~/Pictures/wallpapers/*` để random
wallpaper mỗi lần boot.

### 2.6 Notification — Dunst

| Tool | Vai trò | Đã có bài? |
|---|---|---|
| dunst | Notification daemon (volume, wifi, update, alert) | ❌ **CHƯA CÓ** |

Dunst là notification system phổ biến nhất cho WM. Nó nhẹ, theme được,
và tự động hoạt động với hầu hết ứng dụng.

**Cài đặt:**
```bash
pacman -S dunst
```

**Cấu hình** `~/.config/dunst/dunstrc`:

```ini
[global]
    monitor = 0
    width = 350
    height = 100
    offset = 16x56
    origin = top-right
    notification_limit = 5
    progress_bar = true
    indicate_hidden = yes
    transparency = 10
    corner_radius = 8
    gap_size = 4
    separator_height = 2
    padding = 12
    horizontal_padding = 12
    frame_width = 2
    frame_color = "#44475A"
    font = "JetBrains Mono 11"
    line_height = 4
    markup = full
    format = "<b>%s</b>\n%b"
    sort = yes
    show_age_threshold = 60
    ignore_newline = false
    stack_duplicates = true
    hide_duplicate_count = false
    show_indicators = no
    icon_position = left
    max_icon_size = 32

[urgency_low]
    background = "#282A36"
    foreground = "#F8F8F2"
    timeout = 5

[urgency_normal]
    background = "#282A36"
    foreground = "#F8F8F2"
    timeout = 8

[urgency_critical]
    background = "#FF5555"
    foreground = "#FFFFFF"
    timeout = 0
```

**Autostart trong bspwmrc:**
```bash
dunst &
```

**Test thử:**
```bash
notify-send "Hello" "Dunst is working!"
notify-send -u critical "Urgent" "This is critical"
```

**Tích hợp volume:** Khi nhấn phím volume, pulseaudio tự động gửi notification
qua dunst (nếu có `libnotify`). Cài:
```bash
pacman -S libnotify
```

**Tích hợp screenshot (flameshot):** flameshot tự động gửi notification
khi chụp xong.

### 2.7 File manager — Thunar / PCManFM

| Tool | Vai trò | Đã có bài? |
|---|---|---|
| pcmanfm | File manager GTK cực nhẹ | ❌ (chỉ mention) |
| thunar | File manager nhẹ, phổ biến, có bulk rename | ❌ (chỉ mention) |

**Cài:**
```bash
# Thunar (phổ biến hơn)
pacman -S thunar thunar-volman thunar-archive-plugin tumbler

# PCManFM (cực nhẹ)
pacman -S pcmanfm
```

**Tích hợp:**
- `thunar` + `tumbler` → xem thumbnail ảnh/video
- `thunar-archive-plugin` → click phải → nén/giải nén
- `thunar-volman` → tự động mount USB

**Phím tắt trong sxhkdrc:**
```bash
super + e
    thunar
# hoặc
super + e
    pcmanfm
```

### 2.8 Terminal emulator — Alacritty / Kitty

| Tool | Vai trò | Đã có bài? |
|---|---|---|
| alacritty | GPU-accelerated terminal, siêu nhanh | ❌ (chỉ mention trong rules) |
| kitty | GPU terminal, nhiều feature (tabs, split, image) | ❌ **CHƯA CÓ** |

**Cài Alacritty:**
```bash
pacman -S alacritty
```

**Config** `~/.config/alacritty/alacritty.toml` (theme Dracula):

```toml
[window]
opacity = 0.92
padding = { x = 8, y = 8 }

[font]
size = 11

[font.normal]
family = "JetBrains Mono"
style = "Regular"

[font.bold]
family = "JetBrains Mono"
style = "Bold"

[font.italic]
family = "JetBrains Mono"
style = "Italic"

[colors.primary]
background = "#282A36"
foreground = "#F8F8F2"

[colors.normal]
black   = "#21222C"
red     = "#FF5555"
green   = "#50FA7B"
yellow  = "#FFB86C"
blue    = "#BD93F9"
magenta = "#FF79C6"
cyan    = "#8BE9FD"
white   = "#F8F8F2"

[colors.bright]
black   = "#6272A4"
red     = "#FF6E6E"
green   = "#69FF94"
yellow  = "#FFCA80"
blue    = "#CAA9FA"
magenta = "#FF92D0"
cyan    = "#A4FFFF"
white   = "#FFFFFF"
```

**Opacity + blur combo:** Alacritty opacity 0.92 + picom blur → background
mờ ảo đặc trưng của r/unixporn.

**Cài Kitty (thay thế, nhiều feature hơn):**
```bash
pacman -S kitty
```

Kitty có sẵn: tabs, split pane, remote file editing, image rendering
(kitty icat), và SSH kitty protocol.

### 2.9 Dock — Tint2 (tuỳ chọn)

| Tool | Vai trò | Đã có bài? |
|---|---|---|
| tint2 | Panel/dock kiểu taskbar | ❌ **CHƯA CÓ** |

Trên r/unixporn, phần lớn người dùng **không dùng dock** vì bspwm đã có
workspace switching qua phím tắt (Super + 1-9). Tuy nhiên tint2 vẫn được
dùng như một panel phụ.

```bash
pacman -S tint2
```

### 2.10 System tray tools

| Tool | Vai trò | Đã có bài? |
|---|---|---|
| network-manager-applet (nm-applet) | Tray icon Wi-Fi | ❌ (chỉ mention trong bspwmrc) |
| pavucontrol | GUI volume control | ❌ (chỉ mention) |
| blueman | Bluetooth tray + manager | ✅ 05-drivers/06-bluetooth.md |

**Cài:**
```bash
pacman -S network-manager-applet pavucontrol blueman
```

**Autostart trong bspwmrc:**
```bash
nm-applet &
blueman-applet &
```

**pavucontrol** không chạy nền — nó được mở khi cần (click volume trên
polybar hoặc phím tắt Super + v).

### 2.11 Authentication — polkit-gnome

| Tool | Vai trò | Đã có bài? |
|---|---|---|
| polkit-gnome | Xác thực quyền cho GUI (mount USB, cài app) | ❌ (chỉ mention) |

**Cài:**
```bash
pacman -S polkit-gnome
```

**Autostart trong bspwmrc:**
```bash
/usr/lib/polkit-gnome/polkit-gnome-authentication-agent-1 &
```

Nếu không có polkit, các ứng dụng GUI như GParted, mount USB, hoặc
cài gói từ pamac sẽ không xin được quyền root.

### 2.12 Screenshot — Flameshot

| Tool | Vai trò | Đã có bài? |
|---|---|---|
| flameshot | Chụp ảnh màn hình có GUI chú thích | ❌ (chỉ mention trong essential-tools) |

**Cài:**
```bash
pacman -S flameshot
```

**Phím tắt trong sxhkdrc:**
```bash
# Flameshot GUI (chụp + chú thích ngay)
Print
    flameshot gui

# Flameshot fullscreen (chụp thẳng)
super + Print
    flameshot full -p ~/Pictures/screenshots/
```

**Flameshot vs các tool khác:**
| Tool | Đặc điểm |
|---|---|
| flameshot ⭐ | GUI chú thích (mũi tên, text, highlight, blur) |
| maim + slop | CLI, nhẹ, không GUI |
| scrot | Basic, không có tính năng gì thêm |

### 2.13 Font & Theme

| Tool | Vai trò | Đã có bài? |
|---|---|---|
| Fonts | JetBrains Mono, Fira Code, Noto, Nerd Font | ✅ 08-fonts.md |
| Themes | GTK, icon, cursor | ✅ 09-themes.md |

**Bổ sung — Catppuccin theme ⭐ (rất phổ biến trên r/unixporn):**

Hiện tại handbook dùng Nordic + Dracula. Ngoài ra, **Catppuccin** là
theme phổ biến nhất trên r/unixporn năm 2025–2026.

```bash
# GTK theme (từ AUR)
yay -S catppuccin-gtk-theme-mocha

# Icon theme (từ AUR)
yay -S catppuccin-icon-theme

# Cursor theme (từ AUR)
yay -S catppuccin-cursors-mocha
```

Cấu hình trong `~/.config/gtk-3.0/settings.ini`:
```ini
[Settings]
gtk-theme-name=Catppuccin-Mocha-Standard-Blue-Dark
gtk-icon-theme-name=Catppuccin-Icon-Theme
gtk-cursor-theme-name=Catppuccin-Mocha-Dark-Cursors
```

Catppuccin có 4 biến thể: Latte (sáng), Frappé, Macchiato, Mocha (tối).
Mocha là phổ biến nhất cho r/uniporn.

---

## 3. Bộ setup "combo thực tế" — cài tất cả trong 1 lệnh

```bash
# === Core ===
pacman -S bspwm sxhkd

# === Desktop ===
pacman -S picom polybar rofi dunst alacritty feh

# === File manager ===
pacman -S thunar thunar-volman thunar-archive-plugin tumbler

# === Screenshot ===
pacman -S flameshot

# === System tray + tools ===
pacman -S network-manager-applet pavucontrol blueman \
          xfce4-power-manager polkit-gnome libnotify

# === Fonts ===
pacman -S ttf-jetbrains-mono noto-fonts noto-fonts-cjk \
          noto-fonts-emoji ttf-dejavu ttf-liberation \
          ttf-nerd-fonts-symbols

# === Theme ===
pacman -S nordic-theme papirus-icon-theme bibata-cursor-theme

# === Optional AUR (cần yay) ===
yay -S picom-ibhagwan-git rofi-power-menu catppuccin-gtk-theme-mocha
```

---

## 4. Bspwmrc — autostart đầy đủ

File `~/.config/bspwm/bspwmrc` hoàn chỉnh cho modern setup:

```bash
#!/bin/bash

# ---- Monitor ----
xrandr --output eDP-1 --mode 1920x1080 --rate 144

# ---- Workspaces ----
bspc monitor -d I II III IV V VI VII VIII IX

# ---- Config ----
bspc config border_width         2
bspc config window_gap           8
bspc config split_ratio          0.50
bspc config borderless_monocle   true
bspc config gapless_monocle      true
bspc config focus_follows_pointer false
bspc config pointer_follows_focus false

# ---- Rules ----
bspc rule -a Alacritty              state=tiled
bspc rule -a firefox                desktop='^2' state=tiled
bspc rule -a Gimp                   desktop='^8' state=tiled
bspc rule -a nitrogen:*             state=floating
bspc rule -a Rofi:*                 state=floating
bspc rule -a Polybar:*              state=floating
bspc rule -a Dunst:*                state=floating
bspc rule -a "Viewnior:*"           state=floating
bspc rule -a "Pcmanfm:*"            state=floating
bspc rule -a "Thunar:*"             state=floating
bspc rule -a "Xfce4-power-manager:*" state=floating
bspc rule -a "Flameshot:*"          state=floating
bspc rule -a "Pavucontrol:*"        state=floating

# ---- Compositor ----
picom --config ~/.config/picom/picom.conf &

# ---- Bar ----
polybar main &

# ---- Wallpaper ----
feh --bg-fill ~/Pictures/wallpapers/current.jpg &

# ---- Notification ----
dunst &

# ---- System tray ----
nm-applet &
blueman-applet &

# ---- Power management ----
xfce4-power-manager &

# ---- Polkit ----
/usr/lib/polkit-gnome/polkit-gnome-authentication-agent-1 &
```

---

## 5. Tổng kết visual — "r/unixporn checklist"

| Yếu tố | Công cụ | Config gợi ý |
|---|---|---|
| Window gap | bspwm | `window_gap = 8` |
| Rounded corner | picom | `detect-rounded-corners = true` |
| Shadow | picom | `shadow-radius = 12`, `shadow-opacity = 0.4` |
| Blur background | picom-ibhagwan-git | `blur-method = "kawase"` |
| Opacity terminal | alacritty + picom | `opacity = 0.92` + `opacity-rule` |
| Status bar | polybar | Module: bspwm, date, pulseaudio, network, battery |
| Notification | dunst | Góc trên phải, corner radius, transparent |
| Launcher | rofi | Theme Nord/Catppuccin, icon Papirus |
| Terminal font | JetBrains Mono | Size 11 |
| Color scheme | Catppuccin Mocha / Dracula / Nord | Đồng bộ tất cả app |
| Wallpaper | feh | `--bg-fill` |
