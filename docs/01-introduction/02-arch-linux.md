# Arch Linux là gì?

Arch Linux là bản phân phối Linux **rolling release** theo triết lý KISS (Keep It Simple, Stupid). Không như Ubuntu hay Fedora, Arch không có trình cài đồ họa, không có desktop mặc định, không công cụ cấu hình tự động. Bạn làm mọi thứ từ terminal.

## Đặc điểm chính

### Rolling Release

Không có phiên bản "Arch 2025" hay "Arch 2026". Cập nhật liên tục. Chạy `pacman -Syu` là cập nhật toàn bộ hệ thống bao gồm kernel, driver, ứng dụng — tất cả một lệnh.

| Ưu điểm | Nhược điểm |
|---------|-----------|
| Luôn dùng phần mềm mới nhất | Cập nhật có thể gây hỏng nếu không theo dõi |
| Không cần nâng cấp phiên bản | Phải đọc archlinux.org/news trước update |
| Cài một lần, update mãi mãi | Kernel mới đôi khi conflict với driver |

### Pacman

Trình quản lý gói tích hợp, cú pháp đơn giản:

```bash
pacman -Syu          # Cập nhật hệ thống
pacman -S <gói>      # Cài gói
pacman -R <gói>      # Xóa gói
pacman -Qs <từ>      # Tìm gói đã cài
pacman -Qdt          # Liệt kê gói orphan
```

### AUR — Arch User Repository

Kho cộng đồng chứa phần mềm không có trong repo chính thức — bao gồm cả driver Realtek RTL8852BE, EnvyControl, và nhiều tool khác dùng trong handbook này.

Truy cập AUR qua `yay` (có hướng dẫn cài ở bài 06-package-management):

```bash
yay -S <gói>         # Cài từ AUR
yay -Syu             # Cập nhật cả repo + AUR
```

### Arch Wiki

**Arch Wiki là wiki Linux tốt nhất thế giới.** Bất kỳ vấn đề nào — từ cấu hình NVIDIA Optimus đến sửa lỗi GRUB — đều có giải pháp ở đó. Luôn tra wiki trước khi hỏi Google.

## Tại sao nên dùng Arch?

- **Học được nhiều**: Bạn hiểu Linux từ gốc, không bị công cụ tự động che giấu
- **Tối ưu**: Chỉ cài những gì cần. Hệ thống nhẹ, nhanh, không bloatware
- **Linh hoạt**: Arch làm theo ý bạn, không làm theo ý nhà phát triển
- **Cộng đồng**: Wiki + diễn đàn + Reddit rất chất lượng

## Tại sao không nên dùng Arch?

- **Tốn thời gian ban đầu**: Có thể mất cả ngày cho lần cài đầu tiên
- **Không tự động**: Lỗi — tự sửa. Không có "click để sửa"
- **Yêu cầu đọc hiểu**: Không thể cài Arch mà không đọc tài liệu
- **Rủi ro update**: Kernel mới có thể làm hỏng driver NVIDIA hoặc Wi-Fi

## Kết luận

Arch Linux phù hợp nếu bạn muốn hiểu Linux thực sự, có thời gian tìm hiểu, và muốn hệ thống tinh gọn đúng nhu cầu. Không phù hợp nếu bạn muốn máy "chạy ngay sau cài" không cần cấu hình.

> Mẹo: Luôn đọc https://archlinux.org/news trước khi `pacman -Syu`.
> Subscribe RSS để không bị bất ngờ.
