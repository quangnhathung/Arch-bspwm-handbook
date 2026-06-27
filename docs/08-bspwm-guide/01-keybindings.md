# Keybindings — Danh sách phím tắt

Ngày: 27/06/2026

## Ký hiệu

| Ký hiệu | Phím |
|---------|------|
| `Super` | Windows key (Mod4) |
| `Alt` | Alt (Mod1) |
| `Shift` | Shift |
| `Ctrl` | Control |
| `Return` | Enter |
| `Space` | Spacebar |
| `{1-9,0}` | Phím số 1-9 và 0 (= 10) |
| `{Left,Down,Up,Right}` | Phím mũi tên |
| `KP_Left,KP_Down,KP_Up,KP_Right` | Phím mũi tên (keypad tương thích) |
| `Prior` / `Next` | Page Up / Page Down |
| `XF86...` | Phím đặc biệt (volume, brightness, media) |

---

## WM Independent (Hệ thống)

| Phím | Chức năng | Ghi chú |
|------|-----------|---------|
| `Super + Return` | Mở terminal | Alacritty |
| `Super + space` | Mở launcher ứng dụng | Rofi drun |
| `Super + Escape` | Reload sxhkd | `pkill -USR1 -x sxhkd` |
| `alt + Tab` | Chuyển cửa sổ | Rofi window switcher |

## BSPWM — Quit / Restart

| Phím | Chức năng |
|------|-----------|
| `Super + Alt + q` | Thoát bspwm |
| `Super + Alt + r` | Restart bspwm |

## BSPWM — Đóng / Kill cửa sổ

| Phím | Chức năng |
|------|-----------|
| `Super + w` | Đóng cửa sổ (graceful — `bspc node -c`) |
| `Super + Shift + w` | Kill ứng dụng (force — `bspc node -k`) |

## BSPWM — Window State

| Phím | Chức năng |
|------|-----------|
| `Super + t` | Chuyển sang chế độ **tiled** |
| `Super + s` | Chuyển sang chế độ **floating** |
| `Super + f` | Chuyển sang chế độ **fullscreen** |

## BSPWM — Node Flags

| Phím | Chức năng |
|------|-----------|
| `Super + Ctrl + l` | Khóa vị trí (locked) |
| `Super + Ctrl + x` | Ghim cửa sổ (sticky — hiện trên mọi desktop) |
| `Super + Ctrl + p` | Riêng tư (private — ẩn khỏi bộ nhớ tạm) |

## BSPWM — Focus

| Phím | Chức năng |
|------|-----------|
| `Super + Left` | Focus sang trái (west) |
| `Super + Down` | Focus xuống dưới (south) |
| `Super + Up` | Focus lên trên (north) |
| `Super + Right` | Focus sang phải (east) |
| `Super + bracketleft` | Focus desktop trước đó |
| `Super + bracketright` | Focus desktop kế tiếp |
| `` Super + grave `` | Focus node cuối (last node) |
| `Super + Tab` | Focus desktop cuối (last desktop) |

## BSPWM — Swap cửa sổ

| Phím | Chức năng |
|------|-----------|
| `Super + Shift + Left` | Swap với cửa sổ bên trái |
| `Super + Shift + Down` | Swap với cửa sổ bên dưới |
| `Super + Shift + Up` | Swap với cửa sổ bên trên |
| `Super + Shift + Right` | Swap với cửa sổ bên phải |

## BSPWM — Workspace

| Phím | Chức năng |
|------|-----------|
| `Super + {1-9}` | Chuyển đến workspace 1-9 |
| `Super + 0` | Chuyển đến workspace 10 |
| `Super + Shift + {1-9}` | Di chuyển cửa sổ đến workspace 1-9 |
| `Super + Shift + 0` | Di chuyển cửa sổ đến workspace 10 |

## BSPWM — Preselect (chọn hướng chia)

| Phím | Chức năng |
|------|-----------|
| `Super + Shift + Left` | Preselect hướng trái (west) |
| `Super + Shift + Down` | Preselect hướng dưới (south) |
| `Super + Shift + Up` | Preselect hướng trên (north) |
| `Super + Shift + Right` | Preselect hướng phải (east) |
| `Super + Ctrl + {1-9}` | Đặt tỉ lệ chia 0.1 → 0.9 |
| `Super + Ctrl + space` | Cancel preselect (node hiện tại) |
| `Super + Ctrl + Shift + space` | Cancel tất cả preselect trên desktop |

## BSPWM — Resize

| Phím | Chức năng |
|------|-----------|
| `Super + Alt + Left` | Mở rộng — kéo cạnh trái ra ngoài |
| `Super + Alt + Down` | Mở rộng — kéo cạnh dưới xuống |
| `Super + Alt + Up` | Mở rộng — kéo cạnh trên lên |
| `Super + Alt + Right` | Mở rộng — kéo cạnh phải ra ngoài |
| `Super + Ctrl + Left` | Thu hẹp — kéo cạnh phải vào trong |
| `Super + Ctrl + Down` | Thu hẹp — kéo cạnh trên xuống |
| `Super + Ctrl + Up` | Thu hẹp — kéo cạnh dưới lên |
| `Super + Ctrl + Right` | Thu hẹp — kéo cạnh trái vào trong |

> **Giải thích:** `Super + Alt` mở rộng (ra xa tâm), `Super + Ctrl` thu hẹp (lại gần tâm).

## Âm thanh

| Phím | Chức năng | Script |
|------|-----------|--------|
| `XF86AudioRaiseVolume` | Tăng âm lượng | `~/.local/bin/volume.sh up` |
| `XF86AudioLowerVolume` | Giảm âm lượng | `~/.local/bin/volume.sh down` |
| `XF86AudioMute` | Tắt/Bật tiếng | `~/.local/bin/volume.sh mute` |
| `XF86AudioMicMute` | Tắt/Bật micro | `wpctl set-mute @DEFAULT_AUDIO_SOURCE@ toggle` |

## Độ sáng

| Phím | Chức năng | Script |
|------|-----------|--------|
| `XF86MonBrightnessUp` | Tăng sáng màn hình | `~/.local/bin/brightness.sh up` |
| `XF86MonBrightnessDown` | Giảm sáng màn hình | `~/.local/bin/brightness.sh down` |

## Wallpaper

| Phím | Chức năng |
|------|-----------|
| `Super + Prior` (Page Up) | Wallpaper trước |
| `Super + Next` (Page Down) | Wallpaper kế tiếp |
| `Super + Shift + Next` | Wallpaper ngẫu nhiên |

## UI / Utility

| Phím | Chức năng | Ghi chú |
|------|-----------|---------|
| `Super + Shift + b` | Dynamic Island | `~/.config/polybar/peek.sh` |
| `Print` | Chụp toàn màn hình + copy | Flameshot, lưu vào `~/images/Screenshots` |
| `Super + Print` | Mở Flameshot GUI (chọn vùng) | Flameshot AppImage |

---

## Cheat Sheet nhanh

```
# BSPWM
  Super + ←/↓/↑/→          → focus
  Super + Shift + ←/↓/↑/→  → swap window / preselect
  Super + Alt + ←/↓/↑/→    → expand
  Super + Ctrl + ←/↓/↑/→   → shrink

# Workspace
  Super + 1-9              → go to workspace 1-9
  Super + 0                → go to workspace 10
  Super + Shift + 1-9      → send window to workspace 1-9
  Super + Shift + 0        → send window to workspace 10

# Cửa sổ
  Super + w       → close
  Super + Shift + w   → kill
  Super + t       → tiled
  Super + s       → floating
  Super + f       → fullscreen

# Preselect
  Super + Ctrl + 1-9       → split ratio
  Super + Ctrl + space     → cancel preselection
  Super + Ctrl + Shift + space → cancel all

# Hệ thống
  Super + Return  → terminal (Alacritty)
  Super + space   → app launcher (Rofi)
  Super + Escape  → reload sxhkd
  alt + Tab       → window switcher
  Print           → screenshot full
  Super + Print   → screenshot region

# Hardware
  XF86AudioRaiseVolume  → volume up
  XF86AudioLowerVolume  → volume down
  XF86AudioMute         → mute toggle
  XF86AudioMicMute      → mic toggle
  XF86MonBrightnessUp   → brightness up
  XF86MonBrightnessDown → brightness down

# Wallpaper
  Super + Prior    → prev
  Super + Next     → next
  Super + Shift + Next → random

# Dynamic Island
  Super + Shift + b  → peek
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
