# Cấu hình BIOS cho Lenovo LOQ 15IAX9

## UEFI vs Legacy

### UEFI (Unified Extensible Firmware Interface)

Chuẩn firmware hiện đại, thay thế Legacy BIOS từ khoảng 2012. Đặc điểm:

- Hỗ trợ ổ cứng **GPT** (dung lượng >2TB)
- Boot nhanh hơn nhờ tối ưu firmware
- Hỗ trợ Secure Boot, boot manager
- Giao diện đồ họa, hỗ trợ chuột
- **Bắt buộc dùng UEFI** để cài Arch Linux theo tài liệu này

### Legacy BIOS (CSM — Compatibility Support Module)

Chế độ tương thích ngược với BIOS cũ. Đặc điểm:

- Dùng bảng phân vùng **MBR** (giới hạn 2TB, tối đa 4 primary partition)
- Boot chậm hơn
- Không hỗ trợ Secure Boot
- **Không dùng** cho cài đặt này

## Vào BIOS và Boot Menu

| Thao tác | Phím | Thời điểm |
|---|---|---|
| Vào BIOS | **F2** (hoặc Fn+F2) | Nhấn liên tục ngay khi nhấn nút nguồn |
| Boot Menu | **F12** (hoặc Fn+F12) | Nhấn liên tục ngay khi nhấn nút nguồn |

**Lưu ý**: Trên Lenovo LOQ 15IAX9, nếu đã bật **Hotkey Mode** thì chỉ cần nhấn F2/F12 (không cần Fn). Nếu tắt Hotkey Mode, cần nhấn Fn+F2 hoặc Fn+F12.

## Các cài đặt bắt buộc

### 1. Tắt Secure Boot

```
BIOS → Security → Secure Boot → Disabled
```

GRUB (bootloader của Arch) không được ký số mặc định, nên Secure Boot sẽ chặn không cho boot. Phải tắt trước khi cài.

### 2. Boot Mode = UEFI

```
BIOS → Boot → Boot Mode → UEFI
```

Không chọn Legacy, Legacy First, hay Both. Chỉ chọn **UEFI** hoặc **UEFI Only**.

### 3. Tắt Fast Boot

```
BIOS → Boot → Fast Boot → Disabled
```

Fast Boot bỏ qua kiểm tra thiết bị USB khi khởi động, gây lỗi không boot được từ USB.

Trên một số phiên bản BIOS Lenovo LOQ, Fast Boot có thể nằm trong:
- `Boot → Fast Boot`
- hoặc `Configuration → Flip to Boot`

Nếu không tìm thấy, kiểm tra trực tiếp bằng cách boot USB sau đó.

### 4. SATA Mode = AHCI

```
BIOS → Configuration → SATA Controller Mode → AHCI
```

RAID mode mặc định trên Lenovo LOQ dành cho Windows + Intel Optane / Intel RST. Linux không hỗ trợ driver RAID của Intel RST mặc định — chuyển sang AHCI để Linux nhận diện ổ NVMe.

**Nếu đã cài Windows với RAID**: Sau khi chuyển sang AHCI, Windows sẽ không boot được (màn hình xanh). Đây là lý do cần backup trước — vì chúng ta sẽ xóa Windows anyway.

### 5. Hotkey Mode (tùy chọn)

```
BIOS → Configuration → Hotkey Mode → Enabled / Disabled
```

| Trạng thái | Tác dụng |
|---|---|
| **Enabled** (mặc định) | Phím F1–F12 hoạt động như phím chức năng đặc biệt (volume, brightness). Muốn dùng F2/F12 cần nhấn Fn+F2 |
| **Disabled** | Phím F1–F12 hoạt động như phím chức năng chuẩn. Nhấn F2 trực tiếp vào BIOS |

Khuyến nghị: Để **Enabled** (mặc định) và dùng Fn+F2 / Fn+F12. Hoặc tắt nếu muốn thao tác nhanh.

## Lưu và thoát

```
BIOS → Nhấn F10 → Yes → Enter
```

Máy sẽ reboot. Nếu không reboot tự động, nhấn nút nguồn để khởi động lại.

## Bảng tác động của từng cài đặt

| Cài đặt | Giá trị | Tác dụng | Hậu quả nếu không chỉnh |
|---|---|---|---|
| Secure Boot | Disabled | Cho phép GRUB (Arch) boot | Arch không boot được — "Security Violation" |
| Boot Mode | UEFI | Dùng GPT, hỗ trợ ổ >2TB | Legacy không boot được ổ NVMe >2TB |
| Fast Boot | Disabled | USB được nhận diện đúng lúc boot | USB boot bị skip — không vào được live env |
| SATA Mode | AHCI | Linux driver chuẩn NVMe hoạt động | Linux không thấy ổ cứng — không có ổ để cài |
| Hotkey Mode | Enabled/Disabled | Tùy chọn thao tác phím Fn | Không ảnh hưởng đến cài đặt |

## Tổng kết

Sau khi hoàn tất cấu hình BIOS:

- [x] Secure Boot = **Disabled**
- [x] Boot Mode = **UEFI**
- [x] Fast Boot = **Disabled**
- [x] SATA Mode = **AHCI**
- [x] Hotkey Mode = **tùy chọn**
- [x] Đã lưu và reboot
