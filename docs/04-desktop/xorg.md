# Xorg Display Server

## Mục tiêu

Cài đặt và cấu hình Xorg — display server cho môi trường desktop.

## Kiến thức nền

### Display Server là gì?

Display server là chương trình trung gian giữa ứng dụng đồ họa và phần cứng.
Nó nhận dữ liệu từ ứng dụng và gửi đến GPU để hiển thị.

```
Application → Display Server → GPU → Màn hình
```

### Xorg vs Wayland

| | Xorg | Wayland |
|---|---|---|
| Đời | Cũ (1987) | Mới (2015+) |
| Kiến trúc | Client-Server | Compositor |
| Tương thích | Tốt với hầu hết ứng dụng | Đang cải thiện |
| NVIDIA | Cần cấu hình | Chưa ổn định |
| Security | Kém (app có thể đọc màn hình khác) | Tốt hơn |

Chúng ta dùng **Xorg** vì:
- Tương thích tốt với NVIDIA.
- Tương thích tốt với bspwm (Wayland chưa support bspwm).
- Ổn định, lâu đời.

### Xinit là gì?

`xinit` là chương trình khởi động X server. Nó đọc `~/.xinitrc` để biết chạy
chương trình gì sau khi X server khởi động.

Với bspwm, `~/.xinitrc` sẽ chứa:

```bash
exec bspwm
```

## Các bước thực hiện

### Bước 1: Cài Xorg

```bash
pacman -S xorg xorg-server xorg-init xorg-xrandr
```

| Gói | Vai trò |
|---|---|
| `xorg` | Nhóm gói Xorg (ký hiệu, font, v.v.) |
| `xorg-server` | X server (phần lõi) |
| `xorg-init` | Script khởi động (xinit, startx) |
| `xorg-xrandr` | Công cụ quản lý màn hình (resolution, multi-monitor) |

### Bước 2: Cài Xorg input drivers

```bash
pacman -S xf86-input-libinput
```

`xf86-input-libinput` là driver input mới nhất, hỗ trợ tốt touchpad,
bàn phím, chuột trên máy hiện đại.

### Bước 3: Cài Xorg GPU drivers (cơ bản)

```bash
pacman -S mesa libgl
```

- `mesa`: OpenGL implementation mã nguồn mở (dùng cho Intel GPU).
- `libgl`: Thư viện OpenGL.

Driver NVIDIA và Intel sâu hơn được xử lý riêng trong docs/05-drivers/.

### Bước 4: Tạo file ~/.xinitrc

```bash
su - archuser
mkdir -p ~/.config
echo "exec bspwm" > ~/.xinitrc
exit
```

Giải thích:
- `su - archuser`: Chuyển sang user (vì đang ở chroot với root).
- `echo "exec bspwm"`: Tạo file .xinitrc đơn giản nhất.

### Bước 5: Kiểm tra Xorg

Trong chroot không thể test Xorg (cần GPU). Sau reboot:

```bash
# Kiểm tra Xorg có cài đúng không
Xorg -version

# Thử khởi động Xorg (nếu có GPU)
startx
```

## Cấu hình Xorg nâng cao (khi cần)

### Tạo file config cho touchpad

```bash
mkdir -p /etc/X11/xorg.conf.d
vim /etc/X11/xorg.conf.d/30-touchpad.conf
```

Nội dung:

```
Section "InputClass"
    Identifier "touchpad"
    Driver "libinput"
    MatchIsTouchpad "on"
    Option "Tapping" "on"           # Tap để click
    Option "TappingButtonMap" "lrm" # 1 ngón = left, 2 = right, 3 = middle
    Option "NaturalScrolling" "on"  # Cuộn tự nhiên (giống macOS)
    Option "DisableWhileTyping" "on"
EndSection
```

### Tắt chuột tăng tốc (mouse acceleration)

```bash
vim /etc/X11/xorg.conf.d/50-mouse-acceleration.conf
```

```
Section "InputClass"
    Identifier "mouse"
    Driver "libinput"
    MatchIsPointer "on"
    Option "AccelProfile" "flat"
EndSection
```

## Troubleshooting

### "Failed to run /usr/bin/xinit"

Chưa cài xorg-init → `pacman -S xorg-init`.

### "/usr/bin/xinit: No such file or directory"

Chưa cài xorg-init.

### "Cannot open /dev/tty0"

Chạy startx từ non-root user.

## Tổng kết

- Xorg đã được cài với driver input libinput.
- `~/.xinitrc` đã được tạo.
- Sau khi cài bspwm và reboot, `startx` sẽ khởi động bspwm.
