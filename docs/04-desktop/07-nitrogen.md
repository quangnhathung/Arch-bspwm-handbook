# Sxhkd — Keybinding

## Mục tiêu

Cấu hình sxhkd (Simple X Hotkey Daemon) để điều khiển bspwm và ứng dụng
bằng bàn phím.

## Kiến thức nền

### sxhkd là gì?

sxhkd là một daemon (chương trình nền) lắng nghe sự kiện bàn phím và thực thi
lệnh khi phím tắt được nhấn.

```
Nhấn phím → sxhkd nhận sự kiện → tra cứu trong sxhkdrc → thực thi lệnh
```

### Cú pháp sxhkdrc

```
# Comment
modifier + key
    command

# Ví dụ:
super + Return
    alacritty
```

- `super`: Windows key (còn gọi là Mod4).
- Các modifier: `super`, `alt`, `control`, `shift`.
- Có thể kết hợp nhiều modifier: `super + shift + Return`.
- Lệnh thụt vào đầu dòng bằng tab (không phải space).

## Các bước thực hiện

### Bước 1: Cấu hình sxhkdrc

```bash
vim /home/archuser/.config/sxhkd/sxhkdrc
```

### Bước 2: File sxhkdrc đầy đủ

```bash
#
# sxhkdrc — bspwm keybindings
#

# ---- Terminal ----
super + Return
    alacritty

# ---- Launcher ----
super + d
    rofi -show drun
super + shift + d
    rofi -show run

# ---- Close window ----
super + q
    bspc node -c

# ---- Kill application ----
super + shift + q
    bspc node -k

# ---- Focus ----
super + {h,j,k,l}
    bspc node -f {west,south,north,east}

super + Tab
    bspc node -f last

# ---- Move window ----
super + shift + {h,j,k,l}
    bspc node -s {west,south,north,east}

# ---- Swap with other monitor ----
super + shift + m
    bspc node -m next

# ---- Split mode ----
super + {t,o}
    bspc node -t {tiled,floating}
super + shift + {t,o}
    bspc node -t {pseudo_tiled,fullscreen}

# ---- Split ratio ----
super + {1,2}
    bspc node -i # expand window
    bspc node -o # shrink window

# ---- Workspace ----
super + {1-9}
    bspc desktop -f '{1-9}'

# ---- Move to workspace ----
super + shift + {1-9}
    bspc node -d '{1-9}'

# ---- Resize window ----
super + alt + {h,j,k,l}
    bspc node -z {left -20 0,bottom 0 20,top 0 -20,right 20 0}

# ---- Reload configs ----
super + Escape
    pkill -USR1 -x sxhkd && bspc wm -r

# ---- Screenshot ----
Print
    maim -u ~/Pictures/screenshots/$(date +%Y%m%d-%H%M%S).png

super + Print
    maim -su ~/Pictures/screenshots/$(date +%Y%m%d-%H%M%S).png

# ---- Lock screen ----
super + shift + Escape
    betterlockscreen -l

# ---- Power menu ----
super + shift + x
    rofi -show power-menu -modi power-menu:rofi-power-menu

# ---- Volume ----
XF86AudioRaiseVolume
    pamixer -i 5
XF86AudioLowerVolume
    pamixer -d 5
XF86AudioMute
    pamixer -t

# ---- Brightness ----
XF86MonBrightnessUp
    brightnessctl set +5%
XF86MonBrightnessDown
    brightnessctl set 5%-

# ---- Media ----
XF86AudioPlay
    playerctl play-pause
XF86AudioNext
    playerctl next
XF86AudioPrev
    playerctl previous

# ---- Program shortcuts ----
super + b
    firefox
super + e
    pcmanfm
super + r
    rofi -show run
super + shift + r
    rofi -show window

# ---- Float / Unfloat ----
super + space
    bspc node -t floating; bspc node -g sticky

# ---- Stack / Unstack ----
super + s
    bspc node --presel-dir east; bspc node --presel-ratio 0.5

# ---- Toggle monocle ----
super + m
    bspc desktop -l next
```

### Bước 3: Thêm sxhkd vào bspwmrc

Kiểm tra file `bspwmrc` đã có dòng:

```bash
sxhkd &
```

Nếu chưa, thêm vào trước `exec` (hoặc đơn giản là chạy nền).

### Bước 4: Kiểm tra và reload

Sau khi chỉnh sửa sxhkdrc, reload:

```bash
pkill -USR1 -x sxhkd
```

## Giải thích các phím tắt quan trọng

### Cơ bản

| Phím | Chức năng |
|---|---|
| `Super + Enter` | Mở terminal (Alacritty) |
| `Super + d` | Mở launcher (Rofi) |
| `Super + q` | Đóng cửa sổ |
| `Super + h/j/k/l` | Focus sang trái/xuống/lên/phải |

### Workspace

| Phím | Chức năng |
|---|---|
| `Super + 1-9` | Chuyển sang workspace 1-9 |
| `Super + Shift + 1-9` | Di chuyển cửa sổ sang workspace 1-9 |

### Window management

| Phím | Chức năng |
|---|---|
| `Super + t` | Chuyển tiled/floating |
| `Super + Shift + t` | Pseudo-tiled / fullscreen |
| `Super + o` | Toggle floating |
| `Super + Space` | Float + sticky |

### Resize

| Phím | Chức năng |
|---|---|
| `Super + Alt + h/j/k/l` | Resize window |

## Ký hiệu trong sxhkdrc

- `super`: Mod4 (Windows key).
- `{h,j,k,l}`: Batch syntax — sinh ra 4 binding riêng biệt.
  ```super + {h,j,k,l}``` tương đương 4 dòng riêng lẻ.
- `{1-9}`: Range syntax — sinh ra 9 binding.

## Troubleshooting

### sxhkd không hoạt động

```bash
# Kiểm tra sxhkd có chạy không
pgrep -x sxhkd

# Nếu không chạy, start
sxhkd &

# Kiểm tra lỗi
killall sxhkd
sxhkd -t 10   # Chạy foreground với timeout 10s, log lỗi ra terminal
```

### Phím tắt không hoạt động

- Kiểm tra syntax trong sxhkdrc (dùng tab, không space cho lệnh).
- Reload: `pkill -USR1 -x sxhkd`.
- Kiểm tra xem phím tắt có bị conflict với ứng dụng khác không.

## Tổng kết

- sxhkdrc đã được cấu hình với các phím tắt quan trọng.
- sxhkd chạy nền và lắng nghe phím.
- Các phím tắt bao gồm: terminal, launcher, window management, workspace,
  resize, volume, brightness, media.
