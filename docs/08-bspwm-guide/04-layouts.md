# Layouts — Bố cục cửa sổ

Ngày: 25/06/2026

bspwm có 3 chế độ layout cho desktop: **tiled**, **monocle**, **floating**.

| Chế độ | Mô tả |
|--------|-------|
| **Tiled** | Mặc định. Chia màn hình thành các ô bằng BSP (Binary Space Partition). |
| **Monocle** | Xếp chồng tất cả cửa sổ, chỉ thấy một cửa sổ tại một thời điểm. |
| **Floating** | Tất cả cửa sổ đều floating, không tile. |

---

## Tiled layout (mặc định)

### Cách hoạt động

Mỗi lần mở cửa sổ mới, node hiện tại bị chia làm hai (dọc hoặc ngang). Hướng chia phụ thuộc vào preselect hoặc longest side mặc định.

```
Mở cửa sổ 1:
┌────────────────────┐
│                    │
│       Win 1       │
│                    │
└────────────────────┘

Mở cửa sổ 2 (chia dọc):
┌──────────┬─────────┐
│          │         │
│  Win 1   │  Win 2  │
│          │         │
└──────────┴─────────┘

Mở cửa sổ 3 (chia ngang Win 2):
┌──────────┬─────────┐
│          │  Win 2  │
│  Win 1   ├─────────┤
│          │  Win 3  │
└──────────┴─────────┘

Mở cửa sổ 4 (chia ngang Win 1):
┌──────────┬─────────┐
│  Win 1a  │  Win 2  │
├──────────┤─────────┤
│  Win 1b  │  Win 3  │
└──────────┴─────────┘
```

### Kiểm soát hướng chia

Dùng preselect để chọn hướng chia trước:

```
Super + Ctrl + l → preselect phải
→ Mở trình duyệt → nó xuất hiện bên phải
```

Không preselect → bspwm tự động chia theo cạnh dài nhất (longest side).

### Tỉ lệ chia

Mặc định 50/50. Giữ nguyên nếu không resize. Khi resize, tỉ lệ thay đổi.

---

## Monocle layout

### Cách hoạt động

Tất cả cửa sổ xếp chồng lên nhau, chỉ thấy một cửa sổ tại một thời điểm. Giống tab trên trình duyệt.

```
Workspace 5 (monocle):
┌────────────────────┐
│  [1] [2] [3]       │
│                    │
│     (cửa sổ 2)    │
│                    │
└────────────────────┘
```

### Kích hoạt

```bash
# Chuyển layout hiện tại sang monocle
bspc desktop -l monocle

# Quay lại tiled
bspc desktop -l tiled

# Toggle luân phiên (tiled → monocle → floating → tiled)
bspc desktop -l next
```

Phím tắt:

```
super + m
	bspc desktop -l next
```

### Chuyển cửa sổ trong monocle

```bash
# Focus cửa sổ tiếp theo
bspc node -f next.local

# Focus cửa sổ trước
bspc node -f prev.local
```

### Khi nào dùng monocle

- Workspace có 5+ cửa sổ.
- Cần tập trung vào một cửa sổ nhưng vẫn giữ các cửa sổ khác mở.
- Làm việc trên màn hình nhỏ (laptop 13-14").

---

## Floating layout

### Cách hoạt động

Tất cả cửa sổ đều floating. Không tile. Có thể kéo thả tự do.

```
Workspace 4 (floating):
┌────────────────────┐
│  ┌──────┐         │
│  │ Win1 │         │
│  └──────┘         │
│       ┌──────────┐│
│       │  Win2    ││
│       └──────────┘│
│ ┌────────────┐    │
│ │   Win3     │    │
│ └────────────┘    │
└────────────────────┘
```

### Kích hoạt

```bash
bspc desktop -l floating
```

### Khi nào dùng floating

- Làm việc với ứng dụng đồ họa (GIMP, Inkscape, Krita).
- Nhiều dialog, palette cần sắp xếp tự do.
- Trình chiếu, kiosk mode.

---

## Tiled + Floating hỗn hợp

Trong layout tiled, vẫn có thể set từng cửa sổ riêng lẻ sang floating:

```
Super + t → toggle tiled/floating cho cửa sổ hiện tại
```

Cửa sổ floating nằm trên layer riêng, không ảnh hưởng đến layout tiled:

```
┌──────────┬─────────┐
│          │         │
│  Win 1   │  Win 2  │  ← tiled
│          │         │
├──────────┴─────────┤
│   ┌────────────┐  │
│   │   Win 3    │  │  ← floating (trên layer riêng)
│   │  (float)   │  │
│   └────────────┘  │
└───────────────────┘
```

Đây là chế độ linh hoạt nhất: tiled cho workflow chính, floating cho dialog/popup.

---

## So sánh các chế độ

| Chế độ | Không gian | Nhìn thấy | Focus | Dùng khi |
|--------|-----------|-----------|-------|----------|
| Tiled | Chia ô | Nhiều cửa sổ cùng lúc | Một cửa sổ | Mặc định, code, browsing |
| Monocle | Xếp chồng | Một cửa sổ | Tất cả | Nhiều cửa sổ, màn hình nhỏ |
| Floating | Tự do | Xếp layer | Di chuyển tự do | Ứng dụng đồ họa, dialog |

---

## Lệnh CLI

```bash
# Xem layout hiện tại
bspc query -D -d --names

# Set layout
bspc desktop -l tiled
bspc desktop -l monocle
bspc desktop -l floating

# Toggle (tiled → monocle → floating → ...)
bspc desktop -l next
```

---

## Best practices

1. **Mặc định tiled** — hầu hết thời gian.
2. **Chuyển sang monocle** khi workspace quá đông hoặc cần tập trung.
3. **Floating** chỉ dùng cho ứng dụng đặc thù (GIMP, dialog).
4. **Tận dụng tiled + floating hỗn hợp:** workspace chính để tiled, dialog/popup tự động floating nhờ rule.

---

## Troubleshooting

### Monocle không hoạt động

```bash
bspc wm -r   # restart bspwm (không mất cửa sổ)
```

### Tất cả cửa sổ đều floating

```bash
# Kiểm tra desktop layout
bspc query -D -d --names

# Nếu là floating → set lại tiled
bspc desktop -l tiled
```
