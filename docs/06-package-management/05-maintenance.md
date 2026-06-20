# Yay — AUR Helper

## Mục tiêu

Cài đặt và sử dụng yay để truy cập Arch User Repository (AUR).

## Kiến thức nền

### AUR là gì?

Arch User Repository (AUR) là kho lưu trữ cộng đồng do người dùng Arch đóng góp.
Nó chứa các gói không có trong repository chính thức.

AUR chứa các `PKGBUILD` — script hướng dẫn cách build gói từ source.
Khi bạn "cài từ AUR", thực chất bạn đang tải PKGBUILD, kiểm tra, build,
và cài bằng makepkg.

### Tại sao cần AUR helper?

AUR helper tự động hóa quy trình:
1. Tìm kiếm gói trên AUR.
2. Clone PKGBUILD.
3. Kiểm tra (tùy chọn).
4. Cài dependencies.
5. Build và cài.

### yay

yay (Yet Another Yogurt) là AUR helper phổ biến nhất.
Cú pháp tương tự pacman:

```bash
yay -S gói       # Cài từ AUR hoặc chính thức
yay -Syu         # Cập nhật tất cả (AUR + chính thức)
yay -Ss từ_khóa  # Tìm trong AUR + chính thức
```

### Lưu ý về bảo mật AUR

Gói AUR do cộng đồng đóng góp, không được kiểm duyệt chính thức.
**Rủi ro**: Mã độc có thể được đưa vào PKGBUILD.

An toàn:
- Chỉ cài gói AUR phổ biến (nhiều vote).
- Đọc PKGBUILD trước khi build.
- Tin tưởng maintainer có uy tín.

## Các bước thực hiện

### Bước 1: Cài yay từ AUR

```bash
pacman -S --needed base-devel git
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

Giải thích:
- `git clone`: Tải PKGBUILD của yay từ AUR.
- `makepkg -si`: Build gói (`-s`: tự động cài dependencies, `-i`: cài gói).

### Bước 2: Xác nhận cài đặt

```bash
yay --version
```

### Bước 3: Cấu hình yay (tùy chọn)

```bash
yay --save --answerclean None --answerdiff None
```

Các option:

| Option | Mô tả |
|---|---|
| `--answerclean None` | Không hỏi về xóa file build |
| `--answerdiff None` | Không hỏi về diff PKGBUILD |
| `--answeredit None` | Không hỏi về edit PKGBUILD |
| `--answerupgrade None` | Không hỏi về upgrade AUR |

Cẩn thận: Không xem diff có thể bỏ lỡ code độc. Chỉ dùng các option này
khi đã quen.

## Các lệnh yay thường dùng

```bash
# Cập nhật tất cả (pacman + AUR)
yay -Syu

# Cài gói (tự động tìm trong AUR hoặc chính thức)
yay -S gói

# Cài gói từ AUR (không tìm trong chính thức)
yay -Sa gói

# Tìm kiếm gói
yay -Ss từ_khóa

# Tìm trong AUR
yay -Ssa từ_khóa

# Xóa gói
yay -Rns gói

# Xóa cache và file build
yay -Sc
```

## Khi nào dùng pacman vs yay

| Tình huống | Dùng |
|---|---|
| Gói có trong repo chính thức | `pacman -S` |
| Gói chỉ có trong AUR | `yay -S` |
| Cập nhật toàn bộ hệ thống | `yay -Syu` |
| Tìm gói | `yay -Ss` (tìm cả 2) |
| Xóa gói | `pacman -Rns` hoặc `yay -Rns` |

## Troubleshooting

### "fatal: could not create work tree dir"

Cần git: `pacman -S git`.

### "PKGBUILD not found"

Gói AUR không tồn tại hoặc tên sai → kiểm tra trên https://aur.archlinux.org.

### Build fail vì thiếu dependencies

```bash
# yay tự động cài dependencies, nhưng nếu lỗi:
yay -S gói --makepkg-opt=-s
```

### "Could not find all required packages"

Một số dependencies chỉ có trong AUR → phải cài từng cái.

## Best practices

1. **Thường xuyên update**: `yay -Syu` hàng tuần.
2. **Đọc PKGBUILD** của gói AUR trước khi cài lần đầu.
3. **Không dùng yay làm root** (yay tự động dùng sudo khi cần).
4. **Dọn cache**: `yay -Sc` định kỳ.
5. **Kiểm tra orphan**: `yay -Yc`.

## Tổng kết

- yay là AUR helper phổ biến, cú pháp giống pacman.
- Cài từ source thông qua AUR.
- Cập nhật đồng thời pacman + AUR với `yay -Syu`.
- Cần thận trọng với bảo mật khi dùng AUR.
