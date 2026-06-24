# Rofi — Launcher

Ngày cập nhật: 25/06/2026

## Mục tiêu

Cài đặt và cấu hình Rofi — application launcher, window switcher, và nhiều
chức năng khác.

## Kiến thức nền

### Rofi là gì?

Rofi là một launcher cho X11/Wayland. Ban đầu là bản fork của dmenu,
Rofi có thể:

- Tìm kiếm và chạy ứng dụng (drun mode).
- Chạy lệnh tùy ý (run mode).
- Chuyển đổi giữa các cửa sổ (window mode).
- Thay thế cho các menu truyền thống.

Rofi hoạt động như một popup overlay, không phải cửa sổ thông thường.
Phím tắt `Super + d` (cấu hình trong sxhkdrc) để mở.

## Các bước thực hiện

### Bước 1: Cài Rofi

```bash
pacman -S rofi
```

### Bước 2: Cài icon theme (nếu muốn hiển thị icon)

```bash
pacman -S papirus-icon-theme
```

### Bước 3: Tạo thư mục config

```bash
su - archuser
mkdir -p ~/.config/rofi
exit
```

### Bước 4: Tạo file cấu hình

Rofi có hai loại config: Xresources (cũ) và config.rasi (mới, dùng chung).

```bash
vim /home/archuser/.config/rofi/config.rasi
```

Nội dung:

```
configuration {
    modi: "drun,run,window,power-menu";
    icon-theme: "Papirus";
    show-icons: true;
    terminal: "alacritty";
    drun-display-format: "{name}";
    font: "JetBrains Mono 12";
    location: 0;
    x-offset: 0;
    y-offset: 0;
    width: 40;
    lines: 12;
    columns: 1;
    matching: "fuzzy";
    sort: true;
    sorting-method: "normal";
    case-sensitive: false;
    cycle: true;
    sidebar-mode: false;
    kb-mode-next: "Alt+Tab";
    kb-mode-previous: "Alt+Shift+Tab";
    kb-row-up: "Up,Control+p";
    kb-row-down: "Down,Control+n";
    kb-accept-entry: "Return,KP_Enter";
    kb-cancel: "Escape,Control+c";
}

@theme "/usr/share/rofi/themes/nord.rasi"
```

### Bước 5: Power menu (tùy chọn)

`rofi-power-menu` là script cho phép Rofi hiển thị menu tắt/mở máy.
Nó **không có sẵn** trong gói `rofi` — cần cài riêng.

Trên Arch, `rofi-power-menu` có trong AUR:

```bash
# Nếu dùng yay (AUR helper)
yay -S rofi-power-menu

# Hoặc cài thủ công từ GitHub
# https://github.com/jluttine/rofi-power-menu
```

Nếu không muốn dùng AUR, bạn có thể tự viết script power menu đơn giản:

```bash
#!/bin/bash
OPTIONS="Lock\nLogout\nReboot\nShutdown"
CHOICE=$(echo -e "$OPTIONS" | rofi -dmenu -p "Power Menu")

case "$CHOICE" in
    Lock) betterlockscreen -l ;;
    Logout) bspc quit ;;
    Reboot) systemctl reboot ;;
    Shutdown) systemctl poweroff ;;
esac
```

Cấu hình trong sxhkdrc:

```
super + shift + x
    rofi -show power-menu -modi power-menu:rofi-power-menu
```

### Bước 6: Rofi Calc (tùy chọn)

Máy tính trong Rofi:

```bash
pacman -S rofi-calc
```

Sau đó thêm `calc` vào `modi` trong config:

```
modi: "drun,run,window,power-menu,calc";
```

## Các mode của Rofi

| Mode | Chức năng | Phím tắt (trong sxhkdrc) |
|---|---|---|
| `drun` | Tìm kiếm và chạy ứng dụng (desktop files) | `Super + d` |
| `run` | Chạy lệnh tùy ý | `Super + Shift + d` |
| `window` | Chuyển đổi cửa sổ | `Super + Shift + r` |
| `power-menu` | Tắt máy, reboot, logout | `Super + Shift + x` |

## Custom theme

Tạo file `~/.config/rofi/nord-custom.rasi`:

```
* {
    background:     #282A36;
    background-alt: #44475A;
    foreground:     #F8F8F2;
    selected:       #44475A;
    active:         #50FA7B;
    urgent:         #FF5552;
}

window {
    location: center;
    width: 600;
    border: 2px;
    border-color: #44475A;
}

inputbar {
    background: @background-alt;
    text-color: @foreground;
    padding: 4px;
}

listview {
    lines: 12;
    columns: 1;
}

element {
    padding: 4px;
    text-color: @foreground;
}

element selected {
    background: @selected;
    text-color: @foreground;
}

element-icon {
    size: 1em;
}

element-text {
    vertical-align: 0.5;
    horizontal-align: 0.5;
}
```

Sau đó sửa `config.rasi` để dùng theme này:

```
@theme "~/.config/rofi/nord-custom.rasi"
```

Hoặc dùng đường dẫn tuyệt đối.

## Tùy chỉnh

### Kích thước

```ini
width: 40;    # % màn hình
lines: 12;    # số dòng hiển thị
```

### Fuzzy matching

```ini
matching: "fuzzy";   # Tìm kiếm mờ (tolerant với lỗi chính tả)
```

### Icon theme

```ini
icon-theme: "Papirus";
show-icons: true;
```

## Troubleshooting

### Rofi không hiển thị icon

```bash
# Kiểm tra icon theme
rofi -show drun -icon-theme Papirus
```

### Rofi không tìm thấy ứng dụng

- Cache desktop files: `sudo update-desktop-database`.
- Kiểm tra thư mục: `ls /usr/share/applications/`.

### Rofi bị tile trên bspwm

Trong bspwmrc đã có rule:

```bash
bspc rule -a Rofi:* state=floating
```

Nếu chưa, thêm vào.

## Tổng kết

- Rofi đã được cài với cấu hình cơ bản.
- Hỗ trợ tìm kiếm ứng dụng, chạy lệnh, chuyển cửa sổ, power menu.
- Giao diện Nord theme, hỗ trợ icon và fuzzy matching.
- `rofi-power-menu` cần cài riêng từ AUR.
