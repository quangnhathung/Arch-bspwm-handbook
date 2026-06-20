# Multi-Monitor — Nhiều màn hình

## Mục tiêu

Cấu hình và sử dụng nhiều màn hình với bspwm.

## Kiến thức nền

### bspwm và multi-monitor

bspwm hỗ trợ multi-monitor tự nhiên:

- Mỗi monitor có cây node riêng.
- Mỗi monitor có bộ workspace riêng.
- Có thể di chuyển cửa sổ giữa các monitor.
- Có thể có bar riêng cho mỗi monitor.

### Cơ chế

bspwm gán một tập workspace cho mỗi monitor.

Ví dụ: laptop + màn hình ngoài

```
eDP-1 (laptop, 1920x1080)
  Workspace: I II III IV V
HDMI-A-1 (màn hình ngoài, 1920x1080)
  Workspace: VI VII VIII IX
```

## Các bước thực hiện

### Bước 1: Xác định monitor

```bash
xrandr --query
```

Output:

```
eDP-1 connected primary 1920x1080+0+0 (normal)
HDMI-A-1 connected 1920x1080+0+0 (normal)
```

### Bước 2: Cấu hình monitor trong bspwmrc

Trong `bspwmrc`:

```bash
# Nếu có 1 monitor
# bspc monitor eDP-1 -d I II III IV V VI VII VIII IX

# Nếu có 2 monitor
bspc monitor eDP-1 -d I II III IV V
bspc monitor HDMI-A-1 -d VI VII VIII IX

# Nếu monitor ngoài ở bên phải
xrandr --output HDMI-A-1 --right-of eDP-1
```

### Bước 3: Cấu hình Polybar cho multi-monitor

Trong `bspwmrc`:

```bash
# Polybar for each monitor
if type "xrandr"; then
    for m in $(xrandr --query | grep " connected" | cut -d" " -f1); do
        MONITOR=$m polybar main &
    done
else
    polybar main &
fi
```

### Bước 4: Di chuyển cửa sổ giữa các monitor

```bash
# Di chuyển sang monitor kế tiếp
bspc node -m next

# Di chuyển đến monitor cụ thể
bspc node -m eDP-1

# Di chuyển và focus theo
bspc node -m next --follow
```

Phím tắt:

```
super + shift + m
    bspc node -m next
```

### Bước 5: Focus giữa các monitor

```bash
# Focus monitor tiếp theo
bspc monitor -f next

# Focus monitor cụ thể
bspc monitor -f eDP-1
```

### Bước 6: Gửi workspace giữa monitor

```bash
# Gửi workspace hiện tại sang monitor khác
bspc desktop --send eDP-1
```

## Cấu hình thường dùng

### Laptop + màn hình ngoài bên phải

```bash
#!/bin/bash
# Trong bspwmrc

# Set monitor layout
xrandr --output eDP-1 --primary --mode 1920x1080 --rate 144
xrandr --output HDMI-A-1 --mode 1920x1080 --rate 60 --right-of eDP-1

# Assign workspaces
bspc monitor eDP-1 -d I II III IV V
bspc monitor HDMI-A-1 -d VI VII VIII IX
```

### Laptop + màn hình ngoài bên trái

```bash
xrandr --output HDMI-A-1 --mode 1920x1080 --rate 60 --left-of eDP-1
```

### Laptop + màn hình ngoài phía trên

```bash
xrandr --output HDMI-A-1 --mode 1920x1080 --rate 60 --above eDP-1
```

### Tự động cấu hình khi cắm màn hình

Dùng `autorandr`:

```bash
pacman -S autorandr
```

```bash
# Lưu cấu hình hiện tại
autorandr --save work

# Khi cắm màn hình, tự động load
autorandr --change
```

Thêm vào `bspwmrc`:

```bash
autorandr --change
```

## Lệnh multi-monitor hữu ích

```bash
# Liệt kê monitor
bspc query -M --names

# Xem workspace trên mỗi monitor
bspc query -D

# Di chuyển tất cả cửa sổ từ monitor này sang monitor khác
for wid in $(bspc query -N -m eDP-1); do
    bspc node $wid -m HDMI-A-1
done
```

## Best practices

1. **Màn hình chính** (laptop) dùng workspace I-V.
2. **Màn hình phụ** dùng workspace VI-IX.
3. **Mỗi màn hình có bar riêng** (Polybar).
4. **Di chuyển cửa sổ** bằng `Super + Shift + m`.
5. **autorandr** để tự động chuyển đổi khi cắm/rút màn hình.

## Troubleshooting

### Màn hình mới không được nhận

```bash
# Quét lại màn hình
xrandr --auto

# Kiểm tra
xrandr --query
```

### Cửa sổ mất sau khi chuyển monitor

```bash
# Tìm và focus
bspc node -f any
```

### Polybar không hiển thị trên monitor phụ

- Kiểm tra cấu hình multi-monitor trong bspwmrc.
- Set biến MONITOR trước khi chạy Polybar.

### Workspace không hiển thị đúng trên Polybar

- Mỗi monitor cần instance Polybar riêng.

## Tổng kết

- bspwm hỗ trợ multi-monitor qua bspc monitor.
- Mỗi monitor có workspace riêng.
- Di chuyển cửa sổ, workspace, focus giữa các monitor.
- Polybar có thể chạy instance riêng cho mỗi monitor.
- autorandr tự động cấu hình khi cắm/rút monitor.
