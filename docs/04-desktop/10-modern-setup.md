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

## Cấu hình thực tế trên máy

Bộ desktop hiện tại dùng **Catppuccin Mocha** + **Dynamic Island Polybar** +
**picom-ftlabs-git** (spring animations) + **Alacritty** (opacity 0.85) +
**Dunst** (Catppuccin North notifications) + **Rofi** (glassmorphism với ảnh nền).

Xem từng bài chi tiết:
- `02-bspwm.md` — bspwmrc với touchpad fix, focus_follows_pointer, Catppuccin borders
- `04-polybar.md` — Dynamic Island floating bar, peek.sh, GPU/mic scripts
- `05-rofi.md` — Rofi glassmorphism, pill search bar, powermenu grid
- `06-picom.md` — picom-ftlabs-git với spring physics animation, corner radius 13px

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

**Config thực tế (picom-ftlabs-git — spring physics):**

```ini
# Shadow kiểu macOS (lan rộng, nhẹ)
shadow = true;
shadow-radius = 28;
shadow-opacity = 0.18;
shadow-offset-y = 8;

# Spring physics animation (zoom + nảy)
animations = true;
animation-stiffness = 300.0;
animation-dampening = 22.0;
animation-for-open-window = "zoom";

# Rounded corner 13px
corner-radius = 13;
round-borders = 1;

# Opacity per-app
opacity-rule = [
    "80:class_g = 'Alacritty' && !focused",
    "100:class_g = 'Alacritty' && focused"
];
```

### 2.3 Status bar — Polybar (Dynamic Island)

| Tool | Vai trò | Đã có bài? |
|---|---|---|
| polybar | Thanh workspace, CPU, GPU, network, pin, giờ | ✅ 04-polybar.md |

**Config thực tế:** Floating bar kiểu Dynamic Island (macOS):
- `override-redirect = true` — bar nổi hoàn toàn, không chiếm không gian bspwm
- `width = 78%`, `offset-x = 11%` — căn giữa, bo góc 20px
- Ẩn mặc định, hiện khi nhấn `Super + Shift + b` (peek.sh)
- Tự động ẩn sau 8 giây

**Modules trên bar thực tế:**
| Vị trí | Module | Hiển thị |
|--------|--------|----------|
| Trái | `power` | Icon  → gọi powermenu |
| Trái | `user` | Tên user (`whoami`) |
| Trái | `wifi` | SSID + vạch sóng |
| Trái | `cpu` | CPU usage |
| Trái | `mic-status` | Mic đỏ khi có app đang ghi âm |
| Giữa | `bspwm` | Dot pager (●) |
| Phải | `gpu` | Intel MHz + NVIDIA % |
| Phải | `pulseaudio` | Volume + icon thay đổi theo mức |
| Phải | `battery` | % pin + icon sạc |
| Phải | `date` | Đồng hồ HH:MM |

**Script tùy chỉnh:**
- `scripts/gpu.sh` — Đọc Intel iGPU frequency + NVIDIA utilization
- `scripts/mic.sh` — Phát hiện app đang ghi âm qua `pactl`

**Font cần có:** `ttf-jetbrains-mono` + `ttf-nerd-fonts-symbols`.

### 2.4 App launcher — Rofi

| Tool | Vai trò | Đã có bài? |
|---|---|---|
| rofi | Launcher, window switcher, power menu | ✅ 05-rofi.md |

**Config thực tế:**
- **Nền ảnh:** `rofi.jpeg` với lớp glassmorphism (`#11111b96`)
- **Pill input bar:** `border-radius: 100px` — hình viên thuốc
- **FZF sorting:** `sorting-method: fzf`
- **Matching:** fuzzy
- **Theme:** Catppuccin Mocha (đồng bộ toàn hệ thống)

**Modes:**
- `rofi -show drun` — tìm app (`Super + space`)
- `rofi -show window` — chuyển cửa sổ (`alt + Tab`)
- `powermenu.rasi` — layout grid 5 cột (gọi từ Polybar click)

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

**Cấu hình thực tế** `~/.config/dunst/dunstrc` (Catppuccin North + custom rules):

```ini
[global]
    origin = top-right
    offset = (20, 20)
    width = 320
    height = (0, 350)
    corner_radius = 12
    frame_width = 2
    gap_size = 8
    padding = 18
    horizontal_padding = 18
    text_icon_padding = 20
    font = "JetBrains Mono Nerd Font 10, Noto Sans 10"
    markup = full
    format = "<b>%s</b>\n%b"
    notification_limit = 5
    idle_threshold = 120
    show_age_threshold = 60
    word_wrap = yes
    icon_theme = "Papirus-Dark,Papirus,Adwaita"
    icon_position = left
    min_icon_size = 64
    max_icon_size = 80
    progress_bar = true
    progress_bar_height = 8
    progress_bar_corner_radius = 4

[urgency_low]
    background = "#2E3440"
    foreground = "#ECEFF4"
    frame_color = "#81A1C1"
    timeout = 4

[urgency_normal]
    background = "#2E3440"
    foreground = "#D8DEE9"
    frame_color = "#5E81AC"
    timeout = 6

[urgency_critical]
    background = "#2E3440"
    foreground = "#BF616A"
    frame_color = "#BF616A"
    timeout = 0

# Custom rules
[dev_tools]
    appname = "code"
    summary = "*Flutter*|*Spring Boot*|*Golang*|*Python*|*Java*"
    frame_color = "#A3BE8C"    # Xanh lá cho dev tools
    timeout = 8

[gaming_mode]
    appname = "steam_app_*"
    fullscreen = pushback      # Ẩn noti khi chơi game
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

**Config thực tế** `~/.config/alacritty/alacritty.toml` (Catppuccin Mocha):

```toml
[colors.primary]
background = "#121212"
foreground = "#cdd6f4"

[colors.cursor]
text = "#1e1e2e"
cursor = "#f5e0dc"

[colors.normal]
black   = "#45475a"
red     = "#f38ba8"
green   = "#a6e3a1"
yellow  = "#f9e2af"
blue    = "#89b4fa"
magenta = "#f5c2e7"
cyan    = "#94e2d5"
white   = "#bac2de"

[colors.bright]
black   = "#585b70"
red     = "#f38ba8"
green   = "#a6e3a1"
yellow  = "#f9e2af"
blue    = "#89b4fa"
magenta = "#f5c2e7"
cyan    = "#94e2d5"
white   = "#a6adc8"

[window]
opacity = 0.85
padding = { x = 6, y = 0 }
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

## 4. Bspwmrc — autostart đầy đủ (cấu hình thực tế)

```bash
#!/usr/bin/env bash

# ---- Touchpad fix ----
for id in $(xinput list | grep -i "touchpad" | grep -o 'id=[0-9]*' | cut -d= -f2); do
    xinput set-prop "$id" "libinput Tapping Enabled" 1
done

# ---- Hotkey daemon ----
pgrep -x sxhkd > /dev/null || sxhkd &

# ---- Cursor fix ----
xsetroot -cursor_name left_ptr &

# ---- Polkit ----
/usr/lib/polkit-gnome/polkit-gnome-authentication-agent-1 &

# ---- Wallpaper (custom script) ----
~/.local/bin/wallpaper.sh &

# ---- Compositor (picom-ftlabs-git) ----
picom --config ~/.config/picom/picom.conf &

# ---- Polybar Dynamic Island ----
~/.config/polybar/launch.sh &

# ---- Notification ----
pgrep -x dunst > /dev/null || dunst &

# ---- Screenshot ----
~/flameshot-13.3.AppImage &

# ---- Workspaces ----
bspc monitor -d I II III IV V VI VII VIII IX

# ---- Padding = 0 (Polybar floating) ----
bspc config top_padding    0
bspc config bottom_padding 0
bspc config left_padding   0
bspc config right_padding  0

# ---- Appearance (Catppuccin) ----
bspc config border_width         1
bspc config window_gap           3
bspc config focused_border_color "#89B4FA"
bspc config normal_border_color  "#45475A"
bspc config active_border_color  "#F38BA8"

# ---- Behavior ----
bspc config split_ratio          0.65
bspc config borderless_monocle   true
bspc config gapless_monocle      true
bspc config focus_follows_pointer true
bspc config pointer_modifier     mod4

# ---- Rules ----
bspc rule -a Pavucontrol state=floating center=true
bspc rule -a Flameshot state=floating
bspc rule -a flameshot state=floating
```

---

## 5. Tổng kết visual — "r/unixporn checklist" (config thực tế)

| Yếu tố | Công cụ | Config thực tế |
|---|---|---|
| Window gap | bspwm | `window_gap = 3` (hẹp) |
| Rounded corner | picom | `corner-radius = 13`, `round-borders = 1` |
| Shadow | picom | `shadow-radius = 28`, `shadow-opacity = 0.18` (macOS style) |
| Animation | picom-ftlabs-git | Spring physics: stiffness 300, dampening 22 |
| Opacity terminal | alacritty + picom | `opacity = 0.85` + `inactive-opacity = 0.9` |
| Status bar | polybar | Dynamic Island: floating, ẩn/hiện, peek 8s |
| Notification | dunst | Catppuccin North, góc trên phải, corner radius 12px |
| Launcher | rofi | Glassmorphism + ảnh nền + pill input bar |
| Terminal font | JetBrains Mono Nerd Font | Size 11 |
| Color scheme | Catppuccin Mocha | Đồng bộ tất cả app |
| Wallpaper | feh | `~/.local/bin/wallpaper.sh` (prev/next/random) |
| GPU monitoring | Polybar script | Intel iGPU MHz + NVIDIA % |
| Mic status | Polybar script | Tự động ẩn/hiện khi có app ghi âm |
