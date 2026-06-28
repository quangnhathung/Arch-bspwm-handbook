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

### Bước 4: Tạo file cấu hình — config.rasi

```bash
vim /home/archuser/.config/rofi/config.rasi
```

**Cấu hình thực tế:** Nền ảnh `rofi.jpeg` + lớp kính mờ + thanh tìm kiếm hình viên thuốc (pill).

```css
configuration {
    modi: "drun";
    show-icons: true;
    icon-theme: "Papirus";
    display-drun: "";
    drun-display-format: "{name}";
    disable-history: false;
    hover-select: true;
    kb-cancel: "Escape,Super+space";
    drun-match-fields: "name";
    matching: "fuzzy";
    sort: true;
    sorting-method: "fzf";
}

* {
    bg-overlay:   #11111b96;   /* Lớp kính mờ (đen 85% opacity) */
    bg-input:     #1e1e2eB3;   /* Nền input mờ */
    fg-main:      #cdd6f4;     /* Chữ trắng Catppuccin */
    fg-dim:       #a6adc8;     /* Chữ phụ */
    primary:      #4b5563;     /* Màu xanh chủ đạo */
    border-color: #bad2fa;     /* Viền xanh */
    background-color: transparent;
    text-color: @fg-main;
    font: "JetBrainsMono Nerd Font 12";
}

window {
    transparency: "real";
    width: 750px;
    border: 1px;
    border-color: @border-color;
    border-radius: 12px;
    background-image: url("/home/quangnhathung/images/Theme/rofi.jpeg", width);
}

mainbox {
    background-color: @bg-overlay;
    orientation: vertical;
    children: [ inputbar, listview ];
    padding: 25px 25px 60px 25px;
    spacing: 20px;
}

inputbar {
    background-color: @bg-input;
    border: 2px solid;
    border-color: @primary;
    border-radius: 100px;         /* Bo tròn tạo hình viên thuốc */
    padding: 12px 20px;
    margin: 0px 0px 15px 0px;
    children: [ prompt, entry ];
}

prompt {
    text-color: @border-color;
    font: "JetBrainsMono Nerd Font 14";
    margin: 0px 15px 0px 0px;
    vertical-align: 0.5;
}

entry {
    background-color: transparent;
    text-color: @fg-main;
    placeholder: "Search applications...";
    placeholder-color: @fg-dim;
    vertical-align: 0.5;
    blink: true;
}

listview {
    columns: 1;
    lines: 8;
    spacing: 10px;
    cycle: true;
    dynamic: true;
    scrollbar: false;
    layout: vertical;
}

element {
    orientation: horizontal;
    border-radius: 8px;
    padding: 10px 15px;
    cursor: pointer;
}

element-icon {
    size: 32px;
    margin: 0 15px 0 0;
}

element selected.normal {
    background-color: @primary;
    text-color: #11111B;
}
```

### Bước 5: Power menu — powermenu.rasi

```bash
vim /home/archuser/.config/rofi/powermenu.rasi
```

Layout grid 5 cột cho các tùy chọn shutdown/reboot/logout:

```css
configuration {
    show-icons: false;
}

* {
    bg:          #1e1e2e;
    bg-selected: #89b4fa;
    fg:          #cdd6f4;
    fg-selected: #11111b;
    border-col:  #89b4fa;
    background-color: transparent;
    text-color: @fg;
    font: "JetBrainsMono Nerd Font 12";
}

window {
    width: 650px;
    location: center;
    background-color: @bg;
    border: 2px solid;
    border-color: @border-col;
    border-radius: 12px;
    padding: 20px;
}

mainbox {
    children: [ listview ];
}

listview {
    columns: 5;
    lines: 1;
    cycle: true;
    dynamic: true;
    scrollbar: false;
    spacing: 15px;
}

element {
    orientation: vertical;
    padding: 20px 0px;
    border-radius: 10px;
}

element selected.normal {
    background-color: @bg-selected;
    text-color: @fg-selected;
}
```

**Script gọi power menu** (`~/.local/bin/powermenu.sh`):

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

Phím tắt trong sxhkdrc: `click-left = ~/.local/bin/powermenu.sh` (trên module power của Polybar).

### Bước 6: Các mode của Rofi

| Mode | Chức năng | Phím tắt (trong sxhkdrc) |
|---|---|---|
| `drun` | Tìm kiếm và chạy ứng dụng (desktop files) | `Super + space` |
| `window` | Chuyển đổi cửa sổ | `alt + Tab` |
| `power-menu` | Tắt máy, reboot, logout (custom .rasi) | Polybar click |

## Các điểm nổi bật trong config thực tế

| Tính năng | Mô tả |
|---|---|
| **Background image** | Ảnh nền `/home/quangnhathung/images/Theme/rofi.jpeg`, scale theo width |
| **Glassmorphism** | Lớp `bg-overlay` với màu `#11111b96` (đen 85% opacity) tạo hiệu ứng kính mờ |
| **Pill input bar** | `border-radius: 100px` biến thanh tìm kiếm thành hình viên thuốc |
| **FZF sorting** | `sorting-method: fzf` — sắp xếp kết quả kiểu fzf (Fuzzy Finder) |
| **Hover select** | `hover-select: true` — chỉ cần di chuột qua item là chọn |
| **Power menu grid** | `powermenu.rasi` layout 5 cột 1 hàng, grid-style |
| **Icon 32px** | Icon ứng dụng hiển thị 32px, theme Papirus |
| **Catppuccin colors** | Đồng bộ màu `#cdd6f4`, `#89b4fa`, `#1e1e2e` với toàn bộ desktop |

## Tùy chỉnh

### Kích thước

```css
window { width: 750px; }    /* px thay vì % — chính xác hơn */
listview { lines: 8; }      /* số dòng hiển thị */
```

### Ảnh nền

Đổi đường dẫn `background-image` trong `window` block.

## Troubleshooting

### Rofi không hiển thị icon

```bash
rofi -show drun -icon-theme Papirus
```

### Rofi không tìm thấy ứng dụng

```bash
sudo update-desktop-database
ls /usr/share/applications/
```

### Rofi bị tile trên bspwm

Trong bspwmrc đã có rule `state=floating` cho Rofi.

## Tổng kết

- Rofi dùng **drun** mode với custom theme Catppuccin.
- Nền ảnh + glassmorphism + pill input bar.
- Power menu dùng theme riêng (`powermenu.rasi`), gọi từ Polybar.
- Fuzzy matching với fzf sorting.
- Đồng bộ màu sắc với toàn bộ hệ thống.
