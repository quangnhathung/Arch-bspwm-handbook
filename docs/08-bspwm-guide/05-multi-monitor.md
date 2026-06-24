# Multi-Monitor — Nhiều màn hình

Ngày: 25/06/2026

## bspwm và multi-monitor

bspwm hỗ trợ multi-monitor tự nhiên:

- Mỗi monitor có cây node riêng (độc lập).
- Mỗi monitor có bộ workspace riêng.
- Có thể di chuyển cửa sổ và workspace giữa các monitor.
- Mỗi monitor có thể chạy Polybar instance riêng.

Cấu trúc:

```
Monitor eDP-1 (laptop, 1920×1080)
  ├── Workspace I
  ├── Workspace II
  └── Workspace III

Monitor HDMI-A-1 (ngoài, 1920×1080)
  ├── Workspace IV
  ├── Workspace V
  └── Workspace VI
```

---

## Các bước cấu hình

### Bước 1: Xác định tên monitor

```bash
xrandr --query
```

Output:

```
eDP-1 connected primary 1920x1080+0+0
HDMI-A-1 connected 1920x1080+0+0
```

### Bước 2: Cấu hình workspace cho từng monitor

Trong `bspwmrc`:

```bash
# Laptop + màn hình ngoài bên phải
xrandr --output eDP-1 --primary --mode 1920x1080 --rate 144
xrandr --output HDMI-A-1 --mode 1920x1080 --rate 60 --right-of eDP-1

bspc monitor eDP-1 -d I II III IV V
bspc monitor HDMI-A-1 -d VI VII VIII IX
```

### Bước 3: Chạy Polybar cho mỗi monitor

```bash
if type "xrandr"; then
    for m in $(xrandr --query | grep " connected" | cut -d" " -f1); do
        MONITOR=$m polybar main &
    done
else
    polybar main &
fi
```

### Bước 4: Di chuyển cửa sổ giữa monitor

```bash
# Sang monitor kế tiếp
bspc node -m next

# Đến monitor cụ thể
bspc node -m eDP-1

# Di chuyển và focus theo
bspc node -m next --follow
```

Phím tắt:

```
super + Shift + m
	bspc node -m next
```

### Bước 5: Focus giữa monitor

```bash
# Focus monitor tiếp theo
bspc monitor -f next

# Focus monitor cụ thể
bspc monitor -f eDP-1
```

### Bước 6: Di chuyển workspace giữa monitor

```bash
# Gửi workspace hiện tại sang monitor khác
bspc desktop --send eDP-1

# Hoán đổi workspace giữa hai monitor
bspc desktop eDP-1:^1 --swap HDMI-A-1:^6
```

---

## autorandr — tự động cấu hình

```bash
sudo pacman -S autorandr
```

```bash
# Lưu cấu hình hiện tại
autorandr --save work

# Khi cắm/rút màn hình, tự động load
autorandr --change
```

Thêm vào `bspwmrc`:

```bash
autorandr --change
```

---

## Các lệnh CLI hữu ích

```bash
# Liệt kê monitor
bspc query -M --names

# Xem workspace trên mỗi monitor
bspc query -D

# Di chuyển tất cả cửa sổ từ monitor này sang monitor khác
for wid in $(bspc query -N -m eDP-1); do
    bspc node "$wid" -m HDMI-A-1
done

# Xoay monitor
xrandr --output HDMI-A-1 --rotate left
```

---

## Best practices

1. **Màn hình chính (laptop)** dùng workspace I-V.
2. **Màn hình phụ** dùng workspace VI-IX.
3. **Mỗi màn hình có bar riêng** (Polybar instance riêng).
4. **Gán phím `Super + Shift + m`** để di chuyển cửa sổ nhanh.
5. **Dùng `autorandr`** để tự động chuyển đổi khi cắm/rút màn hình.
6. **Kiểm tra cấu hình xrandr** trước khi thêm vào bspwmrc.

---

## Troubleshooting

### Monitor mới không được nhận

```bash
xrandr --auto
xrandr --query
```

Nếu vẫn không thấy, kiểm tra cable hoặc driver GPU.

### Cửa sổ "biến mất" sau khi chuyển monitor

```bash
bspc node -f any           # tìm và focus bất kỳ cửa sổ nào
```

### Polybar không hiển thị trên monitor phụ

- Kiểm tra biến `MONITOR` đã set đúng chưa.
- Kiểm tra cấu hình multi-monitor trong `bspwmrc`.
- Mỗi monitor cần instance Polybar riêng.

### Workspace hiển thị sai trên Polybar

- Mỗi monitor có module workspace riêng.
- Kiểm tra config Polybar có dùng `internal/bspwm` đúng cách không.

### Khi rút màn hình ngoài, cửa sổ bị mất

Chạy lại `bspwmrc` hoặc:

```bash
bspc wm -r
```

Hoặc dùng `autorandr` để tự động xử lý.
