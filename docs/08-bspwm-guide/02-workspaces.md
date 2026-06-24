# Workspaces — Desktop ảo

Ngày: 25/06/2026

## Workspace là gì?

Workspace (desktop ảo) là màn hình ảo riêng biệt. Mỗi workspace chứa một bộ cửa sổ riêng, không ảnh hưởng đến workspace khác. Bạn chỉ thấy một workspace tại một thời điểm.

```
Workspace 1          Workspace 2          Workspace 3
+----------------+   +----------------+   +----------------+
| Terminal       |   | Browser        |   | Music          |
| Code           |   | Documentation  |   | Chat           |
+----------------+   +----------------+   +----------------+
```

### Tại sao dùng workspace?

- **Tổ chức:** Mỗi không gian cho một công việc riêng.
- **Tập trung:** Không bị phân tâm bởi cửa sổ không liên quan.
- **Đa màn hình:** Mỗi màn hình có bộ workspace riêng.

---

## Cấu hình workspace

Mặc định 9 workspace (I, II, III, ..., IX).

Trong `bspwmrc`:

```bash
# Tạo 9 workspace với tên La Mã
bspc monitor -d I II III IV V VI VII VIII IX

# Hoặc tên theo mục đích
bspc monitor -d term code web chat music files mail sys games

# Hoặc số
bspc monitor -d 1 2 3 4 5 6 7 8 9
```

### Multi-monitor

Mỗi monitor có bộ workspace riêng:

```bash
bspc monitor eDP-1 -d I II III IV V
bspc monitor HDMI-A-1 -d VI VII VIII IX
```

---

## Thao tác với workspace

| Thao tác | Phím / Lệnh | Mô tả |
|----------|-------------|-------|
| Chuyển workspace | `Super + {1-9}` | Chuyển đến workspace số tương ứng |
| Gửi cửa sổ | `Super + Shift + {1-9}` | Di chuyển cửa sổ hiện tại đến workspace khác |
| CLI chuyển | `bspc desktop -f {name}` | Ví dụ: `bspc desktop -f 5` |
| CLI gửi | `bspc node -d {name}` | Di chuyển cửa sổ đến workspace |
| Gửi + follow | `bspc node -d {name} --follow` | Di chuyển và chuyển focus theo |

### Quay lại workspace trước

```bash
bspc desktop -f last
```

Có thể gán phím `Super + Tab` hoặc `Super + grave` (dấu `).

---

## Gán ứng dụng vào workspace cụ thể (bspc rule)

Trong `bspwmrc`:

```bash
bspc rule -a Firefox desktop='^2'
bspc rule -a Alacritty desktop='^1'
bspc rule -a thunderbird desktop='^5'
bspc rule -a Spotify desktop='^3'
```

- `desktop='^2'` — mở trên workspace số 2 của monitor hiện tại.
- Dấu `^` chỉ "workspace số N của monitor hiện tại".

Kiểm tra rule đang hoạt động:

```bash
bspc rule -l
```

---

## Polybar — hiển thị workspace status

Polybar module `bspwm` tự động hiển thị trạng thái workspace:

| Trạng thái | Ý nghĩa |
|------------|---------|
| Focused | Workspace đang dùng |
| Occupied | Có cửa sổ nhưng không focus |
| Empty | Workspace trống |
| Urgent | Ứng dụng cần chú ý |

Cấu hình module trong `~/.config/polybar/config.ini`:

```ini
[module/bspwm]
type = internal/bspwm
label-focused = %name%
label-focused-foreground = #89B4FA
label-occupied = %name%
label-occupied-foreground = #CDD6F4
label-urgent = %name%
label-urgent-foreground = #F38BA8
label-empty = %name%
label-empty-foreground = #585B70
```

---

## Best practices

1. **Dành workspace cố định cho ứng dụng thường dùng:**
   - WS 1: Terminal / Code
   - WS 2: Trình duyệt
   - WS 3: Chat (Telegram, Discord)
   - WS 4: File manager
   - WS 9: Ứng dụng tạm thời

2. **Mỗi workspace chỉ nên có 2-4 cửa sổ.** Nếu quá nhiều, chia ra workspace khác hoặc dùng monocle layout.

3. **Dùng rule để tự động gán ứng dụng** vào đúng workspace ngay khi mở.

4. **Quay lại workspace trước (`Super + Tab`)** khi cần so sánh nhanh.

5. **Multi-monitor:** Màn hình chính (laptop) dùng workspace đầu; màn hình phụ dùng workspace cuối.

---

## Troubleshooting

### Workspace không hiển thị trên Polybar

- Kiểm tra module `bspwm` trong Polybar config đã đúng chưa.
- Restart Polybar: `killall polybar; polybar main &`

### Không chuyển workspace được

- Kiểm tra sxhkd có chạy: `pgrep -x sxhkd`.
- Kiểm tra keybinding trong `sxhkdrc`.

### Ứng dụng không mở đúng workspace

- Kiểm tra rule: `bspc rule -l`.
- Class name có thể khác tên ứng dụng. Dùng `xprop WM_CLASS` để xác định:

```bash
xprop WM_CLASS
# Click vào cửa sổ cần kiểm tra → hiện class name
```
