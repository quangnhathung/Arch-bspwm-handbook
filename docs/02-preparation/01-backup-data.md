# Sao lưu dữ liệu

## Cảnh báo trước khi bắt đầu

Quá trình cài đặt Arch Linux sẽ **xóa sạch toàn bộ dữ liệu hiện tại** trên ổ cứng.

| Sẽ bị xóa | Sẽ giữ lại |
|---|---|
| Windows và tất cả ứng dụng | Dữ liệu trên USB / ổ cứng ngoài |
| Tài liệu, nhạc, video trên ổ C: | Dữ liệu đã backup sang nơi khác |
| Cài đặt, driver, license | — |
| Phân vùng ổ cứng gốc | — |
| Recovery partition | — |

## Không thể khôi phục

Những thứ sẽ mất vĩnh viễn nếu không backup:

- **Windows license**: Key Windows thường được nhúng trong firmware (OA3).
  Sau khi cài Arch, bạn vẫn có key Windows gốc nếu cần cài lại sau này.
  Kiểm tra trước:
  ```powershell
  # Trên Windows, mở PowerShell Admin và chạy:
  wmic path SoftwareLicensingService get OA3xOriginalProductKey
  ```
  Ghi lại key này nếu bạn muốn.

- **Driver laptop**: Driver Lenovo, NVIDIA, Realtek không dùng được trên Linux.
  Không cần backup driver.

- **File cá nhân**: Cần backup thủ công.

## Dữ liệu cần backup

### Trên Windows

1. **Documents, Desktop, Downloads, Pictures, Videos, Music**
   → Copy sang ổ cứng ngoài hoặc cloud.

2. **Bookmarks trình duyệt**
   → Chrome/Edge/Firefox đều có chức năng export bookmark.

3. **Mật khẩu và key**
   - Mật khẩu Wi-Fi (nếu có)
   - Key SSH nếu bạn có
   - Token / 2FA recovery codes

4. **License phần mềm**
   - Windows license key (ghi lại)
   - License phần mềm mua riêng

5. **Database trình duyệt (tùy chọn)**
   - Password manager export (Bitwarden, KeePass, etc.)

### Cách backup nhanh

```powershell
# Mở PowerShell Admin trên Windows

# Tạo thư mục backup trên ổ D: hoặc USB ngoài
New-Item -ItemType Directory -Path "D:\backup-$(Get-Date -Format 'yyyyMMdd')"

# Copy thư mục người dùng
Copy-Item "$env:USERPROFILE\Documents" "D:\backup-$(Get-Date -Format 'yyyyMMdd')\" -Recurse
Copy-Item "$env:USERPROFILE\Desktop" "D:\backup-$(Get-Date -Format 'yyyyMMdd')\" -Recurse
Copy-Item "$env:USERPROFILE\Downloads" "D:\backup-$(Get-Date -Format 'yyyyMMdd')\" -Recurse
Copy-Item "$env:USERPROFILE\Pictures" "D:\backup-$(Get-Date -Format 'yyyyMMdd')\" -Recurse
Copy-Item "$env:USERPROFILE\Videos" "D:\backup-$(Get-Date -Format 'yyyyMMdd')\" -Recurse
Copy-Item "$env:USERPROFILE\Music" "D:\backup-$(Get-Date -Format 'yyyyMMdd')\" -Recurse
```

## Kiểm tra trước khi mất Windows

- Mở File Explorer và xác nhận các file quan trọng đã có bản sao.
- Boot thử USB Arch để kiểm tra máy boot được từ USB không (xem bài tiếp theo).
- Nếu máy không boot USB, cần vào BIOS chỉnh sửa trước khi xóa Windows.

## Sau khi backup

1. Rút ổ cứng ngoài / USB chứa backup ra khỏi máy.
2. Để sang một bên, không cắm trong lúc cài Arch.
3. Chuẩn bị USB boot Arch (bài tiếp theo).

## Tổng kết

- Backup tất cả dữ liệu quan trọng.
- Ghi lại Windows key (nếu cần).
- Kiểm tra USB boot trước khi xóa Windows.
- Sau khi xóa, KHÔNG có đường quay lại.
