# Boot vào môi trường Live Arch Linux

## Mục tiêu

Boot từ USB Arch Linux để vào môi trường live (chạy từ RAM), nơi thực hiện toàn bộ quá trình cài đặt.

## Boot process — Lenovo LOQ 15IAX9

1. **Tắt máy** hoàn toàn (Shut down, không Restart)
2. Cắm USB Arch đã ghi (xem bài 04-create-usb)
3. Nhấn nút **Nguồn**
4. Ngay khi màn hình sáng, nhấn liên tục **F12** (hoặc **Fn+F12**)
5. Chọn USB trong danh sách Boot Menu (thường hiện là tên USB hoặc "USB HDD")

**Nếu F12 không hoạt động**: Vào BIOS (F2) → Boot → Boot Override → chọn USB.

### Menu GRUB

Sau khi chọn USB, màn hình GRUB:

```
Arch Linux install medium (x86_64, UEFI)
Arch Linux install medium (x86_64, UEFI) — Copy to RAM
UEFI Shell
```

Chọn dòng đầu tiên, nhấn **Enter**.

### Boot log

Khoảng 10–30 giây sau, kernel log chạy qua màn hình. Kết thúc:

```
root@archiso ~ #
```

Đây là terminal của môi trường live. Toàn bộ hệ thống chạy từ RAM.

## Kiểm tra cơ bản

### 1. Kiểm tra chế độ boot (UEFI vs Legacy)

```bash
ls /sys/firmware/efi/efivars
```

- **Thư mục tồn tại** → boot đúng chế độ **UEFI** ✓
- **Không tìm thấy** → máy đang boot Legacy → vào BIOS chỉnh lại

### 2. Kiểm tra ổ cứng

```bash
lsblk
```

Đầu ra tham khảo — Lenovo LOQ 15IAX9 (NVMe 512GB):

```
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
nvme0n1     259:0    0 476.9G  0 disk
├─nvme0n1p1 259:1    0   100M  0 part  # EFI system partition
├─nvme0n1p2 259:2    0    16M  0 part  # Microsoft reserved
├─nvme0n1p3 259:3    0 476.5G  0 part  # Windows C:
```

Nếu **không thấy nvme0n1**: Vào BIOS kiểm tra SATA Mode đã là AHCI chưa.

### 3. Kiểm tra thông tin phần cứng

```bash
# RAM
free -h

# CPU
cat /proc/cpuinfo | grep "model name" | head -1

# GPU — Intel + NVIDIA
lspci | grep -E "VGA|3D"
```

Kết quả tham khảo — Lenovo LOQ 15IAX9 (i5-12450HX + RTX 4050):

```
00:02.0 VGA compatible controller: Intel Corporation Alder Lake-HX GT1 [UHD Graphics]
01:00.0 3D controller: NVIDIA Corporation AD107M [GeForce RTX 4050 Max-Q]
```

## Kết nối mạng

Bắt buộc phải có Internet để cài Arch. Kiểm tra:

```bash
ping -c 3 archlinux.org
```

### LAN (cáp mạng)

Cắm dây mạng → tự động nhận IP. Kiểm tra:

```bash
ip a
ping -c 3 archlinux.org
```

### Wi-Fi — iwd

Card Wi-Fi Lenovo LOQ 15IAX9 dùng chip **Realtek RTL8852BE**. iwd hỗ trợ card này:

```bash
iwctl
```

Trong iwctl:

```
[iwd]# device list
                Name             Mac address          Enabled          Power Save

         wlan0                   xx:xx:xx:xx:xx:xx       on               off

[iwd]# station wlan0 scan
[iwd]# station wlan0 get-networks
[iwd]# station wlan0 connect "Tên Wi-Fi"
[iwd]# exit
```

Kiểm tra lại:

```bash
ping -c 3 archlinux.org
```

**Nếu Wi-Fi không hoạt động**: Dùng USB tethering (bên dưới). Realtek RTL8852BE có thể cần firmware từ `linux-firmware` — có sẵn trên live USB.

### USB tethering — Android

1. Cắm Android vào máy qua cáp USB
2. Trên Android: **Settings → Network → Hotspot & Tethering → USB Tethering** (bật)
3. Trong Arch live:

```bash
ip link show
# Thấy interface mới: enp0s20f0u1 (hoặc tương tự)
ping -c 3 archlinux.org
```

### USB tethering — iPhone

1. Cắm iPhone vào máy qua cáp USB
2. Trên iPhone: **Settings → Personal Hotspot → "Allow Others to Join"** (chọn "USB Only")
3. Trong Arch live:

```bash
ip link show
# Thấy interface mới
dhcpcd enp0s20f0u2
ping -c 3 archlinux.org
```

## Cập nhật đồng hồ hệ thống

```bash
timedatectl set-ntp true
timedatectl status
```

Xác nhận: `NTP service: active`, thời gian đúng múi giờ.

## Các lệnh kiểm tra khác (tham khảo)

```bash
# Phiên bản kernel
uname -a
# Đầu ra: Linux archiso 7.x-arch1-1 ... (kernel 7.x)

# Phân vùng ổ cứng chi tiết
fdisk -l

# Thông tin block device
lsblk -f

# Kiểm tra module kernel cho NVIDIA (có sẵn trong live)
lsmod | grep nvidia
```

## Troubleshooting

### Màn hình đen sau khi chọn USB boot

| Nguyên nhân | Cách xử lý |
|---|---|
| Secure Boot chưa tắt | Vào BIOS, tắt Secure Boot |
| Fast Boot gây lỗi USB | Vào BIOS, tắt Fast Boot |
| ISO ghi sai chế độ | Ghi lại USB với Rufus DD mode |
| Lỗi driver đồ họa | Thử dùng GRUB option **"Copy to RAM"** |
| Cổng USB 3.0 không tương thích | Thử cổng USB 2.0 (màu đen) |

### USB không xuất hiện trong Boot Menu

- Ghi lại USB với Rufus DD mode (quan trọng)
- Thử cổng USB khác
- Vào BIOS → Boot → Boot Override → chọn USB (nếu F12 không hoạt động)
- USB hỏng → thử USB khác

### Wi-Fi không hoạt động

- Card Realtek RTL8852BE: iwd có hỗ trợ — thử `rfkill unblock all`
- Kiểm tra `rfkill list` — nếu bị block: `rfkill unblock wifi`
- Dùng USB tethering làm phương án dự phòng
- Nếu vẫn không được, cắm dây LAN hoặc dùng điện thoại tethering

## Pre-installation checklist

Trước khi sang bước cài đặt, xác nhận:

- [x] Đã boot thành công vào Arch live environment
- [x] `ls /sys/firmware/efi/efivars` → có thư mục (UEFI mode)
- [x] `lsblk` → thấy ổ NVMe
- [x] `ping -c 3 archlinux.org` → có Internet
- [x] `timedatectl status` → NTP active, giờ đúng

Tất cả các điều kiện đã sẵn sàng. Chuyển sang phần cài đặt Arch Linux.
