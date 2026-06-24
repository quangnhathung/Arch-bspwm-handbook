# Tại sao chọn bspwm?

## Window Manager là gì?

**Window Manager (WM)** là chương trình quản lý vị trí và kích thước cửa sổ. Khác với Desktop Environment (GNOME, KDE), WM **không** bao gồm thanh taskbar, menu Start, dock, hay tiện ích đồ họa — bạn tự chọn và ghép các thành phần riêng lẻ.

### Tiling vs Stacking

| Loại | Cách hoạt động | Ví dụ |
|------|---------------|-------|
| **Stacking** | Cửa sổ chồng lên nhau như Windows / macOS | Openbox, Fluxbox |
| **Tiling** | Cửa sổ tự động sắp xếp cạnh nhau, không chồng lấn | bspwm, i3, dwm, Hyprland |

## Binary Space Partitioning (BSP) — Giải thích bằng hình

bspwm thuộc loại **Tiling WM dạng BSP**. Màn hình được chia thành các node bằng đường phân cách ngang/dọc. Mỗi lần mở cửa sổ mới, node hiện tại bị chia đôi:

```
Bước 1: 1 cửa sổ          Bước 2: Mở thêm 1 cửa sổ (chia dọc)
+-------------------------+  +------------+------------+
|                         |  |            |            |
|        Term             |  |   Term     |   Browser  |
|                         |  |            |            |
+-------------------------+  +------------+------------+

Bước 3: Mở thêm 1 (chia ngang Term)     Bước 4: Mở thêm 1 (chia ngang Browser)
+------------+------------+  +------------+------------+
|   Term     |            |  |   Term     |   Browser  |
+------------+  Browser   |  +------------+------------+
|   Code     |            |  |   Code     |   Chat     |
+------------+------------+  +------------+------------+
```

Cấu trúc cây nhị phân này giúp thao tác với cửa sổ nhanh và dễ dự đoán — bạn luôn biết cửa sổ mới sẽ xuất hiện ở đâu.

## Tại sao bspwm?

### 1. Cực kỳ nhẹ

Bản thân bspwm là một binary nhỏ chỉ quản lý cửa sổ. Không GUI config, không settings panel. RAM ~5–10MB.

### 2. Phân tách rõ ràng (Modular)

**bspwm** chỉ quản lý cửa sổ — nó không xử lý keybinding. Việc đó do **sxhkd** đảm nhiệm.

```
bspwm       → quản lý cửa sổ, workspace, node
sxhkd       → nhận phím tắt, gọi lệnh
Polybar     → thanh trạng thái
Rofi        → launcher ứng dụng
Picom       → compositor (đổ bóng, mượt, animation)
```

Bạn có thể thay từng phần mà không ảnh hưởng phần còn lại. Muốn dùng `keyd` thay sxhkd? Được. Muốn dùng `eww` thay Polybar? Được.

### 3. Keyboard-driven

Mọi thao tác qua bàn phím: chuyển workspace, di chuyển cửa sổ, resize, thay đổi state (tiled / monocle / floating). Sau khi thuộc keybinding, thao tác nhanh hơn dùng chuột rất nhiều.

### 4. Cấu hình bằng text file

- `~/.config/bspwm/bspwmrc` — shell script khởi tạo WM
- `~/.config/sxhkd/sxhkdrc` — file keybinding

Cả hai đều là text thuần, có thể đưa vào Git, sao chép sang máy khác.

### 5. Workspace linh hoạt

Mặc định 9 workspace, mỗi màn hình có bộ workspace riêng. Gán ứng dụng vào workspace cụ thể, chuyển workspace bằng `super + 1..9`.

### 6. Không lock-in

Từng phần độc lập: sxhkd → keyd / kmonad, Polybar → eww / dwm-bar, Picom → wayland hoàn toàn.

## So sánh: bspwm vs các WM phổ biến

| Đặc điểm | bspwm | i3 | dwm | Hyprland |
|---------|-------|----|-----|----------|
| **Loại** | BSP tiling | Tiling manual | Dynamic tiling | Wayland compositor |
| **Config** | Shell script | Text file | Patch C source | Text file |
| **Keybinding** | sxhkd (riêng) | Built-in | Built-in | Built-in |
| **Multi-monitor** | Tốt | Tốt | Cần patch | Tốt |
| **RAM (idle)** | ~5MB | ~10MB | ~5MB | ~50MB |
| **Độ khó học** | Dễ | Dễ | Trung bình | Trung bình |
| **Hỗ trợ Wayland** | Không (Xorg) | Không (Xorg) | Không (Xorg) | Có (bản chất) |
| **Animation** | Qua picom | Qua picom | Không | Có sẵn |

## Khi nào KHÔNG nên dùng bspwm?

- Bạn cần kéo thả cửa sổ bằng chuột như Windows
- Bạn ghét đọc tài liệu và cấu hình text file
- Bạn muốn desktop có hiệu ứng 3D, animation mượt (chọn Hyprland)
- Bạn dùng nhiều ứng dụng GTK không tương thích với tiling (một số cũ)

## Kết luận

Chọn bspwm nếu bạn muốn một desktop nhanh, nhẹ, kiểm soát mọi thứ bằng bàn phím, và sẵn sàng đầu tư thời gian để cấu hình. Việc phân tách bspwm + sxhkd giúp hệ thống dễ hiểu, dễ sửa, và cực kỳ linh hoạt.
