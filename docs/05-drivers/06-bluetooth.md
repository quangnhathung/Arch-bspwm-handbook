# Bluetooth

## Mục tiêu

Cài đặt và cấu hình Bluetooth trên laptop.

## Kiến thức nền

### Bluetooth trên Lenovo LOQ

Bluetooth trên máy này đi kèm với chip Wi-Fi Realtek RTL8852BE.
Nó hỗ trợ Bluetooth 5.2.

### BlueZ

BlueZ là bộ giao thức Bluetooth chính thức cho Linux. Nó bao gồm:

- `bluez`: Daemon Bluetooth và công cụ CLI (`bluetoothctl`).
- `bluez-utils`: Công cụ dòng lệnh.

### Bluetooth stack

```
Application (pavucontrol, blueman)
    ↕ D-Bus
bluetoothd (bluez daemon)
    ↕ Kernel module (btusb, hci_uart)
Bluetooth hardware (USB, PCI, UART)
```

## Các bước thực hiện

### Bước 1: Cài BlueZ

```bash
pacman -S bluez bluez-utils
```

| Gói | Vai trò |
|---|---|
| `bluez` | Bluetooth daemon (`bluetoothd`) |
| `bluez-utils` | Công cụ `bluetoothctl`, `hciconfig`, `rfcomm` |

### Bước 2: Enable Bluetooth service

```bash
systemctl enable bluetooth
systemctl start bluetooth
```

### Bước 3: Cài blueman (GUI)

```bash
pacman -S blueman
```

blueman cung cấp:
- `blueman-manager`: Quản lý thiết bị Bluetooth.
- `blueman-applet`: Tray icon (đã thêm trong bspwmrc).
- `blueman-sendto`: Gửi file.

### Bước 4: Kiểm tra Bluetooth adapter

```bash
# Kiểm tra controller
bluetoothctl show

# Hoặc
hciconfig -a
```

Output mong đợi:

```
hci0:   Type: Primary  Bus: USB
        BD Address: xx:xx:xx:xx:xx:xx  ACL MTU: 1021:6  SCO MTU: 255:12
        UP RUNNING
        ...
```

### Bước 5: Cấu hình Audio (nếu dùng Bluetooth headset)

Cần thêm gói cho A2DP (âm thanh chất lượng cao):

```bash
pacman -S pulseaudio-bluetooth
```

Nếu dùng PipeWire (đã cấu hình trong bài audio.md), PipeWire tự động hỗ trợ
Bluetooth audio qua module `pipewire-alsa` và `pipewire-pulse`.

### Bước 6: Kết nối thiết bị

```bash
# Mở bluetoothctl
bluetoothctl

# Bật discoverable
power on
discoverable on
pairable on

# Quét thiết bị
scan on

# Kết nối đến thiết bị (dùng MAC address)
pair xx:xx:xx:xx:xx:xx
trust xx:xx:xx:xx:xx:xx
connect xx:xx:xx:xx:xx:xx

# Thoát
exit
```

### Bước 7: Tray icon

Trong `bspwmrc` đã có:

```bash
blueman-applet &
```

## Cấu hình nâng cao

### Tự động kết nối

BlueZ tự động kết nối đến thiết bị đã trust khi phát hiện.

### Cho phép kết nối từ thiết bị ẩn danh

```bash
vim /etc/bluetooth/main.conf
```

Sửa:

```ini
DiscoverableTimeout = 0
PairableTimeout = 0
```

### Xóa thiết bị đã ghép

```bash
bluetoothctl
remove xx:xx:xx:xx:xx:xx
```

## Troubleshooting

### "No default controller available"

```bash
# Kiểm tra adapter
rfkill list
# Nếu blocked → unblock
rfkill unblock bluetooth

# Kiểm tra module
lsmod | grep btusb
# Nếu không → load
modprobe btusb
```

### Bluetooth không thấy thiết bị

```bash
# Kiểm tra firmware Realtek
dmesg | grep -i bluetooth

# Cài firmware
pacman -S linux-firmware
```

### Thiết bị ghép được nhưng không kết nối

```bash
# Xóa và ghép lại
bluetoothctl
remove xx:xx:xx:xx:xx:xx
scan on
pair xx:xx:xx:xx:xx:xx
trust xx:xx:xx:xx:xx:xx
connect xx:xx:xx:xx:xx:xx
```

### Âm thanh Bluetooth không có

- Kiểm tra PipeWire có module bluetooth không.

```bash
pactl list modules | grep blue
```

- Cài `pipewire-alsa pipewire-pulse ` nếu chưa.
- Kiểm tra profile A2DP:

```bash
pactl set-card-profile bluez_card.xx_xx_xx_xx_xx_xx a2dp_sink
```

## Tổng kết

- BlueZ đã được cài và enable.
- blueman cung cấp giao diện đồ họa và tray icon.
- Bluetooth adapter (Realtek) đi kèm chip Wi-Fi.
- Hỗ trợ ghép nối tai nghe, chuột, bàn phím.
- Audio Bluetooth qua PipeWire.
