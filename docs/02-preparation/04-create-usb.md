# Tạo USB Boot Arch Linux

## Yêu cầu

- USB tối thiểu **4GB** (sẽ bị format sạch hoàn toàn)
- File ISO Arch Linux (khoảng 900MB)
- Máy tính có Windows (hoặc Linux) để ghi USB

## Bước 1: Tải Arch Linux ISO

Truy cập: [https://archlinux.org/download/](https://archlinux.org/download/)

Chọn mirror gần Việt Nam:

- **FPT Cloud** (fcix.vn) — nhanh nhất
- hoặc **kmweb** (Hàn Quốc)

Tải 2 file:

- `archlinux-2026.06.01-x86_64.iso` (tên file tùy theo ngày phát hành)
- `sha256sums.txt`

## Bước 2: Kiểm tra checksum

Mở **PowerShell** trong thư mục chứa file ISO:

```powershell
Get-FileHash -Path ".\archlinux-2026.06.01-x86_64.iso" -Algorithm SHA256
```

Mở file `sha256sums.txt` và so sánh mã hash. Nếu không khớp:

- ISO bị hỏng trong quá trình tải
- Tải lại từ mirror khác

## Bước 3: Ghi USB

### Cách 1: Rufus (Windows — khuyến nghị)

1. Tải Rufus từ [https://rufus.ie](https://rufus.ie) (bản portable, không cần cài)
2. Cắm USB vào máy
3. Mở Rufus:

| Trường | Giá trị |
|---|---|
| Device | USB của bạn |
| Boot selection | Chọn file ISO Arch Linux |
| Partition scheme | **GPT** |
| Target system | **UEFI (non CSM)** |
| File system | FAT32 (mặc định) |

4. Nhấn **START**

5. Rufus sẽ hiện các cảnh báo:

   - *"Write in ISO image mode"* → OK
   - *"Syslinux warning"* / *"Image is not DD-writable"* → **Write in DD mode** (cực kỳ quan trọng)

   > DD mode ghi ISO raw bit-by-bit, tạo USB boot UEFI đúng chuẩn. Nếu chọn "Write in ISO mode", USB sẽ không boot được trên UEFI.

6. Chờ Rufus ghi xong (2–5 phút). Màn hình báo **READY**.

### Cách 2: balenaEtcher (Windows / macOS / Linux)

1. Tải từ [https://etcher.balena.io](https://etcher.balena.io)
2. **Flash from file** → chọn file ISO
3. **Select target** → chọn USB
4. **Flash!**
5. Etcher tự động verify sau khi ghi

**Ưu điểm**: Đơn giản, tự động verify. **Nhược điểm**: Không có tùy chọn DD mode — có thể lỗi với một số USB.

### Cách 3: dd (Linux)

```bash
# Xác định ổ USB
lsblk
# Giả sử USB là /dev/sdb (kiểm tra kỹ, không nhầm với ổ cứng chính!)

# Ghi ISO
sudo dd if=archlinux-2026.06.01-x86_64.iso of=/dev/sdb bs=4M status=progress conv=fsync
```

## Kiểm tra USB sau khi ghi

Cắm USB vào máy, mở File Explorer (Windows) hoặc lsblk (Linux):

```
USB label: ARCH_202606
Cấu trúc thư mục:
├── boot/
├── EFI/
├── arch/
├── loader/
└── (các file khác)
```

Nếu thấy cấu trúc này → USB boot UEFI hợp lệ.

## Troubleshooting

### USB không xuất hiện trong Rufus

- Cắm USB trực tiếp vào cổng trên máy, không qua hub
- Thử cổng USB khác (ưu tiên USB 2.0 nếu có)
- Mở `diskmgmt.msc` → kiểm tra USB có hiện trong Disk Management không
- USB hỏng → thử USB khác

### Lỗi ghi không thành công

- ISO hỏng → tải lại + kiểm tra checksum
- USB bị lỗi bad sector → dùng `h2testw` (Windows) để kiểm tra
- USB giả dung lượng (fake 64GB nhưng thực tế 4GB) → dùng `h2testw` hoặc `ChipGenius`
- Thử cổng USB 2.0 thay vì 3.0
- Dùng Rufus **DD mode** — ISO mode gây lỗi boot

### USB đã ghi nhưng không boot được

- Vào BIOS kiểm tra Secure Boot đã tắt chưa
- Boot Mode có đang là UEFI không
- Fast Boot đã tắt chưa
- Ghi lại USB với DD mode
- Thử balenaEtcher nếu Rufus không được

## Tổng kết

- [x] Tải ISO từ mirror chính thức
- [x] Kiểm tra checksum SHA256
- [x] Ghi USB bằng Rufus (DD mode) hoặc balenaEtcher
- [x] Kiểm tra cấu trúc USB có đúng UEFI
- [x] USB sẵn sàng để boot
