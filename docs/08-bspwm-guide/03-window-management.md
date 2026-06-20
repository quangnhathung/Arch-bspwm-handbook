# Layouts — Bố cục cửa sổ

## Mục tiêu

Hiểu các layout (bố cục) trong bspwm và cách sử dụng chúng.

## Kiến thức nền

### Layout là gì?

Layout là cách bố trí các cửa sổ trong một workspace. bspwm không có
layout cố định như i3 (tabbed, stacking). Thay vào đó, bspwm sử dụng
cấu trúc cây BSP (Binary Space Partition) động.

Tuy nhiên, bspwm hỗ trợ một số chế độ layout cho desktop:

| Chế độ | Mô tả |
|---|---|
| **Tiled** | Mặc định. Chia màn hình thành các ô. |
| **Monocle** | Xếp chồng tất cả cửa sổ lên nhau, chỉ thấy một cửa sổ. |
| **Floating** | Tất cả cửa sổ đều floating. |

## Tiled layout (mặc định)

### Cách hoạt động

Mỗi lần mở cửa sổ mới, node hiện tại bị chia làm hai.
Có thể chia dọc (sang trái/phải) hoặc chia ngang (lên/xuống).

```
Mở cửa sổ 1:
┌──────────────────┐
│                  │
│      Win 1      │
│                  │
└──────────────────┘

Mở cửa sổ 2 (chia dọc):
┌────────┬─────────┐
│        │         │
│ Win 1  │  Win 2  │
│        │         │
└────────┴─────────┘

Mở cửa sổ 3 (chia ngang win 2):
┌────────┬─────────┐
│        │  Win 2  │
│ Win 1  ├─────────┤
│        │  Win 3  │
└────────┴─────────┘
```

### Kiểm soát layout

Dùng preselect (Super + Ctrl + h/j/k/l) để chọn hướng chia trước:

```
Super + Ctrl + l → preselect phải
→ Mở trình duyệt → nó xuất hiện bên phải
```

### Alternating layout

Tạo layout xen kẽ dọc-ngang (giống i3):

Mặc định bspwm chia theo hướng hiện tại (longest side).
Để thay đổi:

```bash
# Chia dọc
bspc node -p east
# Mở cửa sổ → chia dọc

# Chia ngang
bspc node -p south
# Mở cửa sổ → chia ngang
```

### Equal ratio

Mặc định 50/50. Giữ nguyên 50/50 nếu không thay đổi split_ratio.
Khi resize, tỉ lệ thay đổi.

## Monocle layout

### Cách hoạt động

Tất cả cửa sổ xếp chồng lên nhau, chỉ thấy một cửa sổ tại một thời điểm.
Giống tab trên trình duyệt.

```
Workspace 5 (monocle):
┌──────────────────┐
│  [1] [2] [3]     │
│                  │
│    (cửa sổ 2)   │
│                  │
└──────────────────┘
```

### Kích hoạt

```bash
# Chuyển layout hiện tại sang monocle
bspc desktop -l monocle

# Quay lại tiled
bspc desktop -l tiled

# Toggle (luân phiên)
bspc desktop -l next
```

Phím tắt trong sxhkdrc:

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

- Khi workspace có quá nhiều cửa sổ (5+).
- Khi cần tập trung vào một cửa sổ nhưng vẫn giữ các cửa sổ khác mở.
- Khi làm việc trên màn hình nhỏ.

## Floating layout

### Cách hoạt động

Tất cả cửa sổ đều floating. Không tile.

### Kích hoạt

```bash
bspc desktop -l floating
```

### Khi nào dùng floating

- Khi muốn tự do sắp xếp cửa sổ bằng chuột.
- Khi làm việc với ứng dụng đồ họa (GIMP, Inkscape).
- Khi có nhiều dialog, palette.

## Tiled + Floating hỗn hợp

Trong layout tiled, vẫn có thể set từng cửa sổ riêng lẻ sang floating:

```
Super + t → toggle tiled/floating cho cửa sổ hiện tại
```

Cửa sổ floating nằm trên layer riêng, không ảnh hưởng đến layout tiled.

```
┌────────┬─────────┐
│        │         │
│ Win 1  │  Win 2  │  ← tiled
│        │         │
├────────┴─────────┤
│   ┌──────────┐  │
│   │  Win 3   │  │  ← floating (trên layer riêng)
│   │ (float)  │  │
│   └──────────┘  │
└─────────────────┘
```

## So sánh các chế độ

| Chế độ | Không gian | Nhìn thấy | Focus | Dùng khi |
|---|---|---|---|---|
| Tiled | Chia ô | Nhiều cửa sổ cùng lúc | Một cửa sổ | Mặc định |
| Monocle | Xếp chồng | Một cửa sổ | Tất cả | Nhiều cửa sổ |
| Floating | Tự do | Xếp layer | Di chuyển tự do | Ứng dụng đồ họa |

## Lệnh CLI

```bash
# Xem layout hiện tại
bspc query -D -d --names

# Set layout
bspc desktop -l tiled
bspc desktop -l monocle
bspc desktop -l floating

# Toggle
bspc desktop -l next
```

## Best practices

1. **Mặc định tiled**: Hầu hết thời gian.
2. **Chuyển sang monocle** khi cần focus, workspace đông.
3. **Chuyển từng cửa sổ** sang floating khi cần.
4. **Không dùng floating** cho workspace chính (mất kiểm soát).

## Troubleshooting

### Monocle không hoạt động

```bash
# Cần restart bspwm
bspc wm -r
```

### Tất cả cửa sổ đều floating

```bash
# Kiểm tra desktop layout
bspc query -D -d --names
# Nếu là floating → set lại tiled
bspc desktop -l tiled
```

## Tổng kết

- bspwm có 3 chế độ layout: tiled (mặc định), monocle, floating.
- Tiled là layout chính, dùng preselect để kiểm soát.
- Monocle xếp chồng cửa sổ, focus từng cái.
- Floating cho ứng dụng đặc thù.
- Kết hợp tiled + floating trong cùng workspace.
