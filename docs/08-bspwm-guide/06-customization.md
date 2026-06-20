# Customization — Tùy chỉnh bspwm

## Mục tiêu

Hướng dẫn tùy chỉnh giao diện và hành vi của bspwm.

## Các thành phần có thể tùy chỉnh

| Thành phần | File cấu hình | Công cụ |
|---|---|---|
| Window manager | `~/.config/bspwm/bspwmrc` | bspc config |
| Keybinding | `~/.config/sxhkd/sxhkdrc` | sxhkd |
| Bar | `~/.config/polybar/config.ini` | Polybar |
| Launcher | `~/.config/rofi/config.rasi` | Rofi |
| Compositor | `~/.config/picom/picom.conf` | Picom |
| Wallpaper | `~/.config/nitrogen/bg-saved.cfg` | Nitrogen |
| Theme | `~/.config/gtk-3.0/settings.ini` | GTK |
| Terminal | `~/.config/alacritty/alacritty.yml` | Alacritty |

## Tùy chỉnh bspwm

### Window gap

```bash
# Khoảng cách giữa các cửa sổ (px)
bspc config window_gap 8

# Tăng lên 16 cho không gian thoáng hơn
bspc config window_gap 16

# Giảm xuống 0 cho tiết kiệm diện tích
bspc config window_gap 0
```

### Border

```bash
# Độ dày viền
bspc config border_width 2

# Màu viền
bspc config focused_border_color "#50FA7B"
bspc config normal_border_color "#44475A"
bspc config presel_border_color "#FF5555"
```

### Padding (khoảng cách từ cạnh màn hình)

```bash
# Padding trên-dưới-trái-phải
bspc config top_padding 0
bspc config bottom_padding 0
bspc config left_padding 0
bspc config right_padding 0

# Nếu có Polybar, set top_padding = height của bar
bspc config top_padding 28
```

### Mouse behavior

```bash
# Focus theo chuột
bspc config focus_follows_pointer true

# Chuột nhảy theo focus
bspc config pointer_follows_focus true

# Click chuột phải trên desktop → menu
bspc config pointer_modifier super
```

### Split ratio mặc định

```bash
# 50/50
bspc config split_ratio 0.50

# Nghiêng về cửa sổ hiện tại (lấy 60%)
bspc config split_ratio 0.60
```

## Tùy chỉnh sxhkd

### Thêm keybinding mới

Mở `~/.config/sxhkd/sxhkdrc` và thêm:

```
# Ví dụ: Mở file manager
super + e
    pcmanfm

# Ví dụ: Mở trình duyệt
super + b
    firefox

# Ví dụ: Screenshot vùng chọn
super + Print
    maim -su ~/Pictures/screenshots/$(date +%Y%m%d-%H%M%S).png
```

### Key chording

sxhkd hỗ trợ chuỗi phím (chain):

```
# Nhấn Super + o, sau đó nhấn tiếp một phím
super + o
    ; dm
        {t, f, m}
            bspc desktop -l {tiled, floating, monocle}
    ; wn
        {h, j, k, l}
            bspc node -p {west, south, north, east}
```

## Tùy chỉnh Polybar

### Thay đổi màu sắc

Trong `~/.config/polybar/config.ini`:

```ini
[colors]
background = #1E1E2E
background-alt = #313244
foreground = #CDD6F4
primary = #89B4FA
secondary = #A6E3A1
alert = #F38BA8
```

### Thêm module

```ini
[module/github]
type = custom/script
exec = curl -s "https://api.github.com/notifications?access_token=..." | jq length
interval = 60
```

## Tùy chỉnh Rofi

### Theme Rofi

Rofi theme có thể tùy chỉnh sâu bằng file `.rasi`.

Ví dụ tạo theme cá nhân:

```bash
vim ~/.config/rofi/theme.rasi
```

```css
* {
    background-color: #1E1E2E;
    text-color: #CDD6F4;
}

window {
    width: 40%;
    border-color: #89B4FA;
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

Reference: https://github.com/davatorium/rofi/wiki/Themes

## Tùy chỉnh Picom

### Tăng/shadow

```bash
# Giảm shadow
shadow-radius = 8;
shadow-opacity = 0.3;

# Tắt shadow cho ứng dụng cụ thể
shadow-exclude = [
    "class_g = 'firefox'"
];
```

### Tắt animation (nếu lag)

```bash
animations = false;
fading = false;
```

## Theme tổng thể

### Color scheme: Catppuccin Mocha

Catppuccin là theme màu pastel rất đẹp và có port cho hầu hết ứng dụng.

| Ứng dụng | Cách cài |
|---|---|
| Alacritty | Copy từ https://github.com/catppuccin/alacritty |
| Polybar | Copy màu vào config.ini |
| Rofi | Dùng catppuccin theme |
| Picom | Chỉnh màu shadow |
| GTK | `pacman -S catppuccin-gtk-theme` (AUR) |

## Script cá nhân hóa

### Tạo script lock screen

```bash
pacman -S betterlockscreen
betterlockscreen -u ~/Pictures/wallpapers/lock.jpg
```

Trong sxhkdrc:

```
super + shift + Escape
    betterlockscreen -l
```

### Tạo script screenshot

```bash
mkdir -p ~/Pictures/screenshots
```

Đã có trong sxhkdrc:

```
Print
    maim -u ~/Pictures/screenshots/$(date +%Y%m%d-%H%M%S).png

super + Print
    maim -su ~/Pictures/screenshots/$(date +%Y%m%d-%H%M%S).png
```

## Backup và chia sẻ cấu hình

```bash
# Backup toàn bộ config
tar -czf bspwm-config-$(date +%Y%m%d).tar.gz ~/.config/{bspwm,sxhkd,polybar,rofi,picom,alacritty,nitrogen}

# Git init
cd ~/.config
git init
git add bspwm sxhkd polybar rofi picom alacritty
git commit -m "Initial bspwm config"
```

## Best practices

1. **Sao lưu cấu hình** trước khi thay đổi lớn.
2. **Thay đổi từ từ**: Mỗi lần chỉnh một thứ, test trước khi chuyển sang thứ khác.
3. **Dùng Git** để quản lý phiên bản config.
4. **Đọc tài liệu** của từng component (links trong bài này).
5. **Kết hợp màu sắc đồng bộ** giữa các thành phần.

## Tổng kết

- bspwm, sxhkd, Polybar, Rofi, Picom đều có thể tùy chỉnh sâu.
- Mỗi thành phần có file config riêng.
- Sử dụng color scheme đồng bộ (Catppuccin, Nord, Dracula).
- Quản lý config bằng Git để dễ backup và chia sẻ.
- Script cá nhân hóa: lock screen, screenshot.
