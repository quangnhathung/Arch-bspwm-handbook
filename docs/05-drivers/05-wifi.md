# Wi-Fi — Realtek RTL8852BE

## Mục tiêu

Cấu hình Wi-Fi cho card Realtek RTL8852BE (Wi-Fi 6) trên Lenovo LOQ 15IAX9.

## Kiến thức nền

### Realtek RTL8852BE

Card Wi-Fi tích hợp trên Lenovo LOQ 15IAX9:

- Chip: Realtek RTL8852BE
- Chuẩn: 802.11ax (Wi-Fi 6)
- Băng tần: 2.4 GHz + 5 GHz
- Bluetooth 5.2 đi kèm (cùng chip)

### Driver

| Driver | Loại | Trạng thái |
|---|---|---|
| `rtw88_8852be` | In-kernel (linux 6.2+) | Có thể hoạt động nhưng chưa ổn định |
| `rtl8852be-dkms` | AUR (`rtl8852be-dkms`) | **Khuyến nghị** — ổn định nhất |

Kernel 7.x đã có driver `rtw88_8852be` nhưng card Realtek thường hoạt động tốt hơn với driver DKMS từ AUR.

## Các bước thực hiện

### Bước 1: Xác nhận card Wi-Fi

```bash
lspci | grep -i network
```

Output:

```
02:00.0 Network controller: Realtek Semiconductor Co., Ltd. RTL8852BE PCIe 802.11ax Wireless Network Controller
```

### Bước 2: Kiểm tra driver hiện tại

```bash
lsmod | grep rtw
```

Nếu có `rtw88_8852be` → kernel driver đã load. Nếu không → cần cài driver riêng.

Kiểm tra firmware:

```bash
dmesg | grep -i rtw
```

### Bước 3: Cài driver từ AUR

Package AUR chính xác: **`rtl8852be-dkms`**

```bash
yay -S rtl8852be-dkms
```

Nếu chưa có yay:

```bash
pacman -S --needed base-devel git
git clone https://aur.archlinux.org/yay-bin.git
cd yay-bin
makepkg -si

# Sau đó cài driver
yay -S rtl8852be-dkms
```

### Bước 4: Load module

```bash
# Load thủ công
sudo modprobe 8852be

# Kiểm tra
lsmod | grep 8852be
```

### Bước 5: Kiểm tra interface

```bash
ip link show
```

Output mong đợi: `wlan0` hoặc `wlp2s0`.

### Bước 6: Kết nối Wi-Fi

```bash
# Dùng NetworkManager
nmcli device status
nmcli device wifi list
nmcli device wifi connect "Tên_WiFi" password "mật_khẩu"
```

Hoặc dùng iwd:

```bash
iwctl
station wlan0 scan
station wlan0 get-networks
station wlan0 connect "Tên_WiFi"
```

### Bước 7: Kiểm tra kết nối

```bash
ip addr show wlan0
ping -c 3 archlinux.org
```

### Bước 8 (tùy chọn): Load module sớm khi boot

Nếu muốn Wi-Fi hoạt động ngay từ initramfs, thêm vào `/etc/mkinitcpio.conf`:

```
MODULES=(... 8852be ...)
```

Sau đó:

```bash
mkinitcpio -P
```

## Troubleshooting

### "No Wi-Fi adapter found"

```bash
# Kiểm tra RF kill
rfkill list

# Nếu soft blocked:
rfkill unblock wifi

# Kiểm tra module
lsmod | grep 8852be

# Kiểm tra kernel messages
dmesg | grep -i rtw
```

### "Firmware not found"

```bash
# Cài linux-firmware (chứa firmware cho nhiều thiết bị)
pacman -S linux-firmware

# Hoặc cài firmware riêng
yay -S rtl8852be-firmware
```

### Wi-Fi chập chờn, mất kết nối

```bash
# Tắt power saving
iw dev wlan0 set power_save off

# Hoặc qua NetworkManager
nmcli connection modify "Tên_WiFi" 802-11-wireless.powersave 2

# Kiểm tra sau khi tắt:
iw dev wlan0 get power_save
```

### "Operation not permitted"

```bash
rfkill list
sudo rfkill unblock wifi
sudo rfkill unblock bluetooth
```

### Card không được kernel nhận

```bash
lspci -vnn | grep -A 20 "RTL8852BE"
```

Nếu PCI không detect → kiểm tra BIOS có disable Wi-Fi không.

## Fallback: USB tethering

Nếu Wi-Fi chưa hoạt động:

### Android

1. Cắm USB
2. Settings → Network → USB Tethering → Bật

### iPhone

1. Cắm USB
2. Settings → Personal Hotspot → USB Only

## Tổng kết

- Card Wi-Fi: Realtek RTL8852BE — cần driver AUR
- Package: `rtl8852be-dkms` từ AUR
- Load module: `modprobe 8852be`
- Kết nối qua NetworkManager (`nmcli`)
- Troubleshooting: rfkill, modprobe, firmware, power_save
