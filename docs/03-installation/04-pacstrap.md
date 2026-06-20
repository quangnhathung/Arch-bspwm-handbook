# Cài đặt Base System

## Mục tiêu

Cài đặt hệ thống nền tảng Arch Linux vào `/mnt` bằng pacstrap.

## Điều kiện tiên quyết

- Các partition đã mount đúng vào `/mnt`.
- Có kết nối Internet trong live environment.
- Đồng hồ đã đồng bộ.

## Kiến thức nền

### pacstrap là gì?

`pacstrap` là script đi kèm arch-install-scripts, dùng để cài gói vào một thư mục
thay vì vào hệ thống đang chạy. Nó tương đương với `pacman -r /mnt -S gói`.

### Các gói base

Chúng ta cài các gói sau:

| Gói | Vai trò |
|---|---|
| `base` | Hệ thống cơ bản (glibc, coreutils, bash, v.v.) |
| `linux` | Linux kernel |
| `linux-firmware` | Firmware cho phần cứng (Wi-Fi, GPU, v.v.) |
| `intel-ucode` | Microcode cho CPU Intel (sửa lỗi CPU, bảo mật) |
| `sof-firmware` | Firmware cho Intel Smart Sound Technology |
| `vim` | Trình soạn thảo văn bản |
| `sudo` | Cấp quyền root cho user thường |
| `networkmanager` | Quản lý kết nối mạng (Wi-Fi, LAN) |
| `git` | Công cụ phiên bản, cần cho AUR helper sau này |
| `base-devel` | Công cụ biên dịch (make, gcc, v.v.) — cần cho AUR |

### Tại sao cần từng gói?

#### base

Nhóm gói nền tảng: `glibc` (thư viện C), `bash` (shell), `coreutils` (lệnh cơ bản
như cp, mv, ls), `pacman` (quản lý gói), `systemd` (init system), `linux-api-headers`.

Không thể thiếu. Nếu thiếu base, hệ thống không chạy được.

#### linux

Kernel. Nếu cài linux-lts thì kernel sẽ ổn định hơn nhưng cũ hơn.
Với máy mới (Alder Lake, RTX 4050), cần kernel mới nhất → dùng `linux`.

#### linux-firmware

Chứa firmware blob cho phần cứng: Wi-Fi, Bluetooth, GPU, NVMe, v.v.
Thiếu gói này, Wi-Fi và nhiều thiết bị khác không hoạt động.

#### intel-ucode

Microcode là các bản vá lỗi cho CPU được Intel phát hành qua update.
GRUB sẽ tự động load microcode nếu có gói này. Cực kỳ quan trọng cho bảo mật
và ổn định.

#### sof-firmware

Sound Open Firmware — firmware cho Intel DSP (Digital Signal Processor).
Cần cho âm thanh trên máy Intel Alder Lake. Nếu thiếu, loa trong không hoạt động.

#### vim

Cần một text editor trong hệ thống mới. `vi` có sẵn nhưng `vim` tốt hơn nhiều.
Nếu bạn thích `nano`, có thể thay thế — nhưng vim là chuẩn thực tế.

#### sudo

Cho phép user thường chạy lệnh với quyền root. Cần để không phải login
trực tiếp bằng root.

#### networkmanager

Công cụ quản lý mạng của systemd. Hỗ trợ Wi-Fi, LAN, USB tethering,
VPN. Cung cấp `nmtui` (giao diện terminal) và `nmcli` (dòng lệnh).

#### git

Cần để clone AUR packages. Cũng cần cho yay (AUR helper).

#### base-devel

Nhóm gói phát triển: `gcc`, `make`, `autoconf`, `automake`, `pkg-config`.
Cần để biên dịch gói từ AUR.

## Các bước thực hiện

### Bước 1: Cập nhật keyring (quan trọng)

```bash
pacman -Sy archlinux-keyring
```

Giải thích: Keyring chứa các GPG keys dùng để xác thực gói. Nếu ISO cũ,
keyring có thể lỗi thời → cập nhật trước.

### Bước 2: Cài base system

```bash
pacstrap -K /mnt base linux linux-firmware intel-ucode sof-firmware vim sudo networkmanager git base-devel
```

Giải thích option:
- `-K`: Initialize keyring trong hệ thống mới.
- `/mnt`: Thư mục đích.

### Bước 3: Chờ quá trình cài đặt

Quá trình này tải khoảng 500MB-1GB gói. Thời gian tùy tốc độ mạng.

Nếu mạng chậm hoặc cài lại, có thể dùng:

```bash
pacstrap -K /mnt base base-devel linux linux-firmware intel-ucode sof-firmware vim sudo networkmanager git
```

### Bước 4: Kiểm tra cài đặt

```bash
ls /mnt/bin/bash
ls /mnt/usr/bin/pacman
```

Nếu có file → cài đặt thành công.

## Nếu có lỗi

### "Failed to install packages to /mnt"

- Kiểm tra mount: `lsblk` → /mnt phải có filesystem.
- Thiếu Internet: `ping -c 3 archlinux.org`.
- Keyring cũ: `pacman -Sy archlinux-keyring`.

### "Signature from X is unknown trust"

```bash
pacman -Sy archlinux-keyring
# Sau đó chạy lại pacstrap
```

### Hết dung lượng

- Cache trong live environment đầy → `pacman -Scc` để xóa cache.
- Kiểm tra `df -h /mnt` — nếu thiếu dung lượng, subvolume chưa mount đúng.

## Tổng kết

- Base system đã được cài vào `/mnt`.
- Các gói cần thiết cho laptop Lenovo LOQ đã được thêm.
- Sẵn sàng cấu hình fstab, locale, network, và chroot.
