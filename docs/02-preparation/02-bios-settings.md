# Cấu hình BIOS

## Các khái niệm nền tảng

### UEFI là gì?

UEFI (Unified Extensible Firmware Interface) là chuẩn firmware hiện đại thay thế
Legacy BIOS. UEFI hỗ trợ:

- Ổ cứng GPT (thay vì MBR)
- Boot nhanh hơn
- Giao diện đồ họa
- Secure Boot
- Boot manager tích hợp

Hầu hết máy tính từ 2012+ đều dùng UEFI. Lenovo LOQ 15IAX9 mặc định dùng UEFI.

### Legacy Boot là gì?

Legacy Boot (CSM — Compatibility Support Module) là chế độ tương thích với BIOS cũ.
Dùng MBR (Master Boot Record). Hạn chế: không boot được ổ >2TB, chậm, cũ.

**Không dùng Legacy.** Chúng ta cài UEFI.

### Secure Boot là gì?

Secure Boot là cơ chế của UEFI chỉ cho phép chạy bootloader đã được ký số.
Mặc định, Windows Boot Manager được ký, GRUB (Arch) thì không.

**Phải tắt Secure Boot** để Arch boot được. Có thể bật lại sau khi cài shim
nhưng phức tạp, không cần thiết cho máy cá nhân.

### Fast Boot là gì?

Fast Boot là tính năng rút ngắn thời gian khởi động bằng cách bỏ qua kiểm tra
một số thiết bị USB và boot option. Gây khó khăn khi boot USB live.

**Nên tắt Fast Boot** để USB boot hoạt động đáng tin cậy.

## Vào BIOS Lenovo LOQ 15IAX9

### Cách vào BIOS

1. Tắt máy hoàn toàn (Shut down, không Restart).
2. Nhấn nút **Nguồn**.
3. Ngay khi màn hình sáng, nhấn liên tục phím **F2** (hoặc **Fn + F2**) cho đến khi vào BIOS.
4. Nếu F2 không vào được, thử **F1** (một số đời Lenovo).

### Cách vào Boot Menu (chọn USB boot)

- Nhấn **F12** khi khởi động → hiện danh sách thiết bị boot.

## Các cài đặt cần thay đổi

### 1. Tắt Secure Boot

```
Vào tab Security → Secure Boot → Set thành Disabled
```

### 2. Chỉnh Boot Mode về UEFI

```
Vào tab Boot → Boot Mode:
  - Chọn UEFI (có thể là "UEFI Only")
  - KHÔNG chọn Legacy Support
  - KHÔNG chọn Both
```

### 3. Tắt Fast Boot

```
Vào tab Boot → Fast Boot → Set thành Disabled
```

**Trên Lenovo LOQ**, Fast Boot có thể nằm trong:
- `Boot → Fast Boot`
- hoặc `Configuration → Flip to Boot`

Nếu không thấy, giữ nguyên. Kiểm tra boot USB sau đó.

### 4. Chỉnh SATA mode thành AHCI (nếu có RAID)

```
Vào tab Configuration → SATA Controller Mode → Chọn AHCI
```

RAID mode thường chỉ dùng cho Windows + Intel Optane.
Chúng ta không cần RAID.

### 5. Tắt Intel Optane Memory (nếu có)

```
Vào tab Configuration → Intel Optane Memory → Disable
```

Lenovo LOQ 15IAX9 có thể không có Optane, nhưng kiểm tra cho chắc.

### 6. Kiểm tra tab Boot Order

Đảm bảo USB được ưu tiên hoặc dùng F12 để chọn tạm thời.
Không cần thay đổi boot order nếu dùng F12.

### 7. Lưu và thoát

```
Nhấn F10 → Yes → Enter
```

Máy sẽ reboot.

## Ảnh hưởng của các thay đổi

| Cài đặt | Tác dụng | Tác hại nếu không chỉnh |
|---|---|---|
| Tắt Secure Boot | Cho phép GRUB boot | Không boot được Arch |
| UEFI mode | Dùng GPT, boot nhanh | Legacy không boot được ổ lớn |
| Tắt Fast Boot | USB được nhận diện đúng | USB boot bị lỗi |
| AHCI | Tương thích Linux | Linux không thấy ổ NVMe |

## Tổng kết

Sau khi cấu hình BIOS xong:
- Tắt Secure Boot
- Boot Mode = UEFI
- Fast Boot = Disabled
- SATA = AHCI
- Lưu và reboot
