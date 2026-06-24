# Yay — AUR Helper

## Mục tiêu

Cài đặt và sử dụng yay để truy cập Arch User Repository (AUR).

## Kiến thức nền

### AUR là gì?

Arch User Repository (AUR) là kho lưu trữ cộng đồng do người dùng Arch đóng góp.
Nó chứa các `PKGBUILD` — script hướng dẫn cách build gói từ source.

Khi bạn "cài từ AUR", thực chất bạn đang tải PKGBUILD, kiểm tra, build,
và cài bằng `makepkg`.

### Tại sao cần AUR helper?

AUR helper tự động hóa quy trình:
1. Tìm kiếm gói trên AUR.
2. Clone PKGBUILD.
3. Kiểm tra (tùy chọn).
4. Cài dependencies.
5. Build và cài.

## yay — Yet Another Yogurt

yay là AUR helper phổ biến nhất với cú pháp tương tự pacman.

### yay vs yay-bin

| Gói | Cách cài | Thời gian build | Dùng cho |
|---|---|---|---|
| `yay` | Build từ source (git clone + makepkg) | 5–10 phút | Người muốn tối ưu cho CPU của mình |
| `yay-bin` | Pre-compiled binary từ AUR | Tải về, không build | Người muốn cài nhanh |

Cả hai đều từ AUR. `yay-bin` là bản binary có sẵn, không cần build.

### Cài yay (build từ source)

```bash
pacman -S --needed base-devel git
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

Giải thích:
- `git clone`: Tải PKGBUILD của yay từ AUR.
- `makepkg -si`: Build gói (`-s`: tự động cài dependencies, `-i`: cài gói).

### Cài yay-bin (pre-compiled — nhanh hơn)

```bash
pacman -S --needed base-devel git
git clone https://aur.archlinux.org/yay-bin.git
cd yay-bin
makepkg -si
```

Chỉ khác ở tên repo. `yay-bin` không cần build từ source C, chỉ giải nén binary có sẵn.

### Xác nhận cài đặt

```bash
yay --version
```

## Các lệnh yay thường dùng

```bash
# Cập nhật tất cả (pacman + AUR)
yay -Syu

# Cài gói (tự động tìm AUR hoặc chính thức)
yay -S gói

# Cài gói từ AUR (không tìm trong chính thức)
yay -Sa gói

# Tìm kiếm gói (AUR + chính thức)
yay -Ss từ_khóa

# Chỉ tìm trong AUR
yay -Ssa từ_khóa

# Xóa gói
yay -Rns gói

# Xóa cache và file build
yay -Sc

# Xóa orphan packages (AUR)
yay -Yc
```

## So sánh yay -Syu với pacman -Syu

| Lệnh | Cập nhật official | Cập nhật AUR |
|---|---|---|
| `pacman -Syu` | Có | Không |
| `yay -Syu` | Có (gọi pacman) | Có |

`yay -Syu` là lệnh duy nhất bạn cần để cập nhật toàn bộ hệ thống.

## Cấu hình yay

```bash
yay --save --answerclean None --answerdiff None
```

| Option | Mô tả |
|---|---|
| `--answerclean None` | Không hỏi về xóa file build |
| `--answerdiff None` | Không hỏi về diff PKGBUILD |
| `--answeredit None` | Không hỏi về edit PKGBUILD |
| `--answerupgrade None` | Không hỏi về upgrade AUR |

Cẩn thận: Bỏ qua diff PKGBUILD có thể bỏ lỡ code độc.

## Bảo mật AUR

### Rủi ro

- Gói AUR do cộng đồng đóng góp, **không được kiểm duyệt chính thức**.
- PKGBUILD có thể chứa mã độc.

### Cách giảm rủi ro

1. **Chỉ cài gói phổ biến** (nhiều vote, nhiều comment).
2. **Đọc PKGBUILD** trước khi cài lần đầu.
3. **Kiểm tra maintainer**: người có uy tín, nhiều gói.
4. **Đọc comment** trên trang AUR của gói.
5. **Dùng `yay -Sa`** để cài AUR riêng thay vì `-S` (tránh nhầm AUR với official).

## Troubleshooting

### Build fail vì thiếu dependencies

```bash
yay -S gói --makepkg-opt=-s
```

Hoặc cài thủ công dependency bị thiếu.

### "Could not find all required packages"

Một số dependencies chỉ có trong AUR → phải cài từng cái.

### "fatal: could not create work tree dir"

```bash
pacman -S git
```

### "PKGBUILD not found"

Gói AUR không tồn tại → kiểm tra trên https://aur.archlinux.org.

## Best practices

1. **Thường xuyên update**: `yay -Syu` hàng tuần.
2. **Đọc PKGBUILD** của gói AUR lạ trước khi cài.
3. **Dùng yay-bin** nếu muốn cài nhanh, yay nếu muốn tối ưu.
4. **Không dùng yay làm root** (yay tự dùng sudo khi cần).
5. **Dọn cache**: `yay -Sc` định kỳ.
6. **Hạn chế gói AUR**: càng ít gói AUR càng ít rủi ro.

## Tổng kết

- yay là AUR helper phổ biến, cú pháp giống pacman.
- `yay-bin` (pre-compiled) nhanh hơn `yay` (build từ source).
- `yay -Syu` cập nhật cả official lẫn AUR.
- Cần thận trọng với bảo mật khi dùng AUR.
- Dùng `yay -Sa` để chỉ cài từ AUR.
