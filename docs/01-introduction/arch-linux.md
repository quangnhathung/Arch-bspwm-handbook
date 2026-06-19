# Arch Linux là gì?

Arch Linux là một bản phân phối Linux **rolling release** với triết lý KISS
(Keep It Simple, Stupid). Không giống Ubuntu hay Fedora, Arch không có trình
cài đặt đồ họa, không có môi trường desktop mặc định, và không có công cụ
cấu hình tự động.

## Đặc điểm chính

### 1. Rolling Release

Không có phiên bản "Arch Linux 2025" hay "Arch Linux 2026". Cập nhật liên tục.
Khi bạn chạy `pacman -Syu`, toàn bộ hệ thống được cập nhật lên bản mới nhất,
bao gồm kernel, driver, ứng dụng — tất cả trong một lệnh duy nhất.

**Lợi ích**: Luôn dùng phần mềm mới nhất. Không cần nâng cấp phiên bản như Ubuntu.
**Rủi ro**: Cập nhật có thể gây hỏng nếu không đọc tin tức trước khi update.

### 2. Pacman — Trình quản lý gói

Pacman là công cụ quản lý gói tích hợp sẵn. Cú pháp đơn giản:

```bash
pacman -Syu    # Cập nhật toàn bộ hệ thống
pacman -S gói  # Cài gói
pacman -R gói  # Xóa gói
pacman -Qs từ  # Tìm gói đã cài
```

### 3. AUR — Arch User Repository

Kho lưu trữ cộng đồng do người dùng đóng góp. Chứa phần mềm không có trong
repo chính thức. Dùng yay hoặc paru để truy cập:

```bash
yay -S gói     # Cài từ AUR
```

### 4. Wiki

Arch Wiki được coi là wiki Linux tốt nhất thế giới. Bất kỳ vấn đề nào bạn gặp,
khả năng cao đã có giải pháp trên Arch Wiki.

### 5. Tự cấu hình hoàn toàn

Arch không làm gì tự động thay bạn. Bạn tự quyết định:

- Dùng systemd-boot hay GRUB
- Dùng NetworkManager hay systemd-networkd
- Dùng PipeWire hay PulseAudio
- Dùng ext4, BTRFS, XFS, ZFS
- Dùng desktop environment, window manager, hay cả hai

## Tại sao nên dùng Arch?

- **Học được nhiều**: Bạn sẽ hiểu Linux từ gốc thay vì dùng công cụ tự động.
- **Tối ưu**: Bạn chỉ cài những gì mình cần. Hệ thống nhẹ và nhanh.
- **Linh hoạt**: Arch làm theo ý bạn, không làm theo ý nhà phát triển.
- **Cộng đồng**: Arch Wiki và diễn đàn cực kỳ chất lượng.

## Tại sao không nên dùng Arch?

- **Tốn thời gian ban đầu**: Có thể mất cả ngày để cài và cấu hình.
- **Không tự động**: Mọi lỗi đều do bạn tự sửa.
- **Yêu cầu đọc hiểu**: Không thể cài Arch mà không đọc tài liệu.
- **Cập nhật có rủi ro**: Đôi khi bản cập nhật gây hỏng và cần can thiệp thủ công.

## Kết luận

Arch Linux phù hợp nếu bạn muốn hiểu Linux thực sự, có thời gian tìm hiểu,
và muốn một hệ thống tinh gọn đúng nhu cầu. Không phù hợp nếu bạn muốn
máy tính "chạy ngay sau khi cài" mà không cần cấu hình.
