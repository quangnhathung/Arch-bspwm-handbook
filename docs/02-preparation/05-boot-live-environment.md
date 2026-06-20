# Secure Boot

## Giới thiệu

Secure Boot là một tính năng của UEFI giúp ngăn chặn malware và bootkit bằng cách
chỉ cho phép bootloader và kernel đã được ký số hợp lệ mới được thực thi khi
khởi động máy.

## Tại sao tắt Secure Boot?

### Arch Linux và Secure Boot

Arch Linux hỗ trợ Secure Boot nhưng không cài đặt mặc định. Để dùng Secure Boot
với Arch, cần:

1. Tự ký GRUB (hoặc bootloader) bằng key riêng.
2. Nạp key đó vào UEFI firmware.
3. Ký kernel và initramfs mỗi khi cập nhật.

Đây là quy trình thủ công, dễ sai, và không cần thiết cho desktop cá nhân.
Trong tài liệu này, chúng ta **tắt Secure Boot**.

## Cách tắt Secure Boot trên Lenovo LOQ 15IAX9

Đã hướng dẫn chi tiết ở bài BIOS settings. Tóm tắt:

1. Vào BIOS (F2 khi khởi động).
2. Tab **Security** → **Secure Boot**.
3. Chọn **Disabled**.
4. F10 → Save & Exit.

### Xác nhận Secure Boot đã tắt

Sau khi boot vào Arch live USB, chạy:

```bash
bootctl status
```

Nếu thấy `Secure Boot: disabled` là OK.

Hoặc dùng:

```bash
mokutil --sb-state
```

Nếu chưa có mokutil: `pacman -S mokutil` (trong live environment).

## Có nên bật lại sau khi cài xong?

### Lý do KHÔNG nên bật

- Phải cấu hình ký số tự động mỗi khi cập nhật kernel.
- Nếu quên ký, máy sẽ không boot được.
- Không có lợi ích thực tế đáng kể cho máy tính cá nhân dùng tại nhà.

### Khi nào nên bật

- Máy làm server công cộng / kiosk / nhiều người dùng.
- Bắt buộc theo chính sách bảo mật công ty.
- Máy thường xuyên bị tiếp cận vật lý bởi người lạ.

## Nếu muốn bật Secure Boot sau này

Tham khảo Arch Wiki: [Unified Extensible Firmware Interface/Secure Boot](https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface/Secure_Boot)

Quy trình tổng quan:

```bash
# Cài shim và sbsigntools
pacman -S shim sbsigntools

# Tạo key riêng
sudo sbctl create-keys

# Ký GRUB
sudo sbctl sign /boot/efi/EFI/GRUB/grubx64.efi

# Ký kernel
sudo sbctl sign /boot/vmlinuz-linux

# Bật Secure Boot trong BIOS
# Nạp key vào firmware
```

Nhưng đây không nằm trong phạm vi tài liệu hiện tại.

## Kết luận

- Secure Boot cần tắt trước khi cài Arch.
- Có thể bật lại sau nhưng cần cấu hình thêm.
- Với máy cá nhân, giữ tắt là an toàn và tiện lợi nhất.
