# Tìm kiếm gói

## Mục tiêu

Biết cách tìm kiếm gói trong repository chính thức và AUR hiệu quả.

## Tìm trong pacman (repository chính thức)

### Tìm gói chưa cài (`-Ss`)

```bash
pacman -Ss "từ khóa"
```

Ví dụ:

```bash
pacman -Ss "file manager"
pacman -Ss "terminal emulator"
pacman -Ss firefox
```

Output:

```
extra/pcmanfm 1.3.2-2
    Extremely fast and lightweight file manager
extra/firefox 130.0-1
    Fast, Private & Safe Web Browser
```

Repository được hiển thị: `core/`, `extra/`, `community/`.

### Tìm gói đã cài (`-Qs`)

```bash
pacman -Qs "từ khóa"
```

### Liệt kê gói đã cài

```bash
# Đầy đủ thông tin
pacman -Q

# Chỉ tên gói
pacman -Qq

# Đếm số lượng
pacman -Qq | wc -l
```

## Tìm gói sở hữu file

### pacman -Qo (gói đã cài)

```bash
# File nào thuộc gói nào?
pacman -Qo /usr/bin/firefox

# Output: /usr/bin/firefox is owned by firefox 130.0-1
```

### pacman -F (gói chưa cài)

```bash
# Cập nhật database -F
pacman -Fy

# Tìm gói nào sở hữu file (dù chưa cài)
pacman -F /usr/include/stdio.h
```

## pkgfile — Tìm lệnh "command not found"

Khi gõ lệnh và nhận "command not found", `pkgfile` cho biết gói nào chứa lệnh đó.

```bash
# Cài pkgfile
pacman -S pkgfile

# Cập nhật database
pkgfile -u

# Tìm gói chứa lệnh
pkgfile alacritty

# Output: extra/alacritty
```

### Tích hợp pkgfile với shell

Thêm vào `~/.bashrc` hoặc `~/.zshrc`:

```bash
source /usr/share/doc/pkgfile/command-not-found.bash
```

Khi gõ lệnh không tìm thấy, shell sẽ tự gợi ý gói cần cài.

## Tìm trong AUR

### yay -Ss (AUR + chính thức)

```bash
yay -Ss "từ khóa"
```

Tìm cả trong pacman và AUR, kết quả hiển thị chung.

### yay -Ssa (chỉ AUR)

```bash
yay -Ssa "từ khóa"
```

Chỉ tìm trong AUR, bỏ qua official.

Output:

```
aur/visual-studio-code-bin 1.96.0-1 (+2450 3.45)
    Visual Studio Code (binary release)
```

Dấu hiệu nhận biết: `aur/` ở đầu tên gói.

### Xem thông tin gói AUR

```bash
# Thông tin chi tiết
yay -Si tên-gói-aur

# Mở trang AUR trên trình duyệt
yay -Sii tên-gói-aur
```

## Tìm trên web

### Arch Package Search

https://archlinux.org/packages/

Tìm gói chính thức với filter theo repository, kiến trúc.

### AUR Website

https://aur.archlinux.org/

Tìm gói AUR, xem số vote, maintainer, comments.
Comments rất hữu ích — người dùng thường báo lỗi hoặc cách fix.

## Mẹo tìm kiếm hiệu quả

### 1. Dùng từ khóa mô tả

Arch đặt tên gói theo chức năng, không theo tên thương hiệu:

| Bạn muốn | Tìm |
|---|---|
| Terminal | "terminal emulator" |
| File manager | "file manager" |
| Image viewer | "image viewer" |
| Text editor | "text editor" |
| PDF reader | "pdf viewer" |

### 2. Kết hợp grep

```bash
# Lọc kết quả
pacman -Ss "" | grep -i "image viewer"

# Chỉ lấy tên gói
pacman -Ss "" | grep -i "^extra/" | head -20
```

### 3. Dùng regex

```bash
# Tìm chính xác
pacman -Ss "^alacritty$"

# Tìm theo pattern
pacman -Ss "^kde-"
```

### 4. Xem group packages

```bash
# Danh sách group
pacman -Sg

# Gói trong group
pacman -Sg base-devel
pacman -Sg xfce4
```

### 5. Tìm gói tương tự

```bash
# Sau khi tìm được một gói, xem "Required By"
pacman -Qi firefox | grep "Required By"
```

## Tìm phiên bản cũ

### Trong cache

```bash
ls /var/cache/pacman/pkg/ | grep firefox

# Cài từ cache
pacman -U /var/cache/pacman/pkg/firefox-xxx.pkg.tar.zst
```

### Trên Arch Archive

https://archive.archlinux.org/

## Tổng kết

- `pacman -Ss` → tìm trong repository chính thức.
- `yay -Ss` → tìm cả official lẫn AUR.
- `yay -Ssa` → chỉ tìm trong AUR.
- `pacman -Qo` → gói nào sở hữu file (đã cài).
- `pacman -F` → gói nào sở hữu file (chưa cài).
- `pkgfile` → tìm lệnh "command not found".
- Web: https://archlinux.org/packages/ và https://aur.archlinux.org/.
