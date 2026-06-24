# Tổng quan

Tài liệu này hướng dẫn cài đặt **Arch Linux hoàn toàn mới** trên laptop **Lenovo LOQ 15IAX9**, kèm thiết lập môi trường desktop với **bspwm window manager** — một hệ thống tiling cực nhẹ, điều khiển hoàn toàn bằng bàn phím.

## Triết lý

- **Tối giản**: Chỉ cài những gì thực sự cần. Desktop không có menu Start, không dock, không DE nặng nề. Mọi thao tác qua bàn phím.
- **Kiểm soát**: Bạn quyết định mọi thứ — từ cài đặt kernel parameter đến font chữ.
- **Học tập**: Cài Arch thủ công là cách nhanh nhất để hiểu Linux thực sự hoạt động ra sao.
- **Bền vững**: Hệ thống được thiết kế để dễ bảo trì, dễ rollback (BTRFS + Timeshift), dễ mở rộng.

## Phạm vi

### Bao gồm

- Cài đặt Arch Linux: phân vùng BTRFS, GRUB UEFI, base system
- Thiết lập Xorg display server với `xorg-xinit` (lưu ý: gói tên là `xorg-xinit`, không phải `xorg-init`)
- Desktop: bspwm + sxhkd, Polybar, Rofi, Picom (ibhagwan fork cho animation/blur), Nitrogen
- Driver: Intel UHD, NVIDIA RTX 4050 (dùng `nvidia-open` — khuyến nghị chính thức cho dòng RTX 30/40), hybrid graphics Optimus
- Wi-Fi Realtek RTL8852BE, Bluetooth, âm thanh Intel SST (PipeWire)
- Quản lý gói: pacman, yay (AUR helper)
- BTRFS snapshot với Timeshift
- Hướng dẫn sử dụng bspwm chi tiết: keybinding, workspace, window management
- Troubleshooting: màn hình đen, Wi-Fi, NVIDIA, GRUB

### Không bao gồm

- Môi trường phát triển (IDE, Docker, Git workflow)
- Ngôn ngữ lập trình (Go, Java, Python, Node.js)
- Game và tối ưu game (Steam, Proton, Wine)
- Phần mềm văn phòng nâng cao (LibreOffice macro, v.v.)
- Server và dịch vụ mạng phức tạp

## Máy mục tiêu: Lenovo LOQ 15IAX9

| Thành phần | Chi tiết |
|-----------|----------|
| CPU | 12th Gen Intel Core i5-12450HX (Alder Lake, 4P + 4E) |
| GPU tích hợp | Intel UHD Graphics (Alder Lake) |
| GPU rời | NVIDIA GeForce RTX 4050 Laptop GPU 6GB GDDR6 |
| RAM | 16GB DDR5-4800 (2 khe, có thể nâng cấp) |
| Storage | Micron 2400 512GB NVMe PCIe 4.0 |
| Wi-Fi | Realtek RTL8852BE (Wi-Fi 6) |
| Ethernet | Realtek RTL8111/8168 Gigabit |
| Audio | Intel Smart Sound Technology (SST) |
| Màn hình | 15.6" FHD (1920×1080) 144Hz IPS |
| Bàn phím | Tiếng Việt / US, có đèn nền |
| Touchpad | Microsoft Precision, đa điểm |

## Trước khi bắt đầu

1. **Sao lưu toàn bộ dữ liệu** — ổ cứng sẽ bị xóa sạch
2. **USB 4GB+** — để ghi Arch ISO
3. **Kết nối Internet** — bắt buộc trong quá trình cài
4. **Kiên nhẫn** — lần đầu có thể mất 2–4 giờ nếu cài thủ công
5. **Đọc hết phần 01-introduction** trước khi làm bất cứ gì
