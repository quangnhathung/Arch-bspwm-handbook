# Sxhkd — Keybinding

Ngày cập nhật: 27/06/2026

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
- **Lệnh phải thụt vào đầu dòng bằng TAB, không phải space.**
  Đây là lỗi cú pháp thường gặp nhất.

### Batch syntax

sxhkd hỗ trợ batch syntax để viết nhiều binding trên một dòng:

```
# {a,b,c} syntax
super + {h,j,k,l}
	bspc node -f {west,south,north,east}

# {1-9} range syntax
super + {1-9}
	bspc desktop -f '{1-9}'
```

## Các bước thực hiện

### Bước 1: Cấu hình sxhkdrc

```bash
vim /home/archuser/.config/sxhkd/sxhkdrc
```

### Bước 2: File sxhkdrc đầy đủ

```bash
#
# wm independent hotkeys
#

# terminal emulator
super + Return
	alacritty

# program launcher
super + space
    pgrep -x rofi >/dev/null && killall rofi || rofi -show drun -theme ~/.config/rofi/config.rasi

# make sxhkd reload its configuration file:
super + Escape
	pkill -USR1 -x sxhkd

#
# bspwm hotkeys
#

# quit/restart bspwm
super + alt + {q,r}
	bspc {quit,wm -r}

# close (w) and kill (shift + w)
super + {_,shift + }w
	bspc node -{c,k}

# state/flags
super + {t,s,f}
	bspc node -t {tiled,floating,fullscreen}

super + ctrl + {l,x,p}
	bspc node -g {locked,sticky,private}

# focus/swap
super + {_,shift + }{Left,Down,Up,Right}
	bspc node -{f,s} {west,south,north,east}

# focus the next/previous desktop in the current monitor
super + bracket{left,right}
	bspc desktop -f {prev,next}.local

# focus the last node/desktop
super + {grave,Tab}
	bspc {node,desktop} -f last

# focus or send to the given desktop
super + {_,shift + }{1-9,0}
	bspc {desktop -f,node -d} '^{1-9,10}'

# preselect
super + shift + {Left,Down,Up,Right}
	bspc node -p {west,south,north,east}

super + ctrl + {1-9}
	bspc node -o 0.{1-9}

super + ctrl + space
	bspc node -p cancel

super + ctrl + shift + space
	bspc query -N -d | xargs -I id -n 1 bspc node id -p cancel

# move/resize
super + alt + {Left,Down,Up,Right}
	bspc node -z {left -20 0,bottom 0 20,top 0 -20,right 20 0}

super + ctrl + {Left,Down,Up,Right}
	bspc node -z {right -20 0,top 0 20,bottom 0 -20,left 20 0}

# brightness
XF86MonBrightnessUp
	~/.local/bin/brightness.sh up

XF86MonBrightnessDown
	~/.local/bin/brightness.sh down

# wallpaper
super + Prior
    ~/.local/bin/wallpaper.sh prev

super + Next
    ~/.local/bin/wallpaper.sh next

super + shift + Next
    ~/.local/bin/wallpaper.sh random

# dynamic island
super + shift + b
    ~/.config/polybar/peek.sh

# volume
XF86AudioRaiseVolume
	~/.local/bin/volume.sh up

XF86AudioLowerVolume
	~/.local/bin/volume.sh down

XF86AudioMute
	~/.local/bin/volume.sh mute

XF86AudioMicMute
	wpctl set-mute @DEFAULT_AUDIO_SOURCE@ toggle

# window switcher
alt + Tab
    rofi -show window -theme ~/.config/rofi/config.rasi

# screenshot
super + Print
	~/flameshot-13.3.AppImage gui

Print
	~/flameshot-13.3.AppImage gui --accept-on-select -p ~/images/Screenshots -c
```

### Bước 3: Kiểm tra và reload

Sau khi chỉnh sửa sxhkdrc, reload:

```bash
pkill -USR1 -x sxhkd
```

Nếu không có hiệu lực, chạy sxhkd foreground để kiểm tra lỗi:

```bash
killall sxhkd
sxhkd -t 10   # Chạy foreground với timeout 10s, log lỗi ra terminal
```

## Giải thích các phím tắt quan trọng

### Cơ bản

| Phím | Chức năng |
|---|---|
| `Super + Enter` | Mở terminal (Alacritty) |
| `Super + space` | Mở launcher (Rofi drun) |
| `alt + Tab` | Chuyển cửa sổ (Rofi window) |

### BSPWM — Đóng / Kill

| Phím | Chức năng |
|---|---|
| `Super + w` | Đóng cửa sổ (graceful) |
| `Super + Shift + w` | Kill ứng dụng (force) |

### BSPWM — Focus

| Phím | Chức năng |
|---|---|
| `Super + ←/↓/↑/→` | Focus theo hướng |
| `Super + bracketleft` | Desktop trước |
| `Super + bracketright` | Desktop kế tiếp |
| `Super + grave` | Focus node cuối |
| `Super + Tab` | Focus desktop cuối |

### BSPWM — Swap

| Phím | Chức năng |
|---|---|
| `Super + Shift + ←/↓/↑/→` | Swap cửa sổ theo hướng |

### Workspace

| Phím | Chức năng |
|---|---|
| `Super + 1-9` | Chuyển sang workspace 1-9 |
| `Super + 0` | Chuyển sang workspace 10 |
| `Super + Shift + 1-9` | Di chuyển cửa sổ sang workspace 1-9 |
| `Super + Shift + 0` | Di chuyển cửa sổ sang workspace 10 |

### Window state & flags

| Phím | Chức năng |
|---|---|
| `Super + t` | Tiled |
| `Super + s` | Floating |
| `Super + f` | Fullscreen |
| `Super + Ctrl + l` | Locked flag |
| `Super + Ctrl + x` | Sticky flag |
| `Super + Ctrl + p` | Private flag |

### Preselect split

| Phím | Chức năng |
|---|---|
| `Super + Shift + ←/↓/↑/→` | Preselect split direction |
| `Super + Ctrl + 1-9` | Preselect ratio (0.1 → 0.9) |
| `Super + Ctrl + space` | Cancel preselect (node) |
| `Super + Ctrl + Shift + space` | Cancel all preselects |

### Resize

| Phím | Chức năng |
|---|---|
| `Super + Alt + ←/↓/↑/→` | Expand window (ra xa tâm) |
| `Super + Ctrl + ←/↓/↑/→` | Contract window (lại gần tâm) |

## Ký hiệu trong sxhkdrc

- `super`: Mod4 (Windows key).
- `{h,j,k,l}`: Batch syntax — sinh ra 4 binding riêng biệt.
  `super + {h,j,k,l}` tương đương 4 dòng riêng lẻ.
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

- Kiểm tra syntax trong sxhkdrc (dùng **tab**, không space cho lệnh).
- Reload: `pkill -USR1 -x sxhkd`.
- Kiểm tra xem phím tắt có bị conflict với ứng dụng khác không.
- Một số phím (như Print, XF86*) có thể bị chiếm bởi DE component khác.

## Tổng kết

- sxhkdrc đã được cấu hình với các phím tắt quan trọng.
- sxhkd chạy nền và lắng nghe phím.
- Các phím tắt bao gồm: terminal, launcher, window management, workspace,
  resize, volume, brightness, media.
