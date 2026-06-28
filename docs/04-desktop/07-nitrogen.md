# Nitrogen — Wallpaper

Ngày cập nhật: 28/06/2026

## Mục tiêu

Cài đặt Nitrogen để quản lý hình nền (wallpaper).

## Kiến thức nền

### Nitrogen là gì?

**Lưu ý:** Trên máy hiện tại, **Nitrogen không được dùng**. Wallpaper được quản lý
bởi script `~/.local/bin/wallpaper.sh` (dùng `feh`), hỗ trợ prev/next/random.
Bài này giữ lại để tham khảo cho ai muốn dùng Nitrogen.

Nitrogen là chương trình quản lý wallpaper cho X11. Nó hỗ trợ:

- Đặt hình nền từ thư mục.
- Tự động restore wallpaper sau khi reboot.
- Hỗ trợ multi-monitor.
- Các chế độ: center, zoom, fill, fit, span, tile.

### Nitrogen vs feh vs hsetroot

| Công cụ | Loại | Đặc điểm |
|---|---|---|
| Nitrogen | GUI | Có giao diện để chọn wallpaper |
| feh | CLI | Không GUI, nhẹ, dùng được cho slideshow |
| hsetroot | CLI | Nhẹ nhất, chỉ set màu nền |

Dùng Nitrogen vì dễ chọn wallpaper, lưu cấu hình tự động.
Nếu bạn thích CLI, dùng feh (lighter).

## Các bước thực hiện

### Bước 1: Cài Nitrogen

```bash
pacman -S nitrogen
```

### Bước 2: Tạo thư mục chứa wallpaper

```bash
su - archuser
mkdir -p ~/Pictures/wallpapers
exit
```

### Bước 3: Copy wallpaper vào thư mục

```bash
# Ví dụ tải wallpaper từ internet (trong user session)
wget -O ~/Pictures/wallpapers/default.jpg https://example.com/wallpaper.jpg
```

### Bước 4: Cấu hình Nitrogen lần đầu

Mở Nitrogen (sau khi có X):

```bash
nitrogen ~/Pictures/wallpapers/
```

Hoặc chạy từ Rofi → tìm "Nitrogen".

Trong Nitrogen:
1. **Preferences** → Add thư mục `~/Pictures/wallpapers`.
2. Chọn wallpaper.
3. Chọn chế độ: **Scaled** (fit), **Zoomed** (crop), **Centered**.
4. Nhấn **Apply**.

Nitrogen sẽ tự động tạo file `~/.config/nitrogen/bg-saved.cfg`
và `~/.config/nitrogen/nitrogen.cfg`.

### Bước 5: Restore wallpaper khi khởi động

Trong `bspwmrc`, đã có dòng:

```bash
nitrogen --restore &
```

Dòng này đọc file config và đặt lại wallpaper như lần cuối.

### Bước 6: Kiểm tra

```bash
# Khởi động nitrogen để chọn wallpaper
nitrogen ~/Pictures/wallpapers/

# Restore
nitrogen --restore
```

## File cấu hình

### bg-saved.cfg

```ini
[:0.0]
file=/home/archuser/Pictures/wallpapers/default.jpg
mode=4
bgcolor=#000000
```

- `:0.0`: Monitor (có thể thay đổi với multi-monitor).
- `file`: Đường dẫn wallpaper.
- `mode`: 4 = zoomed crop.
- `bgcolor`: Màu nền (nếu ảnh không fill hết).

### nitrogen.cfg

```ini
[geometry]
posx=0
posy=0
sizex=800
sizey=600

[nitrogen]
view=icon
recurse=true
sort=alpha
icon_caps=false
dirs=/home/archuser/Pictures/wallpapers;
```

## Chế độ hiển thị

| Mode | Giá trị | Mô tả |
|---|---|---|
| Centered | 1 | Ảnh ở giữa, không scale |
| Scaled | 2 | Scale vừa màn hình (có thể méo) |
| Stretched | 3 | Kéo giãn fill màn hình |
| Zoomed | 4 | Scale giữ tỉ lệ, crop phần thừa |
| Tiled | 5 | Lặp ảnh |

## Multi-monitor wallpaper

Với nhiều màn hình, Nitrogen có thể set wallpaper khác nhau cho từng màn hình:

```bash
nitrogen --set-zoom-fill --head=0 ~/Pictures/wallpapers/left.jpg
nitrogen --set-zoom-fill --head=1 ~/Pictures/wallpapers/right.jpg
```

Hoặc dùng chế độ `span` để ảnh trải đều các màn hình.

## Không có hình nền (chỉ màu nền)

Nếu chưa có wallpaper hoặc không muốn dùng ảnh:

```bash
nitrogen --set-zoom-fill /path/to/image.jpg
```

Hoặc dùng hsetroot cho đơn giản:

```bash
pacman -S hsetroot
hsetroot -solid "#282A36"
```

## Alternative: feh (lighter, CLI-based)

Nếu bạn không muốn GUI, dùng feh:

```bash
pacman -S feh

# Set wallpaper
feh --bg-fill ~/Pictures/wallpapers/default.jpg

# Trong bspwmrc (thay thế nitrogen --restore)
feh --bg-fill ~/Pictures/wallpapers/default.jpg
```

feh tự động lưu config vào `~/.fehbg` và có thể restore bằng:

```bash
eval $(cat ~/.fehbg)
```

## Troubleshooting

### Nitrogen không restore wallpaper

- Kiểm tra `~/.config/nitrogen/bg-saved.cfg` tồn tại.
- Kiểm tra file wallpaper còn tồn tại.
- Kiểm tra đường dẫn trong file config.

### Nitrogen không tìm thấy thư mục

Thêm thư mục trong Preferences hoặc sửa `nitrogen.cfg`.

## Tổng kết

- Nitrogen đã được cài để quản lý wallpaper.
- Wallpaper tự động restore khi khởi động.
- Hỗ trợ các chế độ hiển thị khác nhau.
- Có thể dùng feh thay thế nếu muốn nhẹ hơn.
