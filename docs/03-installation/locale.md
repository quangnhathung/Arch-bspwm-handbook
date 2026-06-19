# Locale, Timezone, Hostname

## Mục tiêu

Cấu hình ngôn ngữ, múi giờ, và tên máy sau khi chroot vào hệ thống mới.

## Điều kiện tiên quyết

- Đã cài base system bằng pacstrap.
- Đã genfstab và kiểm tra.
- Đã chroot vào `/mnt`.

## Chroot

```bash
arch-chroot /mnt
```

`arch-chroot` là script tự động mount các filesystem cần thiết (proc, sys, dev)
và chroot vào hệ thống mới.

Sau lệnh này, prompt sẽ thay đổi thành:

```
[root@archiso /]#
```

## Các bước cấu hình

### Bước 1: Cấu hình Timezone

```bash
# Liệt kê các múi giờ có sẵn
timedatectl list-timezones | grep -i asia

# Đặt timezone cho Việt Nam
ln -sf /usr/share/zoneinfo/Asia/Ho_Chi_Minh /etc/localtime

# Đồng bộ hardware clock (RTC)
hwclock --systohc
```

Giải thích:
- `ln -sf`: Tạo symlink. Timezone được xác định bằng symlink từ `/etc/localtime`
  đến file timezone tương ứng.
- `/usr/share/zoneinfo/Asia/Ho_Chi_Minh`: Múi giờ Việt Nam (UTC+7).
- `hwclock --systohc`: Ghi thời gian hệ thống vào hardware clock (RTC — BIOS clock).
  Mặc định Linux dùng UTC cho RTC. Windows dùng local time. Nếu muốn dual boot
  với Windows, cần thêm bước. Nhưng chúng ta đã xóa Windows.

#### Nếu muốn dùng RTC ở localtime:

Không cần cho máy chỉ có Arch. Nhưng nếu sau này cần dual boot thì:

```bash
timedatectl set-local-rtc 1
```

### Bước 2: Cấu hình Locale

Locale định nghĩa ngôn ngữ, định dạng số, ngày tháng, tiền tệ.

```bash
# Sửa file locale.gen — bỏ comment dòng en_US.UTF-8 và vi_VN.UTF-8
vim /etc/locale.gen
```

Tìm và uncomment (xóa dấu `#` ở đầu) các dòng:

```
en_US.UTF-8 UTF-8
vi_VN.UTF-8 UTF-8
```

Sau đó:

```bash
# Sinh locale
locale-gen

# Tạo /etc/locale.conf
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

Giải thích:
- `locale-gen`: Đọc `/etc/locale.gen` và sinh các locale đã được uncomment.
- `/etc/locale.conf`: File cấu toàn hệ thống. `LANG=en_US.UTF-8` là ngôn ngữ mặc định.
  (Không dùng vi_VN vì terminal sẽ lỗi font và nhiều ứng dụng không hỗ trợ.)

### Bước 3: Đặt Hostname

```bash
# Tạo file hostname
echo "loq-arch" > /etc/hostname

# Sửa file hosts
vim /etc/hosts
```

Thêm vào `/etc/hosts`:

```
127.0.0.1   localhost
::1         localhost
127.0.1.1   loq-arch.localdomain   loq-arch
```

Giải thích:
- `127.0.0.1 localhost`: Loopback IPv4 mặc định.
- `::1 localhost`: Loopback IPv6 mặc định.
- `127.0.1.1 loq-arch.localdomain loq-arch`: Hostname của máy.
  `127.0.1.1` (không phải 127.0.0.1) là convention để hostname resolution
  hoạt động với network services.

### Bước 4: Kiểm tra

```bash
# Kiểm tra timezone
timedatectl status

# Kiểm tra locale
locale

# Kiểm tra hostname
hostname
hostname -f
```

## Tổng kết

Các cấu hình cơ bản đã được thiết lập:

| Cấu hình | Giá trị |
|---|---|
| Timezone | Asia/Ho_Chi_Minh |
| Locale | en_US.UTF-8 |
| Hostname | loq-arch |

Sẵn sàng cấu hình network và tạo user.
