# Tạo USB Boot Arch Linux

## Mục tiêu

Tạo USB boot từ file ISO Arch Linux để khởi động máy và bắt đầu quá trình cài đặt.

## Điều kiện

- USB 4GB+ (sẽ bị format sạch)
- File ISO Arch Linux (đã tải)
- Máy tính có Windows để ghi USB

## Tải Arch Linux ISO

1. Mở trình duyệt, vào: https://archlinux.org/download/
2. Chọn mirror gần Việt Nam nhất:
   - `fcix.vn` (FPT Cloud)
   - hoặc `archlinux.mirror.dkm.cz`
3. Tải file: `archlinux-YYYY.MM.DD-x86_64.iso`
4. Tải kèm file checksum: `sha256sums.txt`

### Kiểm tra checksum (quan trọng)

Trên PowerShell:

```powershell
# Tính checksum file ISO vừa tải
Get-FileHash -Path "D:\Downloads\archlinux-2025.06.01-x86_64.iso" -Algorithm SHA256

# So sánh với nội dung trong sha256sums.txt
# Mở file txt và đối chiếu mã hash
```

Nếu không khớp → ISO bị hỏng → tải lại.

## Cách 1: Rufus (Windows, được khuyên dùng)

1. Cắm USB vào máy.
2. Mở **Rufus** (tải từ https://rufus.ie/ nếu chưa có).
3. Device: chọn USB của bạn.
4. Boot selection: nhấn **SELECT** → chọn file ISO Arch vừa tải.
5. Partition scheme: **GPT**.
6. Target system: **UEFI (non CSM)**.
7. File system: **FAT32** (mặc định).
8. Nhấn **START**.

   Rufus sẽ hỏi:
   - "Write in ISO image mode" → OK
   - "Syslinux warning" → **Write in DD mode** (quan trọng!)

9. Chờ Rufus ghi xong. Báo "READY" là hoàn tất.

## Cách 2: balenaEtcher (đơn giản)

1. Mở balenaEtcher.
2. **Flash from file** → chọn ISO Arch.
3. **Select target** → chọn USB.
4. **Flash!** → Chờ hoàn tất.

Lưu ý: Etcher tự động verify sau khi ghi.

## Cách 3: dd (Linux)

```bash
# Xác định ổ USB
lsblk
# Giả sử USB là /dev/sdb
sudo dd if=archlinux-2025.06.01-x86_64.iso of=/dev/sdb bs=4M status=progress conv=fsync
```

## Sau khi ghi USB

### Trên Windows, kiểm tra cấu trúc USB

Mở File Explorer → USB sẽ có tên **ARCH_2025xx** (hoặc tương tự).
Trong USB có các thư mục: `boot/`, `EFI/`, `arch/`, `loader/`.

Đây là USB boot UEFI hợp lệ.

### Không nên dùng

- **Ventoy**: Có thể gây lỗi với Arch Linux trong một số trường hợp.
  Nếu Ventoy hoạt động thì dùng được, nhưng ưu tiên Rufus/Diskpart.

## Troubleshooting

### Rufus báo "Disk is not enough"

- USB < 4GB → cần USB lớn hơn.
- USB fake dung lượng → dùng h2testw để kiểm tra.

### Rufus không thấy USB

- Cắm USB trực tiếp vào cổng máy (không qua hub).
- Thử cổng USB khác.
- Dùng Disk Management (diskmgmt.msc) → check trạng thái USB.

### ISO không ghi được

- File ISO hỏng → tải lại + kiểm tra checksum.
- USB hỏng → thử USB khác.

## Tổng kết

- Tải ISO từ mirror chính thức.
- Kiểm tra checksum.
- Ghi USB bằng Rufus (DD mode) hoặc balenaEtcher.
- USB sẵn sàng để boot.
