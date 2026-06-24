# Tổng Quan Desktop — Các Thành Phần Giao Diện

Ngày cập nhật: 25/06/2026

Bài này giới thiệu tổng quan về **bộ giao diện "chuẩn modern bspwm"**
mà đa số người dùng Linux (r/unixporn style) hay dùng. Các bài tiếp theo
trong section này sẽ đi sâu vào cài đặt và cấu hình từng thành phần.

---

## 🧠 1. Window Manager core (bắt buộc)

| Component | Vai trò |
|---|---|
| **bspwm** | Window manager chính — quản lý vị trí, kích thước cửa sổ |
| **sxhkd** | Daemon phím tắt — điều khiển mọi thứ bằng bàn phím |

bspwm không xử lý bàn phím, không có thanh trạng thái, không có compositor.
Nó chỉ quản lý cửa sổ. Mọi thứ khác đều là chương trình riêng.

---

## 🎨 2. Compositor (hiệu ứng: blur, rounded, shadow)

| Component | Vai trò |
|---|---|
| **picom** | Thêm hiệu ứng: bo góc window, blur nền, shadow, transparency |

👉 **Đây là thứ "làm đẹp" quan trọng nhất.** Không có picom, bspwm trông
rất "thô" — không bóng đổ, không bo góc, không xuyên thấu.

---

## 📊 3. Status bar (thanh trên/dưới)

| Component | Vai trò |
|---|---|
| **polybar** | Thanh trạng thái hiển thị workspace, CPU/RAM, network, battery, clock |

Có thể đặt ở trên hoặc dưới màn hình, tuỳ chỉnh module.

---

## 🚀 4. App launcher (menu mở app)

| Component | Vai trò |
|---|---|
| **rofi** ⭐ | Launcher hiện đại — tìm app, chạy lệnh, chuyển cửa sổ, power menu |
| dmenu | Cũ, tối giản, ít dùng trên r/unixporn |

---

## 🖼️ 5. Wallpaper manager

| Component | Vai trò |
|---|---|
| **feh** | CLI, cực nhẹ, chuẩn bspwm |
| nitrogen | GUI, chọn wallpaper bằng chuột |

---

## 🧩 6. Notification system

| Component | Vai trò |
|---|---|
| **dunst** | Hiển thị thông báo: volume, wifi, update, system alert |

---

## 🗂️ 7. File manager (GUI)

| Component | Vai trò |
|---|---|
| **thunar** | Nhẹ, phổ biến, có bulk rename + plugin archive |
| pcmanfm | Cực nhẹ, tối giản |

---

## 🧪 8. Terminal emulator (cực quan trọng)

| Component | Vai trò |
|---|---|
| **alacritty** ⭐ | GPU-accelerated, siêu nhanh, config bằng TOML |
| kitty | Nhiều feature hơn (tabs, split, xem ảnh trong terminal) |

---

## 🧭 9. Dock (tuỳ chọn)

| Component | Vai trò |
|---|---|
| tint2 | Dock/panel kiểu taskbar |

Trên r/unixporn, **đa số không dùng dock** vì bspwm chuyển workspace
bằng phím tắt (Super + 1-9) đã rất nhanh.

---

## 🎧 10. Audio / system tray tools

| Component | Vai trò |
|---|---|
| **pavucontrol** | GUI điều chỉnh âm lượng, đầu ra âm thanh |
| **nm-applet** | Tray icon NetworkManager — chọn Wi-Fi, VPN |
| **blueman** | Tray icon + manager Bluetooth |

---

## 🔐 11. Authentication / system integration

| Component | Vai trò |
|---|---|
| **polkit-gnome** | Xác thực quyền root cho GUI (mount USB, cài app, GParted) |

---

## 📸 12. Screenshot tool

| Component | Vai trò |
|---|---|
| **flameshot** ⭐ | Chụp ảnh + chú thích (mũi tên, text, highlight, blur) |
| scrot | Basic, không GUI |

---

## 🧱 13. Font + theme (rất quan trọng cho "modern look")

| Component | Vai trò |
|---|---|
| **JetBrains Mono** / Fira Code | Font monospace cho terminal + code |
| **Catppuccin** ⭐ | Theme phổ biến nhất trên r/unixporn (Mocha/Frappé/Latte) |
| Dracula / Nord | Theme thay thế |

---

## 🔥 14. Bộ setup "chuẩn r/unixporn" (combo thực tế)

Setup phổ biến nhất mà bạn sẽ thấy trên r/unixporn:

| Phân loại | Công cụ |
|---|---|
| WM | bspwm + sxhkd |
| Compositor | picom (blur + rounded) |
| Bar | polybar |
| Launcher | rofi |
| Wallpaper | feh |
| Notification | dunst |
| Terminal | alacritty |
| System tray | nm-applet + pavucontrol |
| Screenshot | flameshot |
| Theme | Catppuccin Mocha |
| Font | JetBrains Mono |

> Chi tiết cài đặt và config có ở các bài tiếp theo trong section này
> và bài [10-modern-setup.md](10-modern-setup.md).

---

## Luồng hoạt động

```
sxhkd (phím tắt)
  │
  ├──► bspwm (điều khiển cửa sổ)
  ├──► rofi (mở app)
  ├──► alacritty (mở terminal)
  ├──► flameshot (chụp ảnh)
  └──► dunst (gửi thông báo)
  └──► pavucontrol (chỉnh âm lượng)

bspwmrc (autostart khi boot)
  ├──► picom (compositor)
  ├──► polybar (status bar)
  ├──► feh (wallpaper)
  ├──► dunst (notification)
  ├──► nm-applet (Wi-Fi tray)
  ├──► blueman-applet (Bluetooth tray)
  ├──► xfce4-power-manager (pin)
  └──► polkit-gnome (xác thực)
```
