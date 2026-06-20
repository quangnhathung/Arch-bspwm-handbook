# Fonts

## Mục tiêu

Cài đặt font chữ cho terminal, UI, và hiển thị tiếng Việt.

## Kiến thức nền

### Font trong Linux

Linux quản lý font qua thư mục `/usr/share/fonts/` và `~/.local/share/fonts/`.
Mỗi font family là một thư mục chứa file `.ttf` hoặc `.otf`.

### Các loại font

| Loại | Dùng cho | Ví dụ |
|---|---|---|
| Monospace | Terminal, code | FiraCode, JetBrains Mono |
| Sans-serif | UI, văn bản | Noto Sans, Roboto |
| Serif | Văn bản dài | Noto Serif |
| Icon | Polybar, Rofi | Nerd Fonts |

### Font cho tiếng Việt

Font Noto Sans CJK hoặc Noto Sans có hỗ trợ tiếng Việt đầy đủ.
Mặc định Arch không có font đẹp → cần cài thêm.

## Các bước thực hiện

### Bước 1: Cài font monospace

```bash
pacman -S ttf-firacode-nerd ttf-jetbrains-mono-nerd ttf-hack-nerd
```

- `ttf-firacode-nerd`: FiraCode với Nerd Font patches (icon cho terminal).
- `ttf-jetbrains-mono-nerd`: JetBrains Mono (rất đẹp cho code/terminal).
- `ttf-hack-nerd`: Hack font, nhẹ.

### Bước 2: Cài font UI

```bash
pacman -S noto-fonts noto-fonts-emoji noto-fonts-cjk
```

- `noto-fonts`: Font chữ đa ngôn ngữ (hỗ trợ tiếng Việt).
- `noto-fonts-emoji`: Emoji.
- `noto-fonts-cjk`: Chinese/Japanese/Korean (nếu cần).

### Bước 3: Cài font icon cho Polybar

```bash
pacman -S ttf-font-awesome otf-font-awesome
```

Font Awesome cung cấp icon dùng trong Polybar và Rofi.

### Bước 4: Refresh font cache

```bash
fc-cache -fv
```

### Bước 5: Kiểm tra font

```bash
# Liệt kê tất cả font
fc-list | head -20

# Tìm font cụ thể
fc-list | grep -i "FiraCode"

# Kiểm tra font hỗ trợ tiếng Việt
fc-list :lang=vi | head -10
```

### Bước 6: Cấu hình font cho ứng dụng

#### Alacritty

Trong `~/.config/alacritty/alacritty.yml`:

```yaml
font:
  normal:
    family: "FiraCode Nerd Font"
    style: Regular
  bold:
    family: "FiraCode Nerd Font"
    style: Bold
  italic:
    family: "FiraCode Nerd Font"
    style: Italic
  bold_italic:
    family: "FiraCode Nerd Font"
    style: Bold Italic
  size: 11
```

#### Polybar

Trong `~/.config/polybar/config.ini`:

```ini
font-0 = "FiraCode Nerd Font:size=11;3"
font-1 = "Noto Sans:size=10;3"
```

#### Rofi

Trong `~/.config/rofi/config.rasi`:

```ini
font: "FiraCode Nerd Font 12";
```

### Bước 7: Cài thêm font (tùy chọn)

```bash
# Font cho GTK applications
pacman -S adobe-source-code-pro-fonts

# Microsoft fonts (nếu cần)
yay -S ttf-ms-win11-auto
```

## Fallback font

File cấu hình font fallback toàn hệ thống:

```bash
vim /etc/fonts/local.conf
```

```xml
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
  <alias>
    <family>sans-serif</family>
    <prefer>
      <family>Noto Sans</family>
      <family>Noto Sans CJK JP</family>
    </prefer>
  </alias>
  <alias>
    <family>monospace</family>
    <prefer>
      <family>FiraCode Nerd Font</family>
      <family>Noto Sans Mono</family>
    </prefer>
  </alias>
</fontconfig>
```

## Troubleshooting

### Font không hiển thị đúng trong terminal

- Kiểm tra font có hỗ trợ không: `fc-list | grep FiraCode`.
- Cấu hình đúng tên font trong terminal config.

### Font icon hiển thị ô vuông

- Thiếu Nerd Font hoặc Font Awesome.
- Chạy `fc-cache -fv` và restart ứng dụng.

### Tiếng Việt hiển thị ô vuông

- Thiếu Noto Sans hoặc font hỗ trợ tiếng Việt.
- Cài `noto-fonts` và `noto-fonts-cjk`.

## Tổng kết

- Font monospace (FiraCode, JetBrains Mono) cho terminal.
- Font UI (Noto Sans) cho giao diện.
- Font icon (Nerd Font, Font Awesome) cho Polybar.
- Font cache đã được refresh.
- Cấu hình font cho các ứng dụng chính.
