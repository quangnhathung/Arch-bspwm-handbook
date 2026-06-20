# Tổng quan

Tài liệu này hướng dẫn cài đặt Arch Linux hoàn toàn mới trên laptop Lenovo LOQ 15IAX9,
kèm theo thiết lập môi trường desktop với bspwm window manager.

## Triết lý

- **Tối giản**: Chỉ cài những gì thực sự cần. Desktop không có menu Start, không có dock,
  không có DE (Desktop Environment) nặng nề. Mọi thứ đều qua bàn phím.
- **Kiểm soát**: Bạn quyết định mọi thứ — từ cài đặt đến cấu hình.
- **Học tập**: Quá trình cài Arch là cách tốt nhất để hiểu Linux thực sự hoạt động thế nào.
- **Bền vững**: Hệ thống được thiết kế để dễ bảo trì, dễ rollback, dễ mở rộng sau này.

## Phạm vi tài liệu

### Bao gồm

- Cài đặt Arch Linux từ USB đến hệ thống hoàn chỉnh
- Phân vùng BTRFS với subvolume
- Cấu hình GRUB cho UEFI
- Thiết lập Xorg display server
- Cài đặt và cấu hình bspwm + sxhkd
- Polybar, Rofi, Picom, Nitrogen
- Driver Intel + NVIDIA
- Wi-Fi, Bluetooth, âm thanh
- Quản lý gói với pacman và yay
- BTRFS snapshot với Timeshift
- Hướng dẫn sử dụng bspwm chi tiết
- Troubleshooting các lỗi thường gặp

### Không bao gồm

- Môi trường phát triển (IDE, Docker, Git workflow)
- Ngôn ngữ lập trình (Go, Java, Python, NodeJS)
- Game và tối ưu game (Steam, Proton, Wine)
- Phần mềm văn phòng nâng cao
- Server và dịch vụ mạng phức tạp

## Máy mục tiêu: Lenovo LOQ 15IAX9

| Thành phần | Chi tiết |
|---|---|
| CPU | 12th Gen Intel Core i5-12450HX |
| GPU 1 | Intel UHD Graphics (integrated) |
| GPU 2 | NVIDIA GeForce RTX 4050 Laptop GPU |
| RAM | 16GB DDR5 |
| Storage | Micron NVMe 512GB |
| Wi-Fi | Realtek RTL8852BE |
| LAN | Realtek GbE |
| Audio | Intel Smart Sound Technology (SST) |

## Trước khi bắt đầu

1. **Backup dữ liệu**: Toàn bộ dữ liệu trên ổ cứng sẽ bị xóa sạch.
2. **USB boot**: Chuẩn bị USB 4GB+ để ghi Arch ISO.
3. **Internet**: Đảm bảo có kết nối mạng trong quá trình cài.
4. **Kiên nhẫn**: Đây là quá trình thủ công, có thể mất 1-2 giờ cho lần đầu.
