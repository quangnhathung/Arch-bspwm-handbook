# Locale, Timezone, Hostname

## Mục tiêu

Cấu hình ngôn ngữ (locale), múi giờ (timezone), và tên máy (hostname) sau khi
chroot vào hệ thống mới.

## Điều kiện tiên quyết

- Đã cài base system bằng pacstrap.
- Đã genfstab và kiểm tra fstab.
- Đã chroot vào `/mnt`.

## Chroot

Chroot là kỹ thuật "chuyển root" — tạm thời đổi thư mục gốc của hệ thống đang
chạy. Nó cho phép chúng ta thao tác với hệ thống mới như thể đã boot vào nó.

```bash
arch-chroot /mnt
```

`arch-chroot` tự động:
1. Mount `/proc`, `/sys`, `/dev`, `/run` vào `/mnt`.
2. Chroot vào `/mnt`.
3. Cập nhật prompt thành `[root@archiso /]#`.

Sau lệnh này, bạn đang ở trong hệ thống Arch mới.

---

## Cách A: Cài thủ công

### Bước 1: Cấu hình Timezone

```bash
# Liệt kê múi giờ châu Á
timedatectl list-timezones | grep -i "ho_chi\|hanoi\|vietnam"
```

Đặt timezone cho Việt Nam:
```bash
ln -sf /usr/share/zoneinfo/Asia/Ho_Chi_Minh /etc/localtime
```

**Giải thích:**
- `/usr/share/zoneinfo/` chứa tất cả timezone trên thế giới.
- `ln -sf` tạo symlink từ `/etc/localtime` đến timezone mong muốn.
- `Asia/Ho_Chi_Minh` là timezone chuẩn cho Việt Nam (UTC+7, không có DST).

Đồng bộ hardware clock (RTC — đồng hồ trong BIOS):
```bash
hwclock --systohc
```

**Giải thích:**
- `--systohc`: Ghi thời gian hệ thống (system clock) vào hardware clock.
- Linux mặc định dùng UTC cho RTC. Windows dùng local time.
- Máy này chỉ có Arch → dùng UTC là chuẩn.
- Nếu sau này dual boot với Windows, cần: `timedatectl set-local-rtc 1`.

### Bước 2: Cấu hình Locale

Locale định nghĩa ngôn ngữ, định dạng số, ngày tháng, tiền tệ, mã hóa ký tự.

#### 2a. Sửa `/etc/locale.gen`

```bash
vim /etc/locale.gen
```

Tìm và bỏ comment (xóa dấu `#` ở đầu) các dòng:
```
en_US.UTF-8 UTF-8
vi_VN.UTF-8 UTF-8
```

**Giải thích:**
- `en_US.UTF-8`: Locale tiếng Anh Mỹ — dùng làm ngôn ngữ hệ thống.
- `vi_VN.UTF-8`: Locale tiếng Việt — có thể dùng cho một số ứng dụng.
- Chỉ uncomment locale bạn cần. Càng ít locale càng nhanh.

#### 2b. Sinh locale

```bash
locale-gen
```

Output:
```
Generating locales...
  en_US.UTF-8 ... done
  vi_VN.UTF-8 ... done
Generation complete.
```

#### 2c. Tạo `/etc/locale.conf`

```bash
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

**Tại sao không dùng `vi_VN.UTF-8` làm mặc định?**
- Terminal và nhiều ứng dụng CLI không hỗ trợ tiếng Việt đúng cách.
- Font thiếu ký tự có dấu → hiển thị lỗi.
- `en_US.UTF-8` là locale mặc định an toàn nhất.

Xuất biến locale cho session hiện tại:
```bash
export LANG=en_US.UTF-8
```

### Bước 3: Cấu hình Hostname

Hostname là tên của máy tính trên mạng.

```bash
# Tạo file hostname
echo "loq-arch" > /etc/hostname
```

Bạn có thể đặt tên khác: `arch-lenovo`, `loq-box`, v.v.

#### 3a. Sửa `/etc/hosts`

```bash
vim /etc/hosts
```

Thêm nội dung:
```
127.0.0.1   localhost
::1         localhost
127.0.1.1   loq-arch.localdomain   loq-arch
```

**Giải thích:**
- `127.0.0.1 localhost`: Loopback IPv4.
- `::1 localhost`: Loopback IPv6.
- `127.0.1.1 loq-arch.localdomain loq-arch`: Hostname resolution.
  - `127.0.1.1` (không phải `127.0.0.1`) là convention để phân biệt hostname
    thật với localhost.
  - `localdomain` là domain mặc định.
  - Cần cho một số service như `systemd-logind`, `sudo` resolve hostname.

### Bước 4: Kiểm tra tất cả cấu hình

```bash
# Kiểm tra timezone
timedatectl status

# Kiểm tra locale
locale

# Kiểm tra hostname
hostname
hostname -f

# Kiểm tra file cấu hình
cat /etc/hostname
cat /etc/locale.conf
```

Output mong đợi:
```
# timedatectl status
               Local time: Thu 2026-06-25 14:30:00 +07
           Universal time: Thu 2026-06-25 07:30:00 UTC
                 RTC time: Thu 2026-06-25 07:30:00
                Time zone: Asia/Ho_Chi_Minh (+07, +0700)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no

# locale
LANG=en_US.UTF-8
LC_CTYPE="en_US.UTF-8"
LC_NUMERIC="en_US.UTF-8"
...

# hostname
loq-arch
```

---

## Cách B: Dùng Archinstall

Archinstall có menu riêng cho locale, timezone, hostname.

### Bước 1: Trong quá trình archinstall

Khi đến các mục:
1. **`Locales`**: Chọn ngôn ngữ.
   - `Locale language`: `en_US.UTF-8`.
   - `Locale encoding`: `UTF-8`.
   - Có thể thêm `vi_VN.UTF-8` ở mục bổ sung.

2. **`Timezone`**: Chọn `Asia` → `Ho_Chi_Minh`.
   - Gõ tên để search: `Ho_Chi_Minh`.

3. **`Hostname`**: Nhập `loq-arch` (hoặc tên bạn muốn).

### Bước 2: Kiểm tra sau archinstall

Sau khi archinstall hoàn tất (trước khi reboot):

```bash
arch-chroot /mnt

# Kiểm tra
timedatectl status
locale
cat /etc/hostname
cat /etc/hosts
```

### Bước 3: Sửa nếu cần

Nếu archinstall chưa cấu hình đúng:
```bash
# Cấu hình thủ công như cách A
ln -sf /usr/share/zoneinfo/Asia/Ho_Chi_Minh /etc/localtime
hwclock --systohc
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

---

## Tổng kết

| Cấu hình | Giá trị | File |
|---|---|---|
| Timezone | `Asia/Ho_Chi_Minh` | `/etc/localtime` (symlink) |
| Locale mặc định | `en_US.UTF-8` | `/etc/locale.conf` |
| Locale đã sinh | `en_US.UTF-8`, `vi_VN.UTF-8` | `/etc/locale.gen` |
| Hostname | `loq-arch` | `/etc/hostname`, `/etc/hosts` |
| Hardware clock | UTC | `hwclock --systohc` |

Sẵn sàng cấu hình network và tạo user.
