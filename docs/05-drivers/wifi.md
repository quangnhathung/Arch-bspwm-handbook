# Wi-Fi — Realtek RTL8852BE

## Mục tiêu

Cấu hình Wi-Fi cho card Realtek RTL8852BE (Wi-Fi 6).

## Kiến thức nền

### Realtek RTL8852BE

Card Wi-Fi trên Lenovo LOQ 15IAX9 là Realtek RTL8852BE (RTL8852BE).

- Chuẩn: 802.11ax (Wi-Fi 6)
- Băng tần: 2.4GHz + 5GHz
- Bluetooth đi kèm

### Tại sao cần driver riêng?

Kernel Linux mới nhất đã có driver cho RTL8852BE (`rtw88_8852be`)
nhưng có thể chưa ổn định hoặc bị thiếu firmware.

Driver ổn định nhất là `rtl8852be-dkms` từ AUR.

### Các cách kết nối Wi-Fi

1. **iwd** (iNet Wireless Daemon): Trình quản lý Wi-Fi độc lập, nhẹ.
2. **NetworkManager**: Dùng qua backend iwd hoặc wpa_supplicant.
3. **iwctl**: Công cụ CLI của iwd.
4. **nmtui**: Giao diện text của NetworkManager.

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

Nếu có `rtw88_8852be` → kernel driver đã load. Nếu không → cần driver riêng.

Kiểm tra firmware:

```bash
dmesg | grep -i rtw
```

### Bước 3: Cài driver từ AUR

Cài yay trước (nếu chưa có):

```bash
pacman -S --needed base-devel git
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

Sau đó:

```bash
yay -S rtl8852be-dkms
```

### Bước 4: Load module

```bash
# Load module
sudo modprobe 8852be

# Kiểm tra
lsmod | grep 8852be
```

### Bước 5: Kiểm tra Wi-Fi

```bash
# Kiểm tra interface
ip link show

# Sẽ thấy wlan0 (hoặc wlp2s0)
```

### Bước 6: Kết nối Wi-Fi

```bash
# Dùng NetworkManager
nmcli device status
nmcli device wifi list
nmcli device wifi connect "MyWiFi" password "mypassword"
```

Hoặc dùng iwctl:

```bash
iwctl
station wlan0 scan
station wlan0 get-networks
station wlan0 connect "MyWiFi"
```

### Bước 7: Cấu hình NetworkManager (đã enable)

NetworkManager đã được enable trong quá trình cài (bài 03-installation/network.md).

```bash
systemctl status NetworkManager
```

### Bước 8: Kiểm tra kết nối

```bash
ip addr show wlan0
ping -c 3 archlinux.org
```

## Troubleshooting

### "No Wi-Fi adapter found"

**Symptoms**: NetworkManager báo không thấy thiết bị Wi-Fi.

**Cause**: Driver chưa được load hoặc chưa cài.

**Diagnosis**:

```bash
# Kiểm tra RF kill
rfkill list

# Kiểm tra module
lsmod | grep 8852be

# Kiểm tra kernel messages
dmesg | grep -i rtw
```

**Fix**:
1. Cài `rtl8852be-dkms`.
2. Load module: `sudo modprobe 8852be`.
3. Soft block: `rfkill unblock wifi`.

### "Firmware not found"

**Symptoms**: dmesg báo firmware missing.

**Fix**:

```bash
# Cài firmware
pacman -S linux-firmware

# Hoặc cài firmware riêng từ AUR
yay -S rtl8852be-firmware
```

### Wi-Fi chập chờn, thường xuyên mất kết nối

**Fix**:

```bash
# Tắt power saving
iw dev wlan0 set power_save off

# Hoặc qua NetworkManager
nmcli connection modify "MyWiFi" 802-11-wireless.powersave 2
```

### "Operation not permitted"

```bash
# Kiểm tra block
rfkill list

# Unblock
sudo rfkill unblock wifi
sudo rfkill unblock bluetooth
```

### Card Wi-Fi không được kernel nhận

```bash
# Xem PCI device có được nhận không
lspci -vnn | grep -A 10 "RTL8852BE"
```

Nếu PCI không detect → có thể card bị disabled trong BIOS.

## Dùng USB tethering như fallback

Nếu Wi-Fi chưa hoạt động, dùng LAN hoặc USB tethering:

### Android

1. Cắm USB.
2. Settings → Network → USB Tethering → Bật.

### iPhone

1. Cắm USB.
2. Settings → Personal Hotspot → USB Only.

## Tổng kết

- Card Wi-Fi Realtek RTL8852BE cần driver từ AUR (`rtl8852be-dkms`).
- NetworkManager đã cấu hình để quản lý Wi-Fi.
- Có thể dùng nmcli hoặc iwctl để kết nối.
- Nếu Wi-Fi không hoạt động, dùng LAN hoặc USB tethering.
- Troubleshooting cơ bản: rfkill, modprobe, firmware.
