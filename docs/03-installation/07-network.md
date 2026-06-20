# Network

## Mục tiêu

Cấu hình NetworkManager để quản lý kết nối mạng (Wi-Fi, LAN).

## Kiến thức nền

### NetworkManager là gì?

NetworkManager là dịch vụ quản lý kết nối mạng mặc định của hầu hết các bản
phân phối Linux hiện đại. Nó hỗ trợ:

- Wi-Fi (qua wpa_supplicant hoặc iwd backend)
- Ethernet (DHCP tự động)
- USB tethering (Android, iPhone)
- VPN
- nmcli (command line), nmtui (terminal UI), nm-applet (GUI)

### Cách NetworkManager hoạt động

1. Dịch vụ `NetworkManager.service` chạy nền.
2. Quét các interface mạng có sẵn (wlan0, eth0, v.v.).
3. Tự động kết nối đến mạng đã lưu.
4. Cho phép người dùng quản lý kết nối qua nmcli/nmtui.

## Các bước thực hiện

### Bước 1: Enable NetworkManager

```bash
systemctl enable NetworkManager
```

Giải thích: `systemctl enable` tạo symlink để dịch vụ tự động khởi động
khi boot. Chưa start vì chưa out khỏi chroot.

### Bước 2: Cấu hình hostname (đã làm ở bài trước)

Kiểm tra:

```bash
cat /etc/hostname
```

Phải có `loq-arch` hoặc hostname bạn muốn.

### Bước 3: (Tùy chọn) Cấu hình iwd làm backend cho Wi-Fi

NetworkManager có thể dùng iwd (iNet Wireless Daemon) thay vì wpa_supplicant
mặc định. iwd nhẹ hơn và nhanh hơn.

```bash
pacman -S iwd
```

Kích hoạt iwd:

```bash
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

### Bước 4: Kiểm tra cấu hình hiện tại (trong chroot)

NetworkManager chưa chạy trong chroot nên không test được.
Sau khi reboot, kiểm tra:

```bash
nmcli device status
nmcli device wifi list
nmcli device wifi connect "SSID" password "password"
```

## Lệnh NetworkManager cơ bản

### Kết nối Wi-Fi

```bash
# Liệt kê mạng có sẵn
nmcli device wifi list

# Kết nối đến mạng
nmcli device wifi connect "MyWiFi" password "mypassword"

# Kết nối đến mạng ẩn
nmcli device wifi connect "MyWiFi" password "mypassword" hidden yes
```

### Quản lý kết nối

```bash
# Xem trạng thái thiết bị
nmcli device status

# Xem kết nối đã lưu
nmcli connection show

# Xóa kết nối
nmcli connection delete "MyWiFi"

# Disconnect
nmcli device disconnect wlan0

# Reconnect
nmcli device connect wlan0
```

### nmtui (giao diện terminal)

```bash
nmtui
```

Giao diện text đơn giản, dùng phím mũi tên + Enter để thao tác.

## Xử lý Wi-Fi cho Realtek RTL8852BE

Trong live environment, Wi-Fi có thể không hoạt động vì thiếu firmware.
Sau khi cài base, chúng ta sẽ xử lý driver Realtek sau (xem docs/05-drivers/wifi.md).

Trong lúc cài đặt base, hãy dùng một trong các cách sau:

1. **LAN cable** (nếu có) — tự động nhận DHCP.
2. **USB tethering Android** — bật USB Tethering trên điện thoại.
3. **USB tethering iPhone** — bật Personal Hotspot → USB Only.

## Tổng kết

- NetworkManager đã được enable để tự động khởi động.
- Sẵn sàng reboot và dùng nmcli/nmtui để kết nối mạng.
