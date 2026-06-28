# Bspwm — Window Manager

Ngày cập nhật: 28/06/2026

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

### Bước 3: Cài các chương trình autostart (quan trọng)

Các chương trình trong bspwmrc cần được cài trước, nếu không chúng sẽ báo lỗi
hoặc không khởi động được:

```bash
sudo pacman -S picom dunst polkit-gnome xorg-xinput
```

Chi tiết:

| Gói | Mục đích | Bắt buộc? |
|---|---|---|
| `picom` | Compositor (đổ bóng, chống tearing) | Khuyên dùng |
| `dunst` | Notification daemon (volume, brightness alerts) | Nên có |
| `polkit-gnome` | Xác thực quyền cho GUI (mount USB, cài app) | Nên có |
| `xorg-xinput` | Cấu hình input (touchpad tap-to-click) | Nếu dùng touchpad |
| `xdotool` | Điều khiển cửa sổ từ CLI (polybar launch/peek) | Cần cho Dynamic Island |

> **Lưu ý:** Trong cấu hình thực tế, `nitrogen`, `nm-applet`, `blueman-applet`, `xfce4-power-manager`
> **không được dùng**. Polybar Dynamic Island + script `wallpaper.sh` + Dunst thay thế hoàn toàn các
> chức năng đó. Không cần cài các gói này nếu muốn theo đúng config.

> **Lưu ý về `volumeicon`:** Không dùng `volumeicon` — đã thay thế bằng module `pulseaudio`
> trong Polybar + `volume.sh` script + Dunst notification.

### Bước 4: Tạo file cấu hình bspwmrc

```bash
vim /home/archuser/.config/bspwm/bspwmrc
```

Nội dung (cấu hình thực tế trên máy — Catppuccin Dynamic Island):

```bash
#!/usr/bin/env bash

# =========================================================
# 1. AUTOSTART - Khởi động các tiến trình nền
# =========================================================
# Vá lỗi Touchpad: Tự động quét và bật Tapping bằng tên thay vì ID cứng
for id in $(xinput list | grep -i "touchpad" | grep -o 'id=[0-9]*' | cut -d= -f2); do
    xinput set-prop "$id" "libinput Tapping Enabled" 1
done

# Khởi động trình quản lý phím tắt
pgrep -x sxhkd > /dev/null || sxhkd &

# Fix lỗi trỏ chuột biến thành dấu X đen trên nền desktop trống
xsetroot -cursor_name left_ptr &

# Khởi động các tác vụ hệ thống và giao diện
/usr/lib/polkit-gnome/polkit-gnome-authentication-agent-1 &
~/.local/bin/wallpaper.sh &
picom --config ~/.config/picom/picom.conf &
~/.config/polybar/launch.sh &
pgrep -x dunst > /dev/null || dunst &
~/flameshot-13.3.AppImage &

# =========================================================
# 2. WORKSPACE & MONITORS
# =========================================================
bspc monitor -d I II III IV V VI VII VIII IX

# Padding = 0 để Polybar Dynamic Island hoạt động hoàn toàn nổi (float)
bspc config top_padding         0
bspc config bottom_padding      0
bspc config left_padding        0
bspc config right_padding       0

# =========================================================
# 3. APPEARANCE - Viền cửa sổ
# =========================================================
bspc config border_width         1
bspc config window_gap           3
bspc config focused_border_color "#89B4FA" # Xanh biển (cửa sổ đang chọn)
bspc config normal_border_color  "#45475A" # Xám đen (cửa sổ không chọn)
bspc config active_border_color  "#F38BA8" # Đỏ/Hồng (khi có thông báo/urgency)

# =========================================================
# 4. BEHAVIOR - Hành vi
# =========================================================
bspc config split_ratio          0.65
bspc config borderless_monocle   true
bspc config gapless_monocle      true
bspc config focus_follows_pointer true   # Chuột chỉ vào đâu, tự động focus
bspc config pointer_modifier     mod4    # Super + chuột trái để kéo cửa sổ float

# =========================================================
# 5. WINDOW RULES
# =========================================================
bspc rule -a Gimp desktop='^8' state=floating follow=on
bspc rule -a mplayer2 state=floating
bspc rule -a Kupfer.py focus=on
bspc rule -a Screenkey manage=off
bspc rule -a Flameshot state=floating
bspc rule -a flameshot state=floating
bspc rule -a Pavucontrol state=floating center=true
bspc config automatic_scheme alternate
```

Giải thích từng phần:

#### Autostart — `for id in $(xinput list | grep -i "touchpad")...`

Tự động quét ID touchpad và bật **tap-to-click** (chạm để click). Không fix cứng ID vì ID có thể thay đổi qua mỗi lần boot. Thay thế cho cách cũ dùng `xinput set-prop 12 304 1` (gõ sai ID là hỏng).

#### `xsetroot -cursor_name left_ptr &`

Fix lỗi con trỏ chuột biến thành dấu X đen trên desktop trống (thường gặp trên Arch với bspwm).

#### Autostart sequence

- `polkit-gnome` → Xác thực quyền GUI (mount USB, cài app, ...).
- `wallpaper.sh` → Script đặt wallpaper + ghi nhớ ảnh cuối.
- `picom` → Compositor (shadow, round corner, animation).
- `polybar/launch.sh` → Khởi động Dynamic Island bar (ẩn sẵn chờ peek).
- `dunst` → Notification daemon (volume, brightness, ...).
- `flameshot AppImage` → Chụp màn hình.

#### Padding = 0

Tất cả padding đặt về 0 vì Polybar chạy ở chế độ **override-redirect = true** (floating hoàn toàn, không chiếm không gian bspwm). Không cần top_padding.

#### `bspc config`

- `border_width 1`: Viền mỏng hơn mặc định (2px) — nhìn thanh lịch hơn.
- `window_gap 3`: Khoảng cách hẹp 3px giữa các cửa sổ — tiết kiệm không gian.
- `focused_border_color "#89B4FA"`: Màu xanh Catppuccin Blue — đồng bộ với Polybar.
- `normal_border_color "#45475A"`: Màu xám Catppuccin Surface1.
- `active_border_color "#F38BA8"`: Màu đỏ Catppuccin Red — báo urgency.

#### `focus_follows_pointer = true`

Chuột chỉ vào cửa sổ nào → tự động focus. Rất tiện khi code: không cần nhấn phím để chuyển focus, chỉ cần di chuột.

#### `pointer_modifier = mod4`

Giữ **Super (Windows key)** + chuột trái để kéo thả cửa sổ đang floating. Rất trực quan.

#### `split_ratio = 0.65`

Khi chia cửa sổ mới, node mới chiếm 35%, node cũ giữ 65% — không phải 50/50 như mặc định.

#### `automatic_scheme alternate`

Cách chia tự động: luân phiên giữa ngang và dọc thay vì luôn cùng hướng.

### Bước 5: Phân quyền executable

```bash
chmod +x /home/archuser/.config/bspwm/bspwmrc
```

bspwmrc phải có quyền execute (là một script bash).

### Bước 6: Cập nhật .xinitrc

```bash
echo "exec bspwm" > /home/archuser/.xinitrc
```

### Bước 7: Kiểm tra cấu hình (sau reboot)

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

### polkit-gnome-authentication-agent-1: No such file or directory

```bash
# Cài polkit-gnome
pacman -S polkit-gnome

# Tìm đường dẫn đúng
which polkit-gnome-authentication-agent-1
# Kết quả: /usr/lib/polkit-gnome/polkit-gnome-authentication-agent-1 (hoặc /usr/libexec/...)
```

## Tổng kết

- bspwm đã được cài và cấu hình cơ bản.
- bspwmrc đã được tạo với các setting cho laptop.
- Cần sxhkd để có keybinding hoạt động.
