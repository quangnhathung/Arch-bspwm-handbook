# Tại sao chọn bspwm?

## Khái niệm Window Manager

Window Manager (WM) là chương trình quản lý vị trí và kích thước cửa sổ.
Khác với Desktop Environment (DE) như GNOME hay KDE, WM không bao gồm
thanh taskbar, menu Start, dock, hay các tiện ích đồ họa khác.

### Tiling vs Stacking

| Loại | Cách hoạt động | Ví dụ |
|---|---|---|
| **Stacking** | Cửa sổ chồng lên nhau như Windows/macOS | Openbox, i3 |
| **Tiling** | Cửa sổ tự động sắp xếp cạnh nhau, không chồng | bspwm, dwm, i3 |

### bspwm thuộc loại Tiling-WM dạng Binary Space Partitioning (BSP)

Màn hình được chia thành các không gian (node) bằng các đường phân cách ngang
hoặc dọc. Mỗi lần mở cửa sổ mới, node hiện tại bị chia làm hai.

```
+-----------+-----------+
|           |           |
|   Term    |   Browser |
|           |           |
+-----------+-----------+
|           |           |
|   Code    |   Chat    |
|           |           |
+-----------+-----------+
```

## Tại sao bspwm?

### 1. Nhẹ và nhanh

bspwm là một WM cực kỳ nhẹ. Bản thân nó chỉ là một binary quản lý cửa sổ,
không có GUI config, không settings panel. Mọi thứ đều qua file cấu hình
text và được reload bằng hotkey.

### 2. Phân tách rõ ràng

bspwm chỉ quản lý cửa sổ. Nó không xử lý keybinding — việc đó do sxhkd đảm nhiệm.
Kiến trúc phân tách này giúp dễ hiểu, dễ sửa, dễ thay thế từng phần.

```
bspwm ───────────► quản lý cửa sổ, workspace, node
sxhkd ───────────► nhận phím tắt, gọi lệnh
Polybar ─────────► thanh trạng thái
Rofi ────────────► launcher ứng dụng
Picom ───────────► compositor (đổ bóng, mượt)
```

### 3. Không cần chuột

Mọi thao tác đều qua bàn phím. Chuyển workspace, di chuyển cửa sổ, thay đổi
kích thước — tất cả bằng phím tắt. Sau khi quen, thao tác nhanh hơn hẳn dùng chuột.

### 4. Cấu hình bằng text

Không có GUI config. Mọi thứ đều là text file đơn giản:

- `~/.config/bspwm/bspwmrc` — script khởi tạo WM
- `~/.config/sxhkd/sxhkdrc` — keybinding

Có thể đưa vào Git để quản lý. Sao chép sang máy khác dễ dàng.

### 5. Workspace (Desktop ảo)

Mặc định có 9 workspace. Mỗi màn hình có bộ workspace riêng.
Có thể gán ứng dụng vào workspace cụ thể.

### 6. Tính modular

Không có lock-in. Bạn có thể:
- Thay sxhkd bằng keyd, kmonad
- Thay Polybar bằng eww, dwm-bar
- Thay Picom bằng wayland hoàn toàn
- Từng phần độc lập, không ảnh hưởng nhau

## So sánh với các WM khác

| Đặc điểm | bspwm | i3 | dwm | hyprland |
|---|---|---|---|---|
| Loại | BSP | Tiling manual | Dynamic | Wayland |
| Config | File text | File text | Patch C | File text |
| Keybinding | sxhkd | Built-in | Built-in | Built-in |
| Multi-monitor | Tốt | Tốt | Cần patch | Tốt |
| Học | Dễ | Dễ | Trung bình | Trung bình |

## Kết luận

Chọn bspwm vì bạn muốn một hệ thống desktop nhanh, nhẹ, kiểm soát được mọi thứ
bằng bàn phím, và dễ tùy chỉnh. Không phù hợp nếu bạn cần kéo thả cửa sổ
bằng chuột như Windows.
