# Arch Linux + bspwm Handbook — Lenovo LOQ 15IAX9

Tài liệu hướng dẫn cài đặt Arch Linux từ con số 0, cấu hình window manager bspwm,
và các thành phần desktop cơ bản trên laptop **Lenovo LOQ 15IAX9**.

## Đối tượng

Người dùng muốn chuyển từ Windows sang Linux, cài mới hoàn toàn, không dual boot.

## Mục tiêu

| Mục tiêu | Chi tiết |
|---|---|
| Cài đặt | Arch Linux, BTRFS, GRUB UEFI |
| Desktop | Xorg + bspwm + sxhkd |
| Đồ họa | Intel UHD + NVIDIA RTX 4050 (hybrid) |
| Mạng | Wi-Fi Realtek RTL8852BE + LAN |
| Âm thanh | PipeWire + Intel SST |
| Snapshot | BTRFS + Timeshift |

## Cấu trúc tài liệu

```
docs/
├── 01-introduction/      Giới thiệu tổng quan
├── 02-preparation/        Chuẩn bị trước khi cài
├── 03-installation/       Cài đặt hệ thống nền tảng
├── 04-desktop/            Thiết lập môi trường đồ họa
├── 05-drivers/            Driver phần cứng
├── 06-package-management/ Quản lý gói
├── 07-btrfs/              Quản lý BTRFS và snapshot
├── 08-bspwm-guide/        Hướng dẫn sử dụng bspwm
├── 09-troubleshooting/    Xử lý sự cố
└── appendix/              Phụ lục
```

## Luồng thực hiện

1. Đọc **01-introduction** để hiểu tổng quan.
2. Làm theo **02-preparation** để chuẩn bị USB boot.
3. Boot USB và làm theo **03-installation** từ đầu đến cuối.
4. Sau khi reboot vào hệ thống mới, làm theo **04-desktop** để có desktop.
5. Cấu hình driver theo **05-drivers**.
6. Đọc **06-package-management** để biết quản lý gói.
7. Thiết lập snapshot theo **07-btrfs**.
8. Đọc **08-bspwm-guide** để làm quen với bspwm.
9. Gặp lỗi → vào **09-troubleshooting**.
10. **appendix** là cheat sheet và checklist.

## Yêu cầu

- USB 4GB+
- Máy tính có truy cập Internet
- Sẵn sàng xóa toàn bộ dữ liệu cũ trên ổ cứng

## Cảnh báo

> Quá trình này sẽ **xóa sạch toàn bộ dữ liệu Windows hiện tại**.
> Không giữ lại dual boot. Coi máy như một bản cài hoàn toàn mới.
> Sao lưu dữ liệu quan trọng trước khi bắt đầu.
