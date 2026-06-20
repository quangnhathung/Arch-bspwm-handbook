# Fonts

## Mục tiêu

Cài đặt font chữ cho terminal, UI desktop, và hỗ trợ hiển thị tiếng Việt đầy đủ.

## Kiến thức nền

### Các loại font trên Linux

- **Bitmap font**: Font dạng pixel, cũ, ít dùng.
- **TrueType (.ttf)**: Font vector phổ biến nhất, hỗ trợ hinting.
- **OpenType (.otf)**: Kế thừa TrueType, hỗ trợ nhiều tính năng typography hơn.

### Font chữ Việt Nam

Font Unicode (thường là .ttf) có đầy đủ bảng mã UTF-8 bao gồm các ký tự có dấu
(ă, â, ê, ô, ơ, ư, đ, v.v.). Hầu hết font hiện đại đều có sẵn, nhưng cần
chọn font hỗ trợ **Latin Extended** để hiển thị đúng.

### Fallback Font

Khi một font không có ký tự cần hiển thị, fontconfig tự động dùng font khác
(fallback) có ký tự đó. Cấu hình fallback rất quan trọng để không bị lỗi ô vuông
(□) khi gặp ký tự đặc biệt hoặc emoji.

## Các bước thực hiện

### Bước 1: Cài đặt font packages

```bash
pacman -S \
  ttf-dejavu \
  noto-fonts \
  ttf-liberation \
  ttf-jetbrains-mono \
  noto-fonts-cjk \
  noto-fonts-emoji \
  ttf-nerd-fonts-symbols-mono
```

Giải thích từng font:

| Gói | Mục đích |
|---|---|
| `ttf-dejavu` | Font mặc định cho terminal/UI, hỗ trợ Latin Extended |
| `noto-fonts` | Font đa ngôn ngữ của Google, phủ hầu hết Unicode |
| `ttf-liberation` | Tương thích với Arial/Times/ Courier, dùng cho văn phòng |
| `ttf-jetbrains-mono` | Font monospace cho coding, có ràng buộc (ligatures) |
| `noto-fonts-cjk` | Hỗ trợ tiếng Trung/Nhật/Hàn (gián tiếp hỗ trợ Hán-Nôm) |
| `noto-fonts-emoji` | Emoji đầy đủ màu sắc |
| `ttf-nerd-fonts-symbols-mono` | Icon symbols cho polybar, rofi, terminal |

### Bước 2: Xem danh sách font đã cài

```bash
# Liệt kê tất cả font
fc-list | less

# Đếm số lượng font
fc-list | wc -l

# Tìm font cụ thể
fc-list | grep -i "JetBrains"

# Xem chi tiết một font
fc-list --format="%{file}\n" | grep "JetBrains"
fc-query /path/to/font.ttf
```

### Bước 3: Kiểm tra font hỗ trợ tiếng Việt

Dùng lệnh sau để kiểm tra xem font có glyph cho ký tự tiếng Việt không:

```bash
# Kiểm tra Noto Sans có hỗ trợ ơ, ư, đ không
echo "Tiếng Việt: ơ ư đ ê ô ă â" | fc-match -a | head -5

# Kiểm tra font mặc định
fc-match
```

Kết quả `fc-match` cho biết font mặc định của hệ thống. Thường là
`DejaVu Sans.ttf` hoặc `NotoSans-Regular.ttf`.

### Bước 4: Cấu hình font fallback (nếu cần)

Tạo file `~/.config/fontconfig/fonts.conf`:

```bash
mkdir -p ~/.config/fontconfig
vim ~/.config/fontconfig/fonts.conf
```

Nội dung:

```xml
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "urn:fontconfig:fonts.dtd">
<fontconfig>

  <!-- Thêm Noto Sans làm fallback cho DejaVu Sans -->
  <alias>
    <family>DejaVu Sans</family>
    <prefer>
      <family>Noto Sans</family>
      <family>Noto Color Emoji</family>
    </prefer>
  </alias>

  <!-- Fallback cho monospace -->
  <alias>
    <family>JetBrains Mono</family>
    <prefer>
      <family>Noto Sans Mono</family>
      <family>Noto Color Emoji</family>
    </prefer>
  </alias>

</fontconfig>
```

Sau đó cập nhật cache font:

```bash
fc-cache -fv
```

### Bước 5: Cài font bổ sung (tùy chọn)

Có thể cài thêm font từ AUR hoặc tải thủ công:

```bash
# Font từ AUR (cần yay)
yay -S ttf-ms-win11-auto     # Microsoft font (Windows tương thích)
yay -S ttf-google-fonts-git  # Toàn bộ Google Fonts

# Font từ file .ttf tải về
mkdir -p ~/.local/share/fonts
cp /path/to/font.ttf ~/.local/share/fonts/
fc-cache -fv
```

### Bước 6: Cấu hình font cho terminal

Các terminal emulator thường dùng font monospace.

Ví dụ với Alacritty (`~/.config/alacritty/alacritty.toml`):

```toml
[font]
size = 11

[font.normal]
family = "JetBrains Mono"
style = "Regular"

[font.bold]
family = "JetBrains Mono"
style = "Bold"

[font.italic]
family = "JetBrains Mono"
style = "Italic"
```

### Bước 7: Cấu hình font cho Polybar

Trong `~/.config/polybar/config.ini`:

```ini
[bar/main]
font-0 = "JetBrains Mono:size=10"
font-1 = "DejaVu Sans:size=10"
font-2 = "Noto Color Emoji:size=10"
font-3 = "Symbols Nerd Font Mono:size=10"
```

### Bước 8: Cấu hình font cho GTK

Dùng `lxappearance` để chọn font cho GTK:

```bash
pacman -S lxappearance
lxappearance
```

Hoặc sửa trực tiếp file `~/.config/gtk-3.0/settings.ini`:

```ini
[Settings]
gtk-font-name=DejaVu Sans 10
```

## Troubleshooting

### Font bị lỗi ô vuông (□) thay vì ký tự

Nguyên nhân: Fallback font không có glyph cho ký tự đó.

Cách xử lý:
1. Cài thêm `noto-fonts-cjk` và `noto-fonts-emoji`.
2. Cấu hình fallback trong `fonts.conf` như Bước 4.

### Font chữ bị rỗ (không anti-alias)

```bash
# Kiểm tra cấu hình anti-aliasing
xrdb -query | grep Xft

# Thêm vào ~/.Xresources nếu chưa có
echo "Xft.antialias: 1" >> ~/.Xresources
echo "Xft.hinting: full" >> ~/.Xresources
xrdb -merge ~/.Xresources
```

### Không tìm thấy font sau khi cài

```bash
fc-cache -fv   # Force rebuild cache
fc-list | grep -i "ten-font"
```

### Font chữ quá nhỏ trên màn hình 1080p

Tăng DPI trong `~/.Xresources`:

```
Xft.dpi: 120
```

Hoặc trong `bspwmrc`:

```bash
xrandr --output eDP-1 --mode 1920x1080 --rate 144 --dpi 120
```

## Tổng kết

- Đã cài đủ font cho terminal, desktop, và tiếng Việt.
- Font monospace (JetBrains Mono) dùng cho coding.
- Font UI (DejaVu Sans, Noto Sans) dùng cho giao diện.
- Font icon (Nerd Font Symbols) dùng cho polybar/rofi.
- Fontconfig cấu hình fallback để tránh lỗi hiển thị.
