# Bspwm — Window Manager

## Mục tiêu

Cài đặt và cấu hình bspwm (Binary Space Partitioning Window Manager).

## Kiến thức nền

### Window Manager là gì?

Window Manager (WM) là chương trình quản lý vị trí, kích thước, và cách hiển thị
của các cửa sổ ứng dụng trên màn hình.

### bspwm hoạt động như thế nào?

bspwm chia màn hình thành các **node** (ô) bằng các đường phân cách ngang hoặc dọc
(Binary Space Partitioning). Cấu trúc cây (tree) quản lý các node này:

```
Root (Màn hình)
├── Node A (40%)
│   ├── Cửa sổ 1
│   └── Cửa sổ 2
└── Node B (60%)
    └── Cửa sổ 3
```

Khi mở cửa sổ mới, node đang chọn bị chia làm hai.

### bspwm gần như vô dụng nếu không có sxhkd

bspwm CHỈ quản lý cửa sổ. Nó không xử lý bàn phím. Mọi phím tắt (chuyển
workspace, đóng cửa sổ, resize, v.v.) đều do **sxhkd** (Simple X Hotkey Daemon)
xử lý. Không có sxhkd, bạn không thể làm gì với bspwm ngoài nhìn.

## Các bước thực hiện

### Bước 1: Cài bspwm và sxhkd

```bash
pacman -S bspwm sxhkd
```

Luôn cài cả hai cùng lúc.

### Bước 2: Tạo thư mục cấu hình

```bash
su - archuser
mkdir -p ~/.config/bspwm
mkdir -p ~/.config/sxhkd
exit
```

### Bước 3: Tạo file cấu hình bspwmrc

```bash
vim /home/archuser/.config/bspwm/bspwmrc
```

Nội dung:

```bash
#!/bin/bash

# ---- Monitor ----
xrandr --output eDP-1 --mode 1920x1080 --rate 144

# ---- Configuration ----
bspc monitor -d I II III IV V VI VII VIII IX

bspc config border_width         2
bspc config window_gap           8
bspc config split_ratio          0.50
bspc config borderless_monocle   true
bspc config gapless_monocle      true
bspc config focus_follows_pointer false
bspc config pointer_follows_focus false
bspc config pointer_action_modifier super

# ---- Rules ----
bspc rule -a Gimp                   desktop='^8' state=tiled
bspc rule -a firefox                desktop='^2' state=tiled
bspc rule -a Alacritty              state=tiled
bspc rule -a nitrogen:*             state=floating
bspc rule -a Rofi:*                 state=floating
bspc rule -a Polybar:*              state=floating
bspc rule -a "Viewnior:*"           state=floating
bspc rule -a "Pcmanfm:*"            state=floating
bspc rule -a "Xfce4-power-manager:*" state=floating

# ---- Compositor ----
picom --config ~/.config/picom/picom.conf &

# ---- Bar ----
polybar main &

# ---- Launcher daemon ----
# (sxhkd chạy riêng)

# ---- Wallpaper ----
nitrogen --restore &

# ---- System tray ----
nm-applet &
blueman-applet &
volumeicon &

# ---- Power management ----
xfce4-power-manager &

# ---- Polkit ----
/usr/lib/polkit-gnome/polkit-gnome-authentication-agent-1 &
```

Giải thích từng phần:

#### `xrandr`

Đặt độ phân giải 1920x1080 144Hz cho màn hình laptop (`eDP-1`).
Tên output có thể khác (eDP-1-1). Kiểm tra bằng `xrandr` sau khi có NVIDIA driver.

#### `bspc monitor`

Tạo 9 desktop (workspace) có tên I đến IX.

#### `bspc config`

Các cấu hình window manager:
- `border_width 2`: Độ dày viền cửa sổ.
- `window_gap 8`: Khoảng cách giữa các cửa sổ.
- `split_ratio 0.50`: Tỉ lệ chia mặc định (50/50).
- `focus_follows_pointer false`: Focus không tự động theo chuột.
- `pointer_follows_focus false`: Chuột không tự động nhảy theo focus.

#### `bspc rule`

Rules để xử lý cửa sổ cụ thể:
- `firefox` luôn mở ở desktop 2.
- `Rofi` và `Polybar` luôn ở dạng floating.
- `nitrogen` floating để tránh bị tile.

#### Các chương trình nền

- `picom`: Compositor (đổ bóng, chống tearing).
- `polybar`: Thanh trạng thái.
- `nitrogen`: Quản lý wallpaper.
- `nm-applet`: NetworkManager tray icon.
- `blueman-applet`: Bluetooth tray icon.
- `volumeicon`: Âm lượng tray icon.
- `xfce4-power-manager`: Quản lý năng lượng (pin, brightness).
- `polkit-gnome`: Xác thực quyền (cho GUI).

### Bước 4: Phân quyền executable

```bash
chmod +x /home/archuser/.config/bspwm/bspwmrc
```

bspwmrc phải có quyền execute (là một script bash).

### Bước 5: Cập nhật .xinitrc

```bash
echo "exec bspwm" > /home/archuser/.xinitrc
```

### Bước 6: Kiểm tra cấu hình (sau reboot)

```bash
# Khởi động X + bspwm
startx
```

## Các lệnh bspwm cơ bản (qua sxhkd)

Sẽ được cấu hình chi tiết trong bài sxhkd. Một số lệnh qua terminal:

```bash
# Đóng cửa sổ hiện tại
bspc node -c

# Chuyển desktop
bspc desktop -f next

# Chia dọc
bspc node -p east
```

## Troubleshooting

### bspwm không khởi động

Kiểm tra:
- `~/.config/bspwm/bspwmrc` có execute permission không?
- `~/.config/bspwm/bspwmrc` không có lỗi syntax?
- sxhkd đã được cài chưa?

## Tổng kết

- bspwm đã được cài và cấu hình cơ bản.
- bspwmrc đã được tạo với các setting cho laptop.
- Cần sxhkd để có keybinding hoạt động.
