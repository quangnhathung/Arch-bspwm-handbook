# Yêu cầu hệ thống

## Yêu cầu phần cứng tối thiểu

| Thành phần | Yêu cầu |
|-----------|---------|
| CPU | x86_64 (64-bit), Intel hoặc AMD |
| RAM | 1GB tối thiểu, 4GB+ khuyên dùng (desktop + trình duyệt) |
| Ổ cứng | 20GB tối thiểu, 256GB+ khuyên dùng |
| GPU | Hỗ trợ KMS (Kernel Mode Setting) |
| USB | Cổng USB để boot live environment |
| Internet | Bắt buộc trong quá trình cài đặt |

## Thông số máy mục tiêu — Lenovo LOQ 15IAX9

| Thành phần | Chi tiết | Ghi chú |
|-----------|----------|---------|
| **CPU** | Intel Core i5-12450HX | 12th Gen Alder Lake, 8 nhân (4P + 4E), 12 luồng |
| **GPU tích hợp** | Intel UHD Graphics (Alder Lake) | Dùng cho chế độ Intel |
| **GPU rời** | NVIDIA GeForce RTX 4050 6GB GDDR6 | Dùng cho rendering nặng, game, AI |
| **RAM** | 16GB DDR5-4800 | 2 khe SO-DIMM, không hàn → có thể nâng cấp |
| **Storage** | Micron 2400 512GB NVMe | M.2 2280 PCIe 4.0 ×4 |
| **Wi-Fi** | Realtek RTL8852BE | 802.11ax (Wi-Fi 6), 2.4 + 5 GHz |
| **Bluetooth** | Realtek Bluetooth 5.2 | Đi kèm chip Wi-Fi |
| **Ethernet** | Realtek RTL8111/8168 | Gigabit Ethernet (RJ45) |
| **Audio** | Intel Smart Sound Technology (SST) | HDMI/DP audio qua NVIDIA |
| **Màn hình** | 15.6" FHD (1920×1080) | 144Hz, IPS, không touch |
| **Bàn phím** | Tiếng Việt / US | Có đèn nền trắng |
| **Touchpad** | Microsoft Precision | Hỗ trợ đa điểm, gesture |
| **Pin** | 60Wh | 3 cell Li-Po |

## Yêu cầu phần mềm

### Máy dùng để tạo USB boot

- **Hệ điều hành**: Windows / Linux / macOS — bất kỳ
- **Công cụ ghi USB**:
  - Windows: Rufus (khuyên dùng) hoặc balenaEtcher
  - Linux: `dd` hoặc `cp` (có hướng dẫn trong bài)
  - macOS: balenaEtcher hoặc `dd`
- **USB**: 4GB+ (USB 3.0 khuyên dùng để ghi nhanh)

### Trong quá trình cài đặt

- **Arch Linux ISO**: Tải từ https://archlinux.org/download/ (chọn mirror gần nhất)
- **Internet**: Wi-Fi (qua iwctl) hoặc USB tethering từ điện thoại
- **Kiến thức nền tảng**: Biết dùng terminal cơ bản (cd, ls, cat, nano/vim)

## Kiểm tra tương thích trước khi cài

Vào BIOS (nhấn **F2** khi khởi động, hoặc **Fn + F2** nếu phím tắt không hoạt động):

| Mục | Cần đặt | Ghi chú |
|-----|---------|---------|
| **Boot Mode** | UEFI | Không dùng Legacy / CSM |
| **Secure Boot** | Disabled | Có thể bật lại sau, nhưng sẽ cần cấu hình thêm (phức tạp) |
| **Fast Boot** | Disabled | Tránh lỗi boot USB và GRUB sau này |
| **SATA Mode** | AHCI | Nếu đang là RAID → bắt buộc chuyển sang AHCI |
| **UEFI Boot Order** | USB đầu tiên | Để boot từ USB cài đặt |

### Cách kiểm tra nhanh trước khi cài

Sau khi boot từ USB Arch, chạy:

```bash
# Kiểm tra UEFI (có thư mục này là OK)
ls /sys/firmware/efi/efivars

# Kiểm tra kết nối mạng
ip link

# Kiểm tra ổ cứng
lsblk
```

Nếu `ls /sys/firmware/efi/efivars` báo lỗi → bạn đang boot ở Legacy mode, cần vào BIOS chuyển sang UEFI.

## Lưu ý đặc thù cho Lenovo LOQ 15IAX9

### 1. NVIDIA Optimus (Hybrid Graphics)

Máy có cả Intel UHD (tiết kiệm pin) và NVIDIA RTX 4050 (hiệu năng cao). Cần cấu hình đúng:
- Mặc định dùng Intel để tiết kiệm pin
- Chạy ứng dụng nặng qua NVIDIA bằng `prime-run` (từ gói `nvidia-prime`)

### 2. Wi-Fi Realtek RTL8852BE

Chip Wi-Fi này **không được kernel Linux hỗ trợ mặc định**. Cần cài driver:

```bash
# Cách 1: AUR (khuyên dùng)
yay -S rtl8852be-dkms

# Cách 2: Build từ git trên GitHub (nếu AUR không hoạt động)
```

Hướng dẫn chi tiết trong bài 05-drivers/05-wifi.md.

### 3. Âm thanh Intel SST

Cần `sof-firmware` và `alsa-firmware` để Intel Smart Sound Technology hoạt động:

```bash
pacman -S sof-firmware alsa-firmware
```

### 4. Quạt và nhiệt độ

Máy có 2 quạt với chế độ Performance / Balanced / Battery Saving trong BIOS. Cài `tlp` hoặc `auto-cpufreq` để quản lý nhiệt và pin sau khi cài xong.

### 5. Bàn phím đèn nền

Phím tắt **Fn + Space** để bật/tắt đèn nền bàn phím. Mặc định đã hoạt động trên Linux, không cần cấu hình thêm.

### 6. Touchpad

Touchpad Microsoft Precision được kernel hỗ trợ tốt. Dùng `libinput` để cấu hình gesture (có hướng dẫn trong phần desktop).
