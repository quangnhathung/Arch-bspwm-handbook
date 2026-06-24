# Secure Boot

## Secure Boot là gì?

Secure Boot là cơ chế bảo mật của UEFI, chỉ cho phép bootloader và kernel **đã được ký số hợp lệ** mới được thực thi khi khởi động. Mục đích: ngăn rootkit và bootkit xâm nhập vào quá trình boot.

Windows Boot Manager được Microsoft ký số sẵn nên chạy được. GRUB (Arch) mặc định không được ký.

## Tại sao tắt Secure Boot?

Có thể giữ Secure Boot bật và tự ký bootloader, nhưng:

- Cần tạo key riêng, ký GRUB, ký kernel, và ký lại mỗi khi cập nhật kernel
- Quên ký sau update → máy không boot được
- Không mang lại lợi ích bảo mật đáng kể cho máy tính cá nhân tại nhà
- Tăng độ phức tạp không cần thiết

**Kết luận**: Tắt Secure Boot. Đơn giản, an toàn, không rủi ro.

## Cách tắt Secure Boot

Đã hướng dẫn chi tiết ở bài [02-bios-settings.md](02-bios-settings.md). Tóm tắt:

1. Khởi động máy, nhấn **F2** (hoặc **Fn+F2**) → vào BIOS
2. Tab **Security** → **Secure Boot** → chọn **Disabled**
3. **F10** → **Yes** → Enter để lưu và thoát

## Kiểm tra Secure Boot đã tắt

Sau khi boot vào Arch live USB, kiểm tra:

```bash
bootctl status
```

Đầu ra có dòng `Secure Boot: disabled` là OK.

Hoặc dùng:

```bash
mokutil --sb-state
```

Kết quả: `SecureBoot disabled`. Nếu chưa có `mokutil`:

```bash
pacman -S mokutil
mokutil --sb-state
```

## Có nên bật lại sau khi cài Arch?

### Giữ tắt — khuyến nghị cho máy cá nhân

- Không cần cấu hình thêm
- Không lo quên ký kernel sau update
- Không ảnh hưởng đến bảo mật hàng ngày

### Nên bật lại nếu

- Máy làm server công cộng, kiosk, nhiều người dùng
- Bắt buộc theo chính sách bảo mật công ty / tổ chức
- Máy đặt ở nơi công cộng, dễ bị tiếp cận vật lý

## Cách bật Secure Boot với sbctl

Nếu muốn giữ Secure Boot, dùng `sbctl`:

```bash
# Cài sbctl
pacman -S sbctl

# Tạo key
sbctl create-keys

# Ký GRUB
sbctl sign /boot/efi/EFI/GRUB/grubx64.efi

# Ký kernel
sbctl sign /boot/vmlinuz-linux

# Ký fallback initramfs (nếu có)
sbctl sign /boot/initramfs-linux-fallback.img

# Kiểm tra trạng thái
sbctl status

# Bật Secure Boot trong BIOS và enroll key
sbctl enroll-keys --microsoft
```

Sau đó vào BIOS bật Secure Boot. Mỗi khi update kernel, `sbctl` sẽ tự ký nếu được cấuình trong pacman hook.

Tuy nhiên, cách này **không nằm trong phạm vi tài liệu** — chỉ tham khảo.

## Tổng kết

- Secure Boot phải **tắt** trước khi cài Arch
- Kiểm tra bằng `bootctl status` hoặc `mokutil --sb-state`
- Máy cá nhân nên **giữ tắt**
- Có thể bật lại sau với `sbctl` nếu thực sự cần
