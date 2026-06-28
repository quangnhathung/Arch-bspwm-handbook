# Picom — Compositor

Ngày cập nhật: 28/06/2026

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

**Cấu hình thực tế dùng:** `picom-ftlabs-git` (AUR) — fork có animation spring physics.

```bash
yay -S picom-ftlabs-git
```

> Fork `ftlabs` khác với `ibhagwan`: dùng **spring physics** (lò xo) thay vì
> easing thông thường, tạo hiệu ứng nảy tự nhiên. Config sử dụng các option
> `animation-stiffness`, `animation-dampening`, `animation-mass`.

**Các lựa chọn khác:**

| Gói | Animation | Blur | Spring physics |
|-----|-----------|------|----------------|
| `picom` (official) | ❌ | ❌ | ❌ |
| `picom-ibhagwan-git` (AUR) | ✅ Easing | ✅ | ❌ |
| **`picom-ftlabs-git`** (AUR) ⭐ | ✅ **Spring** | ❌ | ✅ |

### Bước 2: Tạo thư mục và file config

```bash
su - archuser
mkdir -p ~/.config/picom
exit
```

```bash
vim /home/archuser/.config/picom/picom.conf
```

### Bước 3: File cấu hình picom.conf (cấu hình thực tế)

**Yêu cầu:** Gói `picom-ftlabs-git` (AUR) — fork có animation spring physics.

```bash
yay -S picom-ftlabs-git
```

```conf
# =========================================================
# 1. BACKEND & PERFORMANCE
# =========================================================
backend = "glx";
vsync = true;
use-damage = false;
xrender-sync-fence = true;
unredir-if-possible = false;     # Quan trọng với NVIDIA

# =========================================================
# 2. SHADOWS
# =========================================================
shadow = true;
shadow-radius = 28;              # Bóng mềm, lan rộng
shadow-opacity = 0.18;           # Nhẹ, tinh tế
shadow-offset-x = 0;
shadow-offset-y = 8;             # Đổ xuống dưới
shadow-exclude = [
  "class_g = 'Rofi'",
  "class_g = 'Dunst'",
  "_GTK_FRAME_EXTENTS@:c"
];

# =========================================================
# 3. OPACITY
# =========================================================
active-opacity = 1.0;
inactive-opacity = 0.9;
frame-opacity = 1.0;
inactive-opacity-override = true;

detect-client-opacity = true;
detect-transient = true;

opacity-rule = [
  "90:class_g = 'Code' && !focused",
  "100:class_g = 'Code' && focused",
  "90:class_g = 'Google-chrome' && !focused",
  "100:class_g = 'Google-chrome' && focused",
  "90:class_g = 'Thunar' && !focused",
  "100:class_g = 'Thunar' && focused",
  "80:class_g = 'Alacritty' && !focused",
  "100:class_g = 'Alacritty' && focused",
  "95:class_g = 'code' && !focused",
  "100:class_g = 'code' && focused",
  "90:class_g = 'Org.gnome.gThumb' && !focused",
  "100:class_g = 'Org.gnome.gThumb' && focused",
];

# =========================================================
# 4. FADING
# =========================================================
fading = true;
fade-in-step = 0.03;
fade-out-step = 0.03;
fade-delta = 5;
no-fading-openclose = false;

# =========================================================
# 5. ANIMATIONS — Spring physics (picom-ftlabs-git)
# =========================================================
animations = true;
animation-stiffness = 300.0;     # Độ cứng lò xo
animation-dampening = 22.0;      # Giảm chấn — tạo độ nảy
animation-clamping = true;
animation-mass = 1.0;

animation-for-open-window = "zoom";
animation-for-unmap-window = "zoom";
animation-for-workspace-switch-in = "zoom";
animation-for-workspace-switch-out = "zoom";
animation-for-transient-window = "zoom";

# =========================================================
# 6. FOCUS & BORDERS
# =========================================================
mark-wmwin-focused = true;
mark-ovredir-focused = true;
detect-rounded-corners = true;

# =========================================================
# 7. WINDOW TYPES
# =========================================================
wintypes:
{
  tooltip = { fade = false; shadow = false; opacity = 0.90; };
  desktop = { shadow = false; };
  menu = { opacity = 0.95; };
  popup_menu = { opacity = 0.95; };
  dropdown_menu = { opacity = 0.95; };
};

# =========================================================
# 8. ROUNDED CORNERS
# =========================================================
corner-radius = 13;              # Bo góc 13px
round-borders = 1;               # Ép viền bspwm theo góc bo

rounded-corners-exclude = [
  "window_type = 'dock'",
  "window_type = 'desktop'",
  "class_g = 'Polybar'",         # Polybar tự bo góc trong config
  "name = 'Notification'",
];
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
