# Không có Wi-Fi

## Symptoms

- NetworkManager không thấy thiết bị Wi-Fi.
- `nmcli device status` không hiện `wlan0`.
- `iwctl` không thấy adapter.
- Biểu tượng Wi-Fi trên Polybar báo "No Wi-Fi".

## Cause

1. **Driver Realtek chưa được cài hoặc load**.
2. **RF kill switch đang block**.
3. **Firmware thiếu hoặc lỗi**.
4. **NetworkManager backend sai**.
5. **Thiết bị Wi-Fi bị disable trong BIOS**.

## Diagnosis

```bash
# Kiểm tra PCI device
lspci | grep -i network

# Kiểm tra kernel module
lsmod | grep 8852
lsmod | grep rtw

# Kiểm tra RF kill
rfkill list

# Kiểm tra firmware
dmesg | grep -i firmware

# Kiểm tra NetworkManager
systemctl status NetworkManager
journalctl -u NetworkManager -n 20 --no-pager
```

## Fix

### Fix 1: Cài driver Realtek

```bash
# Từ AUR
yay -S rtl8852be-dkms

# Load module
sudo modprobe 8852be

# Kiểm tra
lsmod | grep 8852
```

### Fix 2: Unblock RF kill

```bash
# Xem trạng thái
rfkill list

# Unblock
sudo rfkill unblock wifi
sudo rfkill unblock all

# Kiểm tra lại
rfkill list
```

### Fix 3: Cài firmware

```bash
sudo pacman -S linux-firmware

# Firmware Realtek riêng
yay -S rtl8852be-firmware
```

### Fix 4: Kiểm tra NetworkManager backend

```bash
# Xem backend hiện tại
cat /etc/NetworkManager/conf.d/wifi-backend.conf

# Nếu dùng iwd:
sudo systemctl enable --now iwd

# Nếu dùng wpa_supplicant (mặc định):
sudo pacman -S wpa_supplicant
```

### Fix 5: Reset NetworkManager

```bash
sudo systemctl restart NetworkManager

# Xóa các kết nối cũ (nếu cần)
nmcli connection show
nmcli connection delete "Tên Kết Nối"
```

### Fix 6: Kiểm tra BIOS

Vào BIOS → Configuration → Wireless → Enabled.

## Prevention

1. **Cài `rtl8852be-dkms` ngay sau khi cài hệ thống**.
2. **Thêm module vào mkinitcpio** để load sớm

```bash
vim /etc/mkinitcpio.conf
# Thêm 8852be vào MODULES
MODULES=(8852be)
mkinitcpio -P
```

3. **Enable và start iwd** nếu dùng backend iwd.
4. **Luôn kiểm tra firmware** sau kernel update.
