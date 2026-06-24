# Không có Wi-Fi

*Áp dụng cho Lenovo LOQ 15IAX9 — Realtek RTL8852BE — Kernel 7.x — 25/06/2026*

## Triệu chứng (Symptoms)

- NetworkManager (NM) không thấy thiết bị Wi-Fi — `nmcli device status` chỉ hiện `lo` và `p2p`.
- `iwctl` không liệt kê adapter.
- Polybar/tray icon báo "No Wi-Fi" hoặc "device not ready".
- `ip link` không hiện `wlan0` hoặc `wlp*`.

## Nguyên nhân (Causes)

1. **Driver Realtek RTL8852BE chưa được cài** — kernel 7.x không có driver mặc định cho chip này.
2. **Module `8852be` không được load** — cần `modprobe` hoặc thêm vào `mkinitcpio.conf`.
3. **RF kill switch đang block** — `rfkill list` hiện `Soft blocked: yes`.
4. **Firmware rtl8852be thiếu** — driver cần firmware để hoạt động.
5. **NetworkManager dùng sai backend** — cần `wpa_supplicant` hoặc `iwd`.

## Chẩn đoán (Diagnosis)

```bash
# 1. Kiểm tra PCI device
lspci | grep -i network

# 2. Kiểm tra module Realtek
lsmod | grep -iE "8852|rtw"

# 3. Kiểm tra RF kill
rfkill list

# 4. Kiểm tra firmware
dmesg | grep -iE "firmware|8852"

# 5. Kiểm tra NM log
journalctl -u NetworkManager -n 30 --no-pager

# 6. Kiểm tra kernel version
uname -r
```

## Khắc phục (Fix)

### Fix 1: Cài driver Realtek RTL8852BE

```bash
# Từ AUR (cần yay hoặc paru)
yay -S rtl8852be-dkms

# Load module ngay lập tức
sudo modprobe 8852be

# Kiểm tra
lsmod | grep 8852
dmesg | tail
```

### Fix 2: Unblock RF kill

```bash
rfkill list

# Unblock tất cả
sudo rfkill unblock wifi
sudo rfkill unblock all

# Kiểm tra lại
rfkill list
ip link
```

Nếu Wi-Fi vẫn "blocked" → có thể do BIOS:

```
Vào BIOS (F2 khi boot) → Configuration → Wireless → Enabled
```

### Fix 3: Cài firmware Realtek

```bash
yay -S rtl8852be-firmware

# Cài linux-firmware tổng quát
sudo pacman -S linux-firmware

# Reboot
sudo reboot
```

### Fix 4: Kiểm tra NetworkManager backend

```bash
# Mặc định Arch dùng wpa_supplicant
sudo pacman -S wpa_supplicant --needed

# Nếu muốn dùng iwd
sudo systemctl enable --now iwd
# Sửa /etc/NetworkManager/conf.d/wifi-backend.conf:
# [device]
# wifi.backend=iwd
sudo systemctl restart NetworkManager
```

### Fix 5: Reset NetworkManager

```bash
sudo systemctl restart NetworkManager

# Xóa cache kết nối nếu cần
nmcli connection show
nmcli connection delete "ten-ket-noi"

# Restart module
sudo modprobe -r 8852be && sudo modprobe 8852be
```

## Phòng ngừa (Prevention)

1. **Cài `rtl8852be-dkms` ngay sau khi cài Arch, trước khi reboot lần đầu.**
2. **Thêm module vào `mkinitcpio.conf` để load sớm:**

```bash
echo 'MODULES=(8852be)' | sudo tee /etc/mkinitcpio.conf.d/wifi.conf
sudo mkinitcpio -P
```

3. **Kiểm tra firmware sau mỗi lần cập nhật kernel:**
```bash
dmesg | grep -i firmware | grep -i 8852
```

4. **Luôn giữ `linux-firmware` và `rtl8852be-dkms` trong danh sách gói cần update.**
