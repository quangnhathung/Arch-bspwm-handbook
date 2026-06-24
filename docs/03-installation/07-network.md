# Network

## Mục tiêu

Cấu hình NetworkManager để quản lý kết nối mạng (Wi-Fi, LAN, USB tethering).

## Điều kiện tiên quyết

- Đã chroot vào hệ thống mới (`arch-chroot /mnt`).
- Đã cấu hình hostname.

## Kiến thức nền

### NetworkManager là gì?

NetworkManager là dịch vụ quản lý mạng mặc định của Arch Linux (và hầu hết các
bản phân phối hiện đại). Nó hỗ trợ:

- **Wi-Fi** (qua wpa_supplicant hoặc iwd backend).
- **Ethernet** (DHCP tự động).
- **USB tethering** (Android, iPhone).
- **VPN** (WireGuard, OpenVPN).
- **nmcli** (command line), **nmtui** (terminal UI), **nm-applet** (system tray).

### Cách NetworkManager hoạt động

1. Dịch vụ `NetworkManager.service` chạy nền (systemd).
2. Quét các interface mạng có sẵn (`wlan0`, `eth0`, ...).
3. Tự động kết nối đến mạng đã lưu.
4. DHCP tự động nếu mạng hỗ trợ.

---

## Cách A: Cài thủ công

### Bước 1: Enable NetworkManager

```bash
systemctl enable NetworkManager
```

Giải thích:
- `systemctl enable`: Tạo symlink để dịch vụ tự động khởi động khi boot.
- Chưa `start` vì đang trong chroot (không có systemd chạy thật).

### Bước 2: (Tùy chọn) Cài iwd làm backend cho Wi-Fi

Mặc định NetworkManager dùng `wpa_supplicant` cho Wi-Fi. `iwd` (iNet Wireless
Daemon) là thay thế nhẹ hơn, nhanh hơn.

```bash
pacman -S iwd
systemctl enable iwd
```

Cấu hình NetworkManager dùng iwd:
```bash
mkdir -p /etc/NetworkManager/conf.d
vim /etc/NetworkManager/conf.d/wifi-backend.conf
```

Nội dung:
```ini
[device]
wifi.backend=iwd
```

### Bước 3: Kiểm tra cấu hình

```bash
cat /etc/hostname
```

Đảm bảo hostname đã được đặt (bài 06-locale).

### Bước 4: Sau reboot

Sau khi reboot vào hệ thống mới, kết nối mạng:

```bash
# Kiểm tra trạng thái
nmcli device status

# Quét Wi-Fi
nmcli device wifi list

# Kết nối Wi-Fi
nmcli device wifi connect "TenWifi" password "matkhau"
```

---

## Cách B: Dùng Archinstall

Archinstall cài NetworkManager theo mặc định.

### Trong quá trình archinstall

Khi đến mục **`Network configuration`**:
1. Chọn **`NetworkManager`** (mặc định).
2. Archinstall tự động enable NetworkManager.
3. Không cần cấu hình gì thêm.

### Kiểm tra sau archinstall

```bash
arch-chroot /mnt
systemctl is-enabled NetworkManager
# Output: enabled
```

---

## Lệnh NetworkManager cơ bản (sau reboot)

### Kết nối Wi-Fi

```bash
# Liệt kê mạng Wi-Fi có sẵn
nmcli device wifi list

# Kết nối đến mạng (lưu tự động)
nmcli device wifi connect "SSID" password "password"

# Kết nối đến mạng ẩn
nmcli device wifi connect "SSID" password "password" hidden yes
```

### Quản lý kết nối

```bash
# Xem trạng thái thiết bị
nmcli device status

# Xem kết nối đã lưu
nmcli connection show

# Xóa kết nối
nmcli connection delete "SSID"

# Disconnect
nmcli device disconnect wlan0

# Reconnect
nmcli device connect wlan0
```

### nmtui — Giao diện terminal

```bash
nmtui
```

Giao diện text đơn giản dùng phím mũi tên + Enter. Phù hợp cho người không
nhớ cú pháp nmcli.

---

## Xử lý Wi-Fi cho Realtek RTL8852BE

Lenovo LOQ 15IAX9 dùng chip Wi-Fi **Realtek RTL8852BE**.

### Trong live environment

Khi boot Arch ISO:
- Wi-Fi có thể không hoạt động vì driver Realtek không có sẵn.
- Dùng một trong các cách sau để có mạng:
  1. **LAN cable** (cắm dây mạng) — tự động DHCP.
  2. **Android USB tethering** — Bật USB Tethering trên điện thoại.
  3. **iPhone USB tethering** — Bật Personal Hotspot → USB Only.

### Sau khi cài base

Driver cho RTL8852BE không có sẵn trong kernel (cần driver từ AUR).
Xem hướng dẫn chi tiết tại `docs/05-drivers/05-wifi.md`.

Sau khi cài driver, Wi-Fi hoạt động bình thường qua NetworkManager.

---

## Xác minh network hoạt động

Sau reboot, kiểm tra:

```bash
# Kiểm tra NetworkManager đang chạy
systemctl status NetworkManager

# Kiểm tra kết nối Internet
ping -c 3 archlinux.org

# Kiểm tra IP
ip addr show

# Kiểm tra DNS
resolvectl status
```

---

## Tổng kết

- NetworkManager đã được enable.
- Cấu hình iwd backend (tùy chọn).
- Wi-Fi Realtek RTL8852BE cần cài driver từ AUR sau khi có desktop.
- Sẵn sàng tạo user và cài bootloader.
