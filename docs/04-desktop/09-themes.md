# Themes — Giao diện

## Mục tiêu

Cài đặt theme cho GTK, icon, và cursor để có giao diện đồng bộ.

## Kiến thức nền

### Themes trong Linux

- **GTK Theme**: Giao diện của ứng dụng GTK (Firefox, Thunar, v.v.).
- **Icon Theme**: Bộ icon cho ứng dụng và hệ thống.
- **Cursor Theme**: Hình dạng con trỏ chuột.
- **Qt Theme**: Giao diện ứng dụng Qt (có thể dùng chung GTK theme qua qt5ct/qt6ct).

### Mặc định

Arch không có theme mặc định. Ứng dụng GTK sẽ hiển thị theme mặc định xấu.
Cần cài theme riêng.

## Các bước thực hiện

### Bước 1: Cài GTK theme

```bash
pacman -S arc-gtk-theme materia-gtk-theme nordic-theme
```

| Theme | Đặc điểm |
|---|---|
| Arc | Phổ biến, phẳng, đẹp |
| Materia | Material Design, tối |
| Nordic | Bảng màu nord, tối, dễ chịu |

Chọn **Nordic** vì hợp với Dracula/Nord scheme của bspwm-config ở trên.

### Bước 2: Cài icon theme

```bash
pacman -S papirus-icon-theme adwaita-icon-theme
```

Papirus là icon theme đẹp, hiện đại, hỗ trợ folder màu.

### Bước 3: Cài cursor theme

```bash
pacman -S capitaine-cursors bibata-cursor-theme
```

### Bước 4: Cấu hình GTK theme

#### Cấu hình cho GTK3

```bash
su - archuser
mkdir -p ~/.config/gtk-3.0
exit
```

```bash
vim /home/archuser/.config/gtk-3.0/settings.ini
```

Nội dung:

```ini
[Settings]
gtk-theme-name=Nordic
gtk-icon-theme-name=Papirus-Dark
gtk-cursor-theme-name=Bibata-Modern-Ice
gtk-font-name=Noto Sans 10
gtk-application-prefer-dark-theme=1
```

#### Cấu hình cho GTK4

```bash
mkdir -p /home/archuser/.config/gtk-4.0
```

```bash
vim /home/archuser/.config/gtk-4.0/settings.ini
```

Nội dung giống GTK3.

### Bước 5: Cấu hình Qt theme (nếu cần)

Nếu có ứng dụng Qt (như qBittorrent, v.v.):

```bash
pacman -S qt5ct qt6ct
```

```bash
vim /etc/environment
```

Thêm:

```
QT_QPA_PLATFORMTHEME=qt5ct
```

Sau đó chạy `qt5ct` để chọn theme.

Hoặc dùng kvantum:

```bash
pacman -S kvantum
```

### Bước 6: Cấu hình cursor theme toàn hệ thống

```bash
vim /etc/environment
```

Thêm:

```
XCURSOR_THEME=Bibata-Modern-Ice
XCURSOR_SIZE=24
```

### Bước 7: Cài theme cho các ứng dụng cụ thể

#### Alacritty

```bash
vim /home/archuser/.config/alacritty/alacritty.yml
```

Thêm colorscheme (ví dụ Dracula):

```yaml
colors:
  primary:
    background: '#282A36'
    foreground: '#F8F8F2'
  normal:
    black:   '#21222C'
    red:     '#FF5555'
    green:   '#50FA7B'
    yellow:  '#FFB86C'
    blue:    '#BD93F9'
    magenta: '#FF79C6'
    cyan:    '#8BE9FD'
    white:   '#F8F8F2'
  bright:
    black:   '#6272A4'
    red:     '#FF6E6E'
    green:   '#69FF94'
    yellow:  '#FFCA80'
    blue:    '#CAA9FA'
    magenta: '#FF92D0'
    cyan:    '#A4FFFF'
    white:   '#FFFFFF'
```

#### Rofi

Trong `config.rasi` đã set theme:

```
@theme "/usr/share/rofi/themes/nord.rasi"
```

## Công cụ quản lý theme

### Lxappearance

Công cụ GUI để chọn GTK theme, icon, cursor:

```bash
pacman -S lxappearance
```

Chạy `lxappearance` sau khi có X → chọn theme từ dropdown.

### Cập nhật theme sau khi thay đổi

Sau khi sửa file cấu hình, cần logout và login lại X để thấy thay đổi.

```bash
# Hoặc dùng lệnh để apply ngay (không cần logout)
gsettings set org.gnome.desktop.interface gtk-theme Nordic
```

## Danh sách theme khuyên dùng

| Loại | Theme khuyên dùng |
|---|---|
| GTK | Nordic |
| Icon | Papirus-Dark |
| Cursor | Bibata-Modern-Ice |
| Terminal | Dracula (Alacritty) |
| Rofi | Nord |
| Polybar | Dracula (tự cấu hình) |

## Troubleshooting

### Theme không apply

- Kiểm tra file `~/.config/gtk-3.0/settings.ini` tồn tại.
- Kiểm tra theme đã được cài: `ls /usr/share/themes/`.
- Chạy `lxappearance` và chọn lại theme.

### Icon không hiển thị

- Kiểm tra `ls /usr/share/icons/`.
- Papirus phải được cài.

### Cursor không đổi

- Kiểm tra `ls /usr/share/icons/`.
- Thêm `XCURSOR_THEME` vào `/etc/environment`.

## Tổng kết

- GTK theme Nordic cho giao diện tối, dễ chịu.
- Icon Papirus-Dark cho icon đồng bộ.
- Cursor Bibata hiện đại.
- Cấu hình cho GTK3, GTK4, Qt, và các ứng dụng cụ thể.
