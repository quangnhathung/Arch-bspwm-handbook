# Yêu cầu hệ thống

## Yêu cầu phần cứng tối thiểu

| Thành phần | Yêu cầu |
|---|---|
| CPU | x86_64 (64-bit), Intel hoặc AMD |
| RAM | 1GB (tối thiểu), 4GB+ (khuyên dùng cho desktop) |
| Ổ cứng | 20GB (tối thiểu), 256GB+ (khuyên dùng) |
| GPU | Hỗ trợ KMS (Kernel Mode Setting) |
| USB | Cổng USB để boot live environment |
| Internet | Cần trong quá trình cài đặt |

## Thông số máy mục tiêu

Lenovo LOQ 15IAX9 — cấu hình sử dụng trong tài liệu này:

| Thành phần | Chi tiết | Ghi chú |
|---|---|---|
| CPU | Intel Core i5-12450HX | 12th Gen Alder Lake, 8 nhân (4P+4E) |
| GPU tích hợp | Intel UHD Graphics (Alder Lake) | Dùng cho mode Intel |
| GPU rời | NVIDIA GeForce RTX 4050 6GB | Dùng cho rendering nặng |
| RAM | 16GB DDR5-4800 | 2 khe, không hàn |
| Ổ cứng | Micron 2400 512GB NVMe | M.2 2280 PCIe 4.0 |
| Wi-Fi | Realtek RTL8852BE | 802.11ax (Wi-Fi 6) |
| Bluetooth | Realtek Bluetooth | Đi kèm chip Wi-Fi |
| Ethernet | Realtek RTL8111/8168 | Gigabit Ethernet |
| Audio | Intel Smart Sound Technology (SST) | HDMI/DP audio qua NVIDIA |
| Màn hình | 15.6" FHD (1920x1080) | 144Hz, không touch |
| Bàn phím | Tiếng Việt / US | Có đèn nền |
| Touchpad | Microsoft Precision | Hỗ trợ đa điểm |

## Yêu cầu phần mềm

### Máy dùng để tạo USB boot

- Windows / Linux / macOS
- Rufus (Windows) hoặc dd (Linux) hoặc balenaEtcher (any)
- Trình duyệt web để tải Arch ISO
- USB 4GB+

### Trong quá trình cài đặt

- Arch Linux ISO (tải từ https://archlinux.org/download/)
- Kết nối Internet (Wi-Fi hoặc USB tethering)
- Kiến thức cơ bản về dòng lệnh Linux

## Kiểm tra tương thích trước khi cài

1. **UEFI**: Vào BIOS (F2 khi khởi động) → Boot Mode = UEFI (không Legacy).
2. **Secure Boot**: Nên tắt (có thể bật lại sau, nhưng phức tạp).
3. **Fast Boot**: Nên tắt để tránh lỗi boot USB.
4. **RAID mode**: Nếu BIOS set RAID → chuyển sang AHCI.

## Lưu ý đặc thù cho Lenovo LOQ 15IAX9

- Máy có **NVIDIA Optimus** (hybrid graphics). Cần cấu hình đúng để tránh hao pin.
- Wi-Fi chip **Realtek RTL8852BE** không được Linux kernel hỗ trợ mặc định.
  Cần cài driver `rtl8852be-dkms` từ AUR.
- Audio **Intel SST** cần `sof-firmware` và `alsa-firmware`.
- Máy có 2 quạt + chế độ performance trong BIOS.
