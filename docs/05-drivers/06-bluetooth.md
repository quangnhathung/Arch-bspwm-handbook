# Bluetooth

## Mục tiêu

Cài đặt và cấu hình Bluetooth trên Lenovo LOQ 15IAX9.

## Kiến thức nền

### Bluetooth trên Lenovo LOQ

Bluetooth tích hợp trong chip Wi-Fi Realtek RTL8852BE (cùng một module). Hỗ trợ Bluetooth 5.2.

### BlueZ stack

```
Application (blueman, pavucontrol)
    ↕ D-Bus
bluetoothd (bluez daemon)
    ↕ Kernel module (btusb)
Bluetooth hardware (USB bus — Realtek)
```

BlueZ là bộ giao thức Bluetooth chính thức cho Linux.

## Các bước thực hiện

### Bước 1: Cài BlueZ

```bash
pacman -S bluez bluez-utils
```

| Gói | Vai trò |
|---|---|
| `bluez` | Bluetooth daemon (`bluetoothd`) |
| `bluez-utils` | Công cụ CLI (`bluetoothctl`, `hciconfig`, `rfcomm`) |

### Bước 2: Enable Bluetooth service

```bash
sudo systemctl enable bluetooth
sudo systemctl start bluetooth
```

### Bước 3: Cài blueman (GUI)

```bash
pacman -S blueman
```

blueman cung cấp:

- `blueman-manager`: Quản lý thiết bị Bluetooth
- `blueman-applet`: Tray icon (đã thêm trong bspwmrc)
- `blueman-sendto`: Gửi file

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
```

### Bước 5: Bluetooth audio với PipeWire

PipeWire (đã cài trong bài audio) hỗ trợ Bluetooth A2DP (âm thanh chất lượng cao) và HSP/HFP (micro + tai nghe) **tự động** — không cần `pulseaudio-bluetooth`.

Kiểm tra module Bluetooth của PipeWire:

```bash
pactl list modules | grep blue
```

Nếu có `module-bluez5-discover` và `module-bluez5-manager` → PipeWire đã sẵn sàng cho Bluetooth audio.

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

# Kết nối (dùng MAC address từ scan)
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

BlueZ tự động kết nối đến thiết bị đã trust khi ở trong tầm.

### Cho phép kết nối từ thiết bị lạ

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
# Kiểm tra RF kill
rfkill list
# Nếu blocked → unblock
rfkill unblock bluetooth

# Kiểm tra module btusb
lsmod | grep btusb
# Nếu không → load
modprobe btusb

# Kiểm tra dmesg
dmesg | grep -i bluetooth
```

### Bluetooth không thấy thiết bị

```bash
# Cài firmware Realtek
pacman -S linux-firmware

# Kiểm tra firmware đã load
dmesg | grep -i "rtl.*bt"
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

```bash
# Kiểm tra PipeWire Bluetooth module
pactl list modules | grep blue

# Kiểm tra profile
pactl list cards | grep -A 10 bluez

# Set profile A2DP (âm thanh chất lượng cao)
pactl set-card-profile bluez_card.xx_xx_xx_xx_xx_xx a2dp_sink

# Restart PipeWire nếu cần
systemctl --user restart pipewire wireplumber
```

### WirePlumber không quản lý Bluetooth

WirePlumber đã hỗ trợ Bluetooth mặc định. Kiểm tra:

```bash
systemctl --user status wireplumber

# Nếu thiếu policy, cài wireplumber (đã làm ở bài audio)
pacman -S wireplumber
```

## Tổng kết

- BlueZ + blueman: quản lý thiết bị Bluetooth
- Bluetooth audio qua PipeWire (A2DP tự động)
- Tray icon: `blueman-applet &`
- Troubleshooting: rfkill, btusb, firmware, PipeWire profile
