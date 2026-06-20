# Picom — Compositor

## Mục tiêu

Cài đặt và cấu hình Picom — compositor cho Xorg, xử lý đổ bóng, chống tearing.

## Kiến thức nền

### Compositor là gì?

Compositor là chương trình quản lý cách các cửa sổ được hiển thị lên màn hình.
Nó thêm các hiệu ứng:

- **Đổ bóng (shadow)**: Bóng đổ phía sau cửa sổ.
- **Trong suốt (transparency/fade)**: Làm mờ cửa sổ không focus.
- **Chống xé hình (vsync)**: Đồng bộ với tốc độ làm tươi màn hình.
- **Animation**: Hiệu ứng chuyển động khi mở/đóng cửa sổ.

### Tại sao cần compositor cho bspwm?

bspwm không có compositor tích hợp. Nếu không có picom (hoặc compositor khác):

- Không có đổ bóng → giao diện phẳng, khó phân biệt cửa sổ.
- Có thể bị **screen tearing** (xé hình) khi di chuyển cửa sổ hay xem video.
- Không có animation → chuyển cảnh khô cứng.

## Các bước thực hiện

### Bước 1: Cài Picom

```bash
pacman -S picom
```

### Bước 2: Tạo thư mục và file config

```bash
su - archuser
mkdir -p ~/.config/picom
exit
```

```bash
vim /home/archuser/.config/picom/picom.conf
```

### Bước 3: File cấu hình picom.conf

```conf
#################################
#         Animations            #
#################################

animations = true;
animation-window-mass = 0.5;
animation-for-open-window = "zoom";
animation-for-workspace-switch-in = "slide-left";
animation-for-workspace-switch-out = "slide-right";

#################################
#          Shadows              #
#################################

shadow = true;
shadow-radius = 12;
shadow-offset-x = -8;
shadow-offset-y = -8;
shadow-opacity = 0.4;
shadow-red = 0.0;
shadow-green = 0.0;
shadow-blue = 0.0;
shadow-exclude = [
    "name = 'Notification'",
    "class_g = 'Polybar'",
    "class_g = 'Rofi'",
    "class_g = 'Dunst'",
    "class_g = 'Nitrogen'",
    "name = 'picom'"
];

#################################
#       Fading                  #
#################################

fading = true;
fade-in-step = 0.03;
fade-out-step = 0.03;
fade-delta = 3;
no-fading-openclose = true;
fade-exclude = [];

#################################
#      Opacity                  #
#################################

inactive-opacity = 0.95;
active-opacity = 1.0;
frame-opacity = 1.0;
inactive-opacity-override = false;
opacity-rule = [
    "90:class_g = 'Alacritty' && focused",
    "80:class_g = 'Alacritty' && !focused"
];

#################################
#      VSync / Tearing          #
#################################

vsync = true;

#################################
#     Window type settings      #
#################################

wintypes:
{
    tooltip = { fade = true; shadow = false; opacity = 0.85; };
    dock = { shadow = false; };
    desktop = { shadow = false; };
    menu = { opacity = 0.95; };
    popup_menu = { opacity = 0.95; };
    dropdown_menu = { opacity = 0.95; };
};

#################################
#      Blur                     #
#################################

blur-method = "none";
blur-size = 12;
blur-deviation = 5;
blur-strength = 5;
blur-background = false;
blur-background-frame = false;
blur-background-fixed = false;
blur-kern = "3x3box";
blur-background-exclude = [];

#################################
#      Miscellaneous            #
#################################

detect-rounded-corners = true;
detect-transient = true;
detect-client-opacity = true;
refresh-rate = 144;
use-damage = true;

# NVIDIA-specific
unredir-if-possible = false;
```

### Bước 4: Cấu hình cho NVIDIA

Nếu dùng NVIDIA làm GPU chính, thêm option:

```ini
vsync = true;
unredir-if-possible = false;
```

`unredir-if-possible = false` rất quan trọng với NVIDIA:
Khi true, picom sẽ tắt compositor khi cửa sổ fullscreen → gây tearing.
Đặt false để luôn bật compositor.

### Bước 5: Kích hoạt trong bspwmrc

Trong `bspwmrc` đã có dòng:

```bash
picom --config ~/.config/picom/picom.conf &
```

Nếu chưa có, thêm trước dòng `exec`.

### Bước 6: Kiểm tra

```bash
# Khởi động picom thử
picom --config ~/.config/picom/picom.conf

# Kiểm tra lỗi
picom --config ~/.config/picom/picom.conf --log-file /tmp/picom.log
cat /tmp/picom.log
```

## Performance tuning

### Giảm tải cho GPU

```ini
detect-rounded-corners = true;
refresh-rate = 144;
use-damage = true;
```

### Tắt animation nếu lag

```ini
animations = false;
```

### Tắt shadow cho terminal (tăng performance)

```ini
shadow-exclude = [
    "class_g = 'Alacritty'"
];
```

## Troubleshooting

### Screen tearing vẫn còn

```ini
vsync = true;
unredir-if-possible = false;
```

Nếu vẫn bị tearing, thử backend khác:

```bash
picom --backend glx --config ~/.config/picom/picom.conf
```

### Picom làm chậm ứng dụng

Thử backend xrender thay vì glx:

```ini
backend = "xrender";
```

### Picom crash

- Kiểm tra log: `cat /tmp/picom.log`.
- Tắt animation và blur.
- Dùng backend cũ hơn.

## Tổng kết

- Picom đã được cài với cấu hình có shadow, fade, animation.
- VSync được bật để chống tearing.
- Cấu hình NVIDIA-specific để tránh lỗi fullscreen.
- Picom tự động chạy cùng bspwm.
