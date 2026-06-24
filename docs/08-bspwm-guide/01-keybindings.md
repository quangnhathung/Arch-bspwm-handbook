# Keybindings — Danh sách phím tắt

Ngày: 25/06/2026

## Ký hiệu

| Ký hiệu | Phím |
|---------|------|
| `Super` | Windows key (Mod4) |
| `Alt` | Alt (Mod1) |
| `Shift` | Shift |
| `Ctrl` | Control |
| `Return` | Enter |
| `Space` | Spacebar |
| `{1-9}` | Phím số 1-9 |
| `{h,j,k,l}` | h = trái, j = xuống, k = lên, l = phải |
| `XF86...` | Phím đặc biệt (volume, brightness, media) |

---

## Terminal & Launcher

| Phím | Chức năng | Ghi chú |
|------|-----------|---------|
| `Super + Return` | Mở terminal | Alacritty. **Không** dùng Super + t (đã dùng cho toggle tiled/floating) |
| `Super + d` | Mở launcher ứng dụng | Rofi drun |
| `Super + Shift + d` | Mở launcher lệnh | Rofi run |

## Đóng cửa sổ

| Phím | Chức năng |
|------|-----------|
| `Super + q` | Đóng cửa sổ hiện tại (gửi tín hiệu đóng — graceful) |
| `Super + Shift + q` | Kill ứng dụng (SIGKILL — force) |

## Focus (di chuyển con trỏ giữa các cửa sổ)

| Phím | Chức năng |
|------|-----------|
| `Super + h` | Focus sang trái (west) |
| `Super + j` | Focus xuống dưới (south) |
| `Super + k` | Focus lên trên (north) |
| `Super + l` | Focus sang phải (east) |

## Di chuyển cửa sổ

| Phím | Chức năng |
|------|-----------|
| `Super + Shift + h` | Đẩy cửa sổ sang trái |
| `Super + Shift + j` | Đẩy cửa sổ xuống dưới |
| `Super + Shift + k` | Đẩy cửa sổ lên trên |
| `Super + Shift + l` | Đẩy cửa sổ sang phải |

## Workspace

| Phím | Chức năng |
|------|-----------|
| `Super + {1-9}` | Chuyển đến workspace 1-9 |
| `Super + Shift + {1-9}` | Di chuyển cửa sổ đến workspace 1-9 |

## Window State

| Phím | Chức năng |
|------|-----------|
| `Super + t` | Toggle tiled ↔ floating |
| `Super + Shift + t` | Pseudo-tiled |
| `Super + o` | Toggle floating (alternative) |
| `Super + f` | Toggle fullscreen |
| `Super + m` | Toggle monocle layout |
| `Super + space` | Float + sticky (luôn hiện trên mọi workspace) |

## Preselect (chọn hướng chia trước khi mở cửa sổ)

| Phím | Chức năng |
|------|-----------|
| `Super + Ctrl + h` | Preselect split trái |
| `Super + Ctrl + j` | Preselect split dưới |
| `Super + Ctrl + k` | Preselect split trên |
| `Super + Ctrl + l` | Preselect split phải |
| `Super + Ctrl + space` | Cancel preselect |

## Resize

| Phím | Chức năng |
|------|-----------|
| `Super + Alt + h` | Resize — thu hẹp sang trái |
| `Super + Alt + j` | Resize — mở rộng xuống dưới |
| `Super + Alt + k` | Resize — mở rộng lên trên |
| `Super + Alt + l` | Resize — mở rộng sang phải |

## Hệ thống

| Phím | Chức năng | Ghi chú |
|------|-----------|---------|
| `Super + Escape` | Reload bspwm + sxhkd | `pkill -USR1 -x sxhkd && bspc wm -r` |
| `Super + Shift + Escape` | Lock screen | `i3lock-color` hoặc `betterlockscreen` (AUR) |
| `Super + Shift + x` | Power menu | Rofi script (shutdown, reboot, logout) |

## Âm thanh & Độ sáng

| Phím | Chức năng | Gói cần |
|------|-----------|---------|
| `XF86AudioRaiseVolume` | Tăng volume 5% | `pamixer` |
| `XF86AudioLowerVolume` | Giảm volume 5% | `pamixer` |
| `XF86AudioMute` | Toggle mute | `pamixer` |
| `XF86MonBrightnessUp` | Tăng độ sáng 5% | `brightnessctl` |
| `XF86MonBrightnessDown` | Giảm độ sáng 5% | `brightnessctl` |

## Media

| Phím | Chức năng | Gói cần |
|------|-----------|---------|
| `XF86AudioPlay` | Play / Pause | `playerctl` |
| `XF86AudioNext` | Next track | `playerctl` |
| `XF86AudioPrev` | Previous track | `playerctl` |

## Screenshot

| Phím | Chức năng | Gói cần |
|------|-----------|---------|
| `Print` | Chụp toàn màn hình | `maim` (có sẵn trong official repos) |
| `Super + Print` | Chụp vùng chọn | `maim -s` |

---

## Cheat Sheet nhanh

```
# Focus & di chuyển
  h ←     j ↓     k ↑     l →
  Super + h/j/k/l             → focus
  Super + Shift + h/j/k/l     → move window
  Super + Alt + h/j/k/l       → resize

# Workspace
  Super + 1-9              → go to workspace
  Super + Shift + 1-9      → send window to workspace

# Cửa sổ
  Super + q       → close
  Super + t       → toggle tiled/floating
  Super + f       → toggle fullscreen
  Super + m       → monocle mode
  Super + space   → float + sticky

# Hệ thống
  Super + Return  → terminal (Alacritty)
  Super + d       → app launcher (Rofi)
  Super + Escape  → reload config
  Print           → screenshot
  Super + Print   → selection screenshot
```

---

## Tùy chỉnh

Mở file cấu hình:

```bash
vim ~/.config/sxhkd/sxhkdrc
```

Sau đó reload:

```bash
pkill -USR1 -x sxhkd
```

> **Lưu ý:** sxhkd yêu cầu TAB indentation (hard tab), không dùng spaces.
