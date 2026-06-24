# Picom — Compositor

Ngày cập nhật: 25/06/2026

## Mục tiêu

Cài đặt và cấu hình Picom — compositor cho Xorg, xử lý đổ bóng, chống tearing.

## Kiến thức nền

### Compositor là gì?

Compositor là chương trình quản lý cách các cửa sổ được hiển thị lên màn hình.
Nó thêm các hiệu ứng:

- **Đổ bóng (shadow)**: Bóng đổ phía sau cửa sổ.
- **Trong suốt (transparency/fade)**: Làm mờ cửa sổ không focus.
- **Chống xé hình (vsync)**: Đồng bộ với tốc độ làm tươi màn hình.

### Tại sao cần compositor cho bspwm?

bspwm không có compositor tích hợp. Nếu không có picom (hoặc compositor khác):

- Không có đổ bóng → giao diện phẳng, khó phân biệt cửa sổ.
- Có thể bị **screen tearing** (xé hình) khi di chuyển cửa sổ hay xem video.

## CRITICAL: Sự khác biệt giữa các phiên bản Picom

**Có HAI phiên bản picom, và chúng KHÔNG giống nhau:**

| Tính năng | Official picom (pacman) | ibhagwan fork (AUR) |
|---|---|---|
| Gói | `picom` | `picom-ibhagwan-git` |
| Shadow | Có | Có |
| Fading | Có | Có |
| VSync | Có | Có |
| **Animation** | **KHÔNG** | **Có** |
| **Blur** | **KHÔNG** | **Có** |
| Kho lưu trữ | Official | AUR |

**Lưu ý quan trọng:**
- `animations = true`, `animation-window-mass`, `animation-for-open-window`, v.v.
  là cú pháp **chỉ có trong fork của ibhagwan**. Nếu bạn dùng picom từ official
  repos (`pacman -S picom`), các dòng này sẽ bị **bỏ qua hoặc gây lỗi**.
- `blur-method`, `blur-size`, `blur-deviation`, `blur-strength` cũng là
  **fork-only**. Official picom không hỗ trợ blur background.

Quyết định: Với laptop tầm trung như LOQ 15IAX9, official picom là đủ.
Animation và blur làm tốn GPU và có thể gây lag. Nếu bạn muốn các tính năng đó,
cài từ AUR.

## Các bước thực hiện

### Bước 1: Cài Picom

**Option A — Official (khuyên dùng):**

```bash
pacman -S picom
```

**Option B — ibhagwan fork (có animation + blur):**

```bash
# Cần yay hoặc paru
yay -S picom-ibhagwan-git
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

#### Dành cho Official Picom (không animation, không blur)

```conf
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
    "class_g = 'Nitrogen'"
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

#### Dành cho ibhagwan fork (có animation + blur)

Nếu bạn cài `picom-ibhagwan-git`, dùng config này (thêm animation và blur):

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
    "class_g = 'Nitrogen'"
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
#      Blur                     #
#################################

blur-method = "kawase";
blur-size = 12;
blur-deviation = 5;
blur-strength = 5;
blur-background = true;
blur-background-frame = false;
blur-background-fixed = false;
blur-kern = "3x3box";
blur-background-exclude = [
    "window_type = 'dock'",
    "window_type = 'desktop'"
];

#################################
#     Window type settings      #
#################################

wintypes:
{
    tooltip = { fade = true; shadow = false; opacity = 0.85; };
    dock = { shadow = false; };
    desktop = { shadow = false; };
    menu = { opacity = 0.95; };
    popup_menu = { opacity = 0.95; dropdown_menu = { opacity = 0.95; }; };
};

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
unredir-if-possible = false;
```

`unredir-if-possible = false` rất quan trọng với NVIDIA:
Khi true, picom sẽ tắt compositor khi cửa sổ fullscreen → gây tearing.
Đặt false để luôn bật compositor.

### Bước 5: Chọn backend

Picom hỗ trợ hai backend chính:

| Backend | Ưu điểm | Nhược điểm |
|---|---|---|
| `glx` (mặc định) | Dùng GPU, nhanh, vsync tốt | Có thể lỗi với NVIDIA |
| `xrender` | Ổn định, ít lỗi | Chậm hơn, vsync kém hơn |

Không cần thêm dòng backend nếu muốn dùng mặc định (glx).
Nếu muốn đổi sang xrender, thêm vào đầu file:

```ini
backend = "xrender";
```

Có thể chỉ định backend khi chạy:

```bash
picom --backend glx --config ~/.config/picom/picom.conf
picom --backend xrender --config ~/.config/picom/picom.conf
```

### Bước 6: Kích hoạt trong bspwmrc

Trong `bspwmrc` đã có dòng:

```bash
picom --config ~/.config/picom/picom.conf &
```

Nếu chưa có, thêm trước dòng `exec`.

### Bước 7: Kiểm tra

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

### Tắt shadow cho terminal (tăng performance)

```ini
shadow-exclude = [
    "class_g = 'Alacritty'"
];
```

### Nếu dùng fork ibhagwan bị lag

Tắt animation:

```ini
animations = false;
```

Tắt blur:

```ini
blur-background = false;
```

## Troubleshooting

### Screen tearing vẫn còn

```ini
vsync = true;
unredir-if-possible = false;
```

Nếu vẫn bị tearing, thử backend xrender:

```bash
picom --backend xrender --config ~/.config/picom/picom.conf
```

### Picom làm chậm ứng dụng

Thử backend xrender thay vì glx:

```ini
backend = "xrender";
```

### "Unknown option: animations" hoặc "Unknown option: blur-method"

Bạn đang dùng **official picom** nhưng copy config của ibhagwan fork.
Xóa các dòng animations và blur khỏi config.

### Picom crash

- Kiểm tra log: `cat /tmp/picom.log`.
- Dùng backend xrender.
- Tắt `vsync` thử (true → false).

## Tổng kết

- Picom đã được cài với cấu hình shadow, fade, vsync.
- **Phân biệt rõ:** official picom (không animation/blur) vs ibhagwan fork (có).
- VSync được bật để chống tearing.
- Cấu hình NVIDIA-specific để tránh lỗi fullscreen.
- Picom tự động chạy cùng bspwm.
