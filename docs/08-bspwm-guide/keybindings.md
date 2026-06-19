# Keybindings — Danh sách phím tắt

## Mục tiêu

Cung cấp danh sách đầy đủ các phím tắt cho bspwm/sxhkd để người mới dễ tra cứu.

## Ký hiệu

| Ký hiệu | Phím |
|---|---|
| `Super` | Windows key (Mod4) |
| `Alt` | Alt (Mod1) |
| `Shift` | Shift |
| `Ctrl` | Control |
| `Return` | Enter |
| `Space` | Spacebar |
| `{1-9}` | Phím số 1-9 |
| `{h,j,k,l}` | h=trái, j=xuống, k=lên, l=phải |
| `XF86...` | Phím đặc biệt (volume, brightness) |

## Tổng quan

```
Super + Enter    → Mở terminal
Super + d        → Mở launcher
Super + q        → Đóng cửa sổ
Super + 1-9      → Chuyển workspace
Super + h/j/k/l  → Focus cửa sổ
```

## Chi tiết

### Terminal và Launcher

| Phím | Chức năng |
|---|---|
| `Super + Return` | Mở terminal (Alacritty) |
| `Super + d` | Mở launcher ứng dụng (Rofi drun) |
| `Super + Shift + d` | Mở launcher lệnh (Rofi run) |
| `Super + r` | Mở launcher run |
| `Super + Shift + r` | Chuyển đổi cửa sổ (Rofi window) |

### Đóng cửa sổ

| Phím | Chức năng |
|---|---|
| `Super + q` | Đóng cửa sổ hiện tại (graceful) |
| `Super + Shift + q` | Kill ứng dụng (force) |

### Focus (Di chuyển giữa các cửa sổ)

| Phím | Chức năng |
|---|---|
| `Super + h` | Focus sang trái |
| `Super + j` | Focus xuống dưới |
| `Super + k` | Focus lên trên |
| `Super + l` | Focus sang phải |
| `Super + Tab` | Focus về cửa sổ trước đó |

### Di chuyển cửa sổ

| Phím | Chức năng |
|---|---|
| `Super + Shift + h` | Đẩy cửa sổ sang trái |
| `Super + Shift + j` | Đẩy cửa sổ xuống dưới |
| `Super + Shift + k` | Đẩy cửa sổ lên trên |
| `Super + Shift + l` | Đẩy cửa sổ sang phải |
| `Super + Shift + m` | Gửi cửa sổ sang màn hình khác |

### Workspace

| Phím | Chức năng |
|---|---|
| `Super + 1` | Chuyển đến workspace 1 |
| `Super + 2` | Chuyển đến workspace 2 |
| ... | ... |
| `Super + 9` | Chuyển đến workspace 9 |
| `Super + Shift + 1` | Di chuyển cửa sổ đến workspace 1 |
| `Super + Shift + 2` | Di chuyển cửa sổ đến workspace 2 |
| ... | ... |
| `Super + Shift + 9` | Di chuyển cửa sổ đến workspace 9 |

### Window State

| Phím | Chức năng |
|---|---|
| `Super + t` | Chuyển tiled ↔ floating |
| `Super + Shift + t` | Pseudo-tiled |
| `Super + o` | Floating |
| `Super + Shift + o` | Fullscreen |
| `Super + f` | Fullscreen (toggle) |
| `Super + Space` | Float + sticky |
| `Super + m` | Monocle mode (xếp chồng) |

### Split Direction (Preselect)

| Phím | Chức năng |
|---|---|
| `Super + Ctrl + h` | Preselect split trái |
| `Super + Ctrl + j` | Preselect split dưới |
| `Super + Ctrl + k` | Preselect split trên |
| `Super + Ctrl + l` | Preselect split phải |
| `Super + Ctrl + Space` | Cancel preselect |
| `Super + s` | Preselect east + ratio 0.5 |

### Resize

| Phím | Chức năng |
|---|---|
| `Super + Alt + h` | Resize sang trái -20px |
| `Super + Alt + j` | Resize xuống dưới +20px |
| `Super + Alt + k` | Resize lên trên -20px |
| `Super + Alt + l` | Resize sang phải +20px |

### Hệ thống

| Phím | Chức năng |
|---|---|
| `Super + Escape` | Reload bspwm + sxhkd |
| `Super + Shift + Escape` | Lock screen |
| `Super + Shift + x` | Power menu (rofi) |

### Media và Âm thanh

| Phím | Chức năng |
|---|---|
| `XF86AudioRaiseVolume` | Tăng volume 5% |
| `XF86AudioLowerVolume` | Giảm volume 5% |
| `XF86AudioMute` | Toggle mute |
| `XF86MonBrightnessUp` | Tăng độ sáng 5% |
| `XF86MonBrightnessDown` | Giảm độ sáng 5% |
| `XF86AudioPlay` | Play/Pause media |
| `XF86AudioNext` | Next track |
| `XF86AudioPrev` | Previous track |

### Ứng dụng

| Phím | Chức năng |
|---|---|
| `Super + b` | Mở Firefox |
| `Super + e` | Mở file manager (PCManFM) |

## Cheat Sheet nhanh

```
# Di chuyển / Focus
    h ←       j ↓       k ↑       l →
    Super + h/j/k/l         → focus
    Super + Shift + h/j/k/l → move window
    Super + Alt + h/j/k/l   → resize

# Workspace
    Super + 1-9          → go to workspace
    Super + Shift + 1-9  → send to workspace

# Cửa sổ
    Super + q       → close
    Super + t       → toggle tiled/floating
    Super + f       → toggle fullscreen
    Super + m       → monocle mode

# Hệ thống
    Super + Return  → terminal
    Super + d       → launcher
    Super + Escape  → reload config
```

## Tùy chỉnh

Để thêm/sửa keybinding:

```bash
vim ~/.config/sxhkd/sxhkdrc
```

Sau đó reload:

```bash
pkill -USR1 -x sxhkd
```

Xem thêm bài sxhkd.md trong 04-desktop.
