# Tìm kiếm gói

## Mục tiêu

Biết cách tìm kiếm gói trong repository chính thức và AUR.

## Tìm trong pacman (chính thức)

### Tìm gói chưa cài

```bash
# Tìm theo tên hoặc mô tả
pacman -Ss "từ khóa"

# Ví dụ
pacman -Ss "file manager"
pacman -Ss "terminal emulator"
```

Output:

```
extra/pcmanfm 1.3.2-2
    Extremely fast and lightweight file manager
community/ranger 1.9.3-7
    Simple, vim-like file manager
```

### Tìm gói đã cài

```bash
# Tìm trong gói đã cài
pacman -Qs "từ khóa"

# Liệt kê tất cả gói đã cài
pacman -Q

# Liệt kê gói đã cài (dạng danh sách)
pacman -Qq
```

### Tìm gói sở hữu file

```bash
# File nào thuộc gói nào
pacman -Qo /usr/bin/firefox

# Tìm gói sở hữu file (từ database)
pacman -F /path/to/file
```

### Xem thông tin chi tiết

```bash
# Gói chưa cài
pacman -Si firefox

# Gói đã cài
pacman -Qi firefox
```

## Tìm trong AUR

### Dùng yay

```bash
# Tìm trong AUR + chính thức
yay -Ss "từ khóa"

# Chỉ tìm trong AUR
yay -Ssa "từ khóa"
```

Output:

```
aur/rtl8852be-dkms 1.1.0-1 (+120 3.20)
    Driver for Realtek RTL8852BE wireless chip
```

### Xem thông tin AUR

```bash
# Thông tin chi tiết
yay -Si rtl8852be-dkms

# Hoặc mở trang AUR trên browser
yay -Sii rtl8852be-dkms
```

## Tìm trên web

### Trang AUR

https://aur.archlinux.org/

Có thể search trực tiếp trên web, xem số vote, maintainer, comments.

### Arch Package Search

https://archlinux.org/packages/

Tìm gói trong repository chính thức với filter.

## Cách tìm gói hiệu quả

### 1. Biết tên đúng

- `firefox` → tìm "firefox" hoặc "web browser".
- `alacritty` → tìm "terminal".
- `pcmanfm` → tìm "file manager".

### 2. Dùng grep

```bash
# Tìm gói có từ khóa trong mô tả
pacman -Ss "" | grep -i "image viewer"

# Hoặc tìm chính xác
pacman -Ss "^alacritty$"
```

### 3. Tìm theo category

Arch không có category chính thức. Dùng từ khóa mô tả:

```bash
pacman -Ss "audio"
pacman -Ss "video"
pacman -Ss "editor"
pacman -Ss "browser"
```

### 4. Xem gói trong group

```bash
# Các group có sẵn
pacman -Sg

# Gói trong group
pacman -Sg base-devel
```

## Tìm file thiếu

Khi chạy lệnh báo "command not found":

```bash
# Cài pkgfile
pacman -S pkgfile

# Cập nhật database
pkgfile -u

# Tìm gói chứa lệnh
pkgfile alacritty
```

## Tìm phiên bản cũ

Nếu cần phiên bản cũ của gói (do bug ở bản mới):

```bash
# Xem cache
ls /var/cache/pacman/pkg/ | grep firefox

# Cài từ cache
pacman -U /var/cache/pacman/pkg/firefox-xxx.pkg.tar.zst
```

## Ví dụ thực tế

### Tìm terminal emulator

```bash
pacman -Ss "terminal emulator" | head -20
```

Kết quả: alacritty, kitty, xterm, urxvt, st, termite, ...

### Tìm image viewer

```bash
pacman -Ss "image viewer" | head -10
```

Kết quả: feh, sxiv, viewnior, gimp, ...

### Tìm trash bin s menu

```bash
# Tìm ứng dụng quản lý thùng rác có GUI
yay -Ss "trash"

# Kết quả: trash-cli, trash-d, ...
```

## Tổng kết

- `pacman -Ss` tìm trong repo chính thức.
- `yay -Ss` tìm cả AUR.
- `pacman -Qo` tìm gói sở hữu file.
- `pkgfile` tìm lệnh trong gói chưa cài.
- Tìm trên AUR web để xem vote và comments.
