# Sao lưu dữ liệu trước khi cài đặt

## Cảnh báo

Quá trình cài đặt Arch Linux sẽ **xóa sạch toàn bộ ổ cứng**. Mọi dữ liệu trên máy sẽ mất vĩnh viễn nếu không backup trước.

| Sẽ bị xóa | Sẽ giữ lại |
|---|---|
| Windows và toàn bộ ứng dụng | Dữ liệu trên USB / ổ cứng gắn ngoài |
| Tài liệu, nhạc, video, ảnh trên ổ C: | Dữ liệu đã sao chép sang thiết bị khác |
| Cài đặt hệ thống, driver Windows | Windows license key (nhúng trong firmware) |
| Phân vùng ổ cứng gốc | Dữ liệu trên cloud / NAS |
| Recovery partition của OEM | — |

## Lấy Windows license key

Máy Lenovo LOQ 15IAX9 có key Windows được nhúng sẵn trong firmware (OA3). Sau khi cài Arch, key này vẫn còn trong UEFI — có thể dùng lại nếu cần cài Windows sau này.

Mở **PowerShell (Admin)** và chạy:

```powershell
wmic path SoftwareLicensingService get OA3xOriginalProductKey
```

Ghi lại key hiện ra. Nếu không thấy key (trống), máy dùng license kỹ thuật số liên kết với tài khoản Microsoft — không cần key.

## Danh sách dữ liệu cần backup

### Thư mục người dùng

| Thư mục | Vị trí mặc định trên Windows |
|---|---|
| Documents | `C:\Users\<tên>\Documents` |
| Desktop | `C:\Users\<tên>\Desktop` |
| Downloads | `C:\Users\<tên>\Downloads` |
| Pictures | `C:\Users\<tên>\Pictures` |
| Videos | `C:\Users\<tên>\Videos` |
| Music | `C:\Users\<tên>\Music` |

### Trình duyệt web

- **Google Chrome**: `C:\Users\<tên>\AppData\Local\Google\Chrome\User Data\Default\Bookmarks` — hoặc dùng Chrome Settings → Bookmarks → Export
- **Firefox**: Ctrl+Shift+O → Import/Export → Export Bookmarks
- **Edge**: Settings → Favorites → Export

### Mật khẩu và khóa bảo mật

- Export mật khẩu từ trình quản lý mật khẩu (Bitwarden, KeePass, 1Password)
- Copy thư mục `.ssh` nếu có: `C:\Users\<tên>\.ssh`
- Recovery codes của 2FA (GitHub, Google, Discord, etc.)
- Mật khẩu Wi-Fi của mạng tại nhà
- Token / API key đã lưu trong file text

### Dữ liệu khác

- File dự án, source code (thường ở `C:\Users\<tên>\source` hoặc `C:\Projects`)
- File cấu hình ứng dụng (`.gitconfig`, file cài đặt IDE)
- Email local nếu dùng Thunderbird / Outlook

## Lệnh backup bằng PowerShell

Mở **PowerShell (Admin)** và chạy:

```powershell
# Tạo thư mục backup trên ổ D: hoặc USB ngoài
$backupDir = "D:\backup-$(Get-Date -Format 'yyyyMMdd')"
New-Item -ItemType Directory -Path $backupDir -Force

# Copy các thư mục người dùng
$folders = @("Documents", "Desktop", "Downloads", "Pictures", "Videos", "Music")
foreach ($folder in $folders) {
    $src = "$env:USERPROFILE\$folder"
    if (Test-Path $src) {
        Write-Host "Đang backup $folder ..."
        Copy-Item "$src\*" "$backupDir\$folder\" -Recurse -Force
    }
}

# Copy thư mục .ssh
if (Test-Path "$env:USERPROFILE\.ssh") {
    Copy-Item "$env:USERPROFILE\.ssh" "$backupDir\" -Recurse -Force
}

# Copy bookmarks Chrome
$chromeBookmark = "$env:LOCALAPPDATA\Google\Chrome\User Data\Default\Bookmarks"
if (Test-Path $chromeBookmark) {
    Copy-Item $chromeBookmark "$backDir\chrome-bookmarks.html"
}

Write-Host "Hoàn tất backup tại: $backupDir"
```

**Thủ công**: Cũng có thể kéo-thả thư mục từ File Explorer sang USB.

## Kiểm tra backup

Trước khi xóa Windows, mở ổ backup và kiểm tra:

- File PDF, docx có mở được không
- Ảnh có xem được không
- Dung lượng backup có hợp lý không (không bị 0KB)
- Kiểm tra bookmarks đã export được chưa

## Chuẩn bị USB boot

Sau khi backup xong, cần chuẩn bị USB để cài Arch:

- USB tối thiểu 4GB (sẽ bị format sạch)
- Có thể dùng chính USB vừa backup nếu dung lượng đủ
- Tốc độ USB 3.0 trở lên khuyến khích
- Xem hướng dẫn tạo USB boot ở bài 04-create-usb

## Lưu ý

- **Rút USB/ổ cứng chứa backup ra khỏi máy** trước khi cài Arch — tránh ghi đè nhầm.
- Nếu có 2 ổ cứng vật lý, ngắt kết nối ổ chứa backup để an toàn tuyệt đối.

## Tổng kết

- Ghi lại Windows license key (nếu có)
- Backup Documents, Desktop, Downloads, Pictures, Videos, Music
- Export bookmarks, mật khẩu, SSH keys
- Chạy PowerShell script hoặc copy thủ công
- Kiểm tra file backup trước khi mất Windows
- Rút thiết bị backup ra sau khi hoàn tất
