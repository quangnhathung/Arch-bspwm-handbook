# Boot Live Environment

## Mục tiêu

Boot từ USB Arch Linux để vào môi trường live, nơi chúng ta sẽ thực hiện toàn bộ
quá trình cài đặt.

## Cách boot

### Trên Lenovo LOQ 15IAX9

1. **Tắt máy** hoàn toàn (Shut down, không Restart).
2. Cắm USB Arch đã ghi.
3. Nhấn **Nguồn**.
4. Ngay khi màn hình sáng, nhấn liên tục **F12** (hoặc **Fn + F12**).
5. Menu Boot xuất hiện → chọn USB của bạn (thường có tên "USB HDD" hoặc tên USB).
6. Nhấn Enter.

**Nếu F12 không hoạt động**: Vào BIOS → Boot → Boot Override → chọn USB.

### Màn hình GRUB

Sau khi chọn USB, bạn sẽ thấy menu GRUB của Arch:

```
Arch Linux install medium (x86_64, UEFI)
Arch Linux install medium (x86_64, UEFI) — Copy to RAM
UEFI Shell
```

Chọn dòng đầu tiên và nhấn Enter.

### Boot process

Máy sẽ hiện một loạt log kernel chạy qua. Sau khoảng 10-30 giây, bạn sẽ thấy
terminal với dấu nhắc:

```
root@archiso ~ #
```

Bạn đang ở trong môi trường live. Đây là một Arch Linux đầy đủ chạy từ RAM.

## Môi trường live có gì?

- **Root**: Bạn đang là `root`, không cần password.
- **RAM**: Hệ thống chạy hoàn toàn từ RAM (khoảng 1-2GB).
- **Công cụ**: Có sẵn `pacman`, `iwd`, `systemctl`, `fdisk`, `vim`, `ping`, `curl`.
- **Phiên bản**: Linux kernel mới nhất.
- **Không có giao diện đồ họa**: Chỉ terminal.

## Kiểm tra cơ bản

### 1. Kiểm tra chế độ boot

```bash
ls /sys/firmware/efi/efivars
```

Nếu thư mục tồn tại → **đã boot đúng UEFI mode**.
Nếu không → máy đang boot Legacy → vào BIOS chỉnh lại.

### 2. Kiểm tra ổ cứng

```bash
lsblk
```

Đầu ra sẽ giống:

```
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
nvme0n1     259:0    0 476.9G  0 disk
├─nvme0n1p1 259:1    0   100M  0 part
├─nvme0n1p2 259:2    0    16M  0 part
├─nvme0n1p3 259:3    0 476.5G  0 part
```

`nvme0n1` là ổ NVMe 512GB. Các partition (p1, p2, p3) là của Windows.

### 3. Kiểm tra kết nối mạng

#### Nếu dùng LAN (cable)

```bash
ping -c 3 archlinux.org
```

Nếu có kết quả → có mạng rồi.

#### Nếu dùng Wi-Fi — dùng iwctl

```bash
iwctl
```

Bên trong iwctl (dấu nhắc `[iwd]#`):

```bash
# Liệt kê thiết bị Wi-Fi
device list

# Quét mạng
station wlan0 scan

# Liệt kê mạng tìm thấy
station wlan0 get-networks

# Kết nối
station wlan0 connect "Tên Wi-Fi"
# Nhập password khi được hỏi

# Thoát
exit
```

Kiểm tra lại:

```bash
ping -c 3 archlinux.org
```

#### Nếu dùng USB tethering Android

1. Cắm Android vào máy qua USB.
2. Trên Android: Settings → Network → USB Tethering → Bật.
3. Trong Arch live:

```bash
ip link show
# Sẽ thấy interface mới như enp0s20f0u1
ping -c 3 archlinux.org
```

#### Nếu dùng USB tethering iPhone

1. Cắm iPhone vào máy qua USB.
2. Trên iPhone: Settings → Personal Hotspot → Bật. Chọn "USB Only".
3. Trong Arch live:

```bash
ip link show
# Sẽ thấy interface enp0s20f0u2 (hoặc tương tự)
dhcpcd enp0s20f0u2
ping -c 3 archlinux.org
```

### 4. Cập nhật đồng hồ hệ thống

```bash
timedatectl set-ntp true
timedatectl status
```

## Các thao tác hữu ích trong live

### Xem dung lượng RAM

```bash
free -h
```

### Xem CPU

```bash
cat /proc/cpuinfo | grep "model name" | head -1
```

### Xem GPU

```bash
lspci | grep -E "VGA|3D"
```

## Nếu không boot được

### Màn hình đen sau khi chọn USB

- Secure Boot đã tắt chưa? Vào BIOS kiểm tra lại.
- Fast Boot đã tắt chưa?
- Thử option "Copy to RAM" trong GRUB menu.
- Thử cổng USB 2.0 (màu đen) thay vì 3.0 (màu xanh).

### USB không xuất hiện trong boot menu

- USB chưa được ghi đúng cách → ghi lại với Rufus DD mode.
- USB hỏng → thử USB khác.
- Cổng USB hỏng → thử cổng khác.

### Lỗi "No WIFI adapter found"

- iwd không hỗ trợ card này → dùng USB tethering để có mạng.
- Card Wi-Fi chưa được bật → kiểm tra bằng `rfkill list`.

## Tổng kết

- Boot USB thành công với UEFI mode.
- Có terminal với quyền root.
- Có kết nối Internet.
- Đồng hồ đã đồng bộ.

Sẵn sàng bắt đầu cài đặt.
