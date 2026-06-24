# Cài đặt Base System (Pacstrap)

## Mục tiêu

Cài đặt hệ thống nền tảng Arch Linux vào `/mnt` bằng pacstrap.

## Điều kiện tiên quyết

- Các partition đã mount đúng vào `/mnt` (xem bài 03-mounting).
- Có kết nối Internet trong live environment.
- Đồng hồ đã đồng bộ (`timedatectl set-ntp true`).

## Kiến thức nền

### pacstrap là gì?

`pacstrap` là script đi kèm gói `arch-install-scripts`, dùng để cài gói vào một
thư mục thay vì vào hệ thống đang chạy. Nó tương đương với:
```bash
pacman -r /mnt -S gói1 gói2 ...
```

### -K flag (keyring initialization)

`pacstrap -K` tự động:
1. Cài đặt `archlinux-keyring` vào hệ thống mới.
2. Khởi tạo pacman keyring.
3. Đảm bảo chữ ký GPG của gói được xác thực ngay trong lần đầu cài.

## Danh sách gói

Chúng ta cài các gói sau:

| Gói | Dung lượng | Vai trò |
|---|---|---|
| `base` | ~150MB | Hệ thống cơ bản: glibc, bash, coreutils, pacman, systemd, linux-api-headers |
| `linux` | ~120MB | Linux kernel 7.x (bản mới nhất, rolling release) |
| `linux-firmware` | ~500MB | Firmware cho phần cứng: Wi-Fi, Bluetooth, GPU, NVMe, âm thanh |
| `intel-ucode` | ~2MB | Microcode cho CPU Intel i5-12450HX (sửa lỗi CPU, bảo mật) |
| `sof-firmware` | ~2MB | Sound Open Firmware cho Intel SST (cần cho âm thanh Lenovo LOQ) |
| `vim` | ~5MB | Trình soạn thảo văn bản |
| `sudo` | ~1MB | Cấp quyền root cho user thường |
| `networkmanager` | ~10MB | Quản lý kết nối mạng (Wi-Fi, LAN, USB tethering) |
| `git` | ~40MB | Công cụ phiên bản, cần cho AUR helper (yay) |
| `base-devel` | ~100MB | Công cụ biên dịch: gcc, make, autoconf, pkg-config (cần cho AUR) |
| `btrfs-progs` | ~2MB | Công cụ quản lý BTRFS (btrfs subvolume, snapshot, scrub) |

### Giải thích chi tiết từng gói

#### base

Gói nền tảng không thể thiếu:
- `glibc`: Thư viện C chuẩn — mọi chương trình đều cần.
- `bash`: Bourne Again SHell — shell mặc định.
- `coreutils`: Lệnh cơ bản như `cp`, `mv`, `ls`, `rm`, `cat`.
- `pacman`: Trình quản lý gói.
- `systemd`: Init system và service manager.
- `linux-api-headers`: Header file cho kernel API.

#### linux

Kernel Linux. Arch dùng rolling release → kernel luôn là phiên bản mới nhất.
Tại thời điểm 25/06/2026, kernel 7.x.

**Tại sao không dùng `linux-lts`?**
- LTS kernel ổn định nhưng cũ hơn.
- Lenovo LOQ 15IAX9 có phần cứng mới (RTX 4050, Intel Alder Lake HX, Realtek
  RTL8852BE) → cần kernel mới nhất để hỗ trợ driver tốt nhất.
- `nvidia-open` DKMS cũng cần kernel headers tương ứng → kernel mới = driver mới.

#### linux-firmware

Chứa firmware blob cho phần cứng. Thiếu gói này:
- Wi-Fi không hoạt động (cần firmware cho RTL8852BE và Intel AX).
- GPU NVIDIA có thể không khởi tạo được.
- NVMe có thể chạy ở chế độ tương thích chậm.

#### intel-ucode

CPU Intel i5-12450HX là Alder Lake-HX. Intel phát hành microcode update để sửa
lỗi CPU (security vulnerabilities, errata).
- GRUB tự động load microcode vào initramfs nếu có gói này.
- Có thể kiểm tra microcode đã load: `dmesg | grep microcode`.

#### sof-firmware

Sound Open Firmware — firmware cho Intel DSP trên Alder Lake.
- Lenovo LOQ 15IAX9 dùng Intel SST (Smart Sound Technology) cho âm thanh.
- Thiếu gói này → không có âm thanh (loa trong, tai nghe 3.5mm).
- Đã bao gồm trong linux-firmware? Không — cần cài riêng.

#### vim

Text editor. Có thể thay bằng `nano` nếu quen nano hơn.
- Lưu ý: `vi` có sẵn trong base (gói `vi`), nhưng `vim` tốt hơn nhiều (syntax
  highlight, multiple windows, plugins).

#### sudo

Cho phép user thường chạy lệnh với quyền root qua `sudo`.
- Cần cấu hình `/etc/sudoers` (visudo) sau khi tạo user.

#### networkmanager

Dịch vụ quản lý mạng mặc định của Arch.
- Cung cấp `nmcli` (command line), `nmtui` (terminal UI).
- Hỗ trợ Wi-Fi, Ethernet, VPN, USB tethering.

#### git

Cần thiết cho:
- Clone AUR packages.
- Cài đặt yay (AUR helper).
- Quản lý dotfiles (sau này).

#### base-devel

Nhóm gói phát triển: `gcc`, `make`, `autoconf`, `automake`, `pkg-config`.
- Cần để biên dịch gói từ AUR.
- Không cần nếu bạn không dùng AUR, nhưng rất khuyến khích.

#### btrfs-progs

Công cụ quản lý BTRFS: `btrfs` (subvolume, snapshot, scrub, balance, device).

---

## Cách A: Cài thủ công

### Bước 1: Cập nhật archlinux-keyring (quan trọng)

```bash
pacman -Sy archlinux-keyring
```

**Tại sao cần bước này?**
- ISO có thể cũ (dù mới tải, keyring trong ISO có thể đã lỗi thời).
- Nếu keyring cũ → GPG signature không hợp lệ → pacstrap thất bại.
- `pacman -Sy` đồng bộ repository database và cập nhật keyring.

### Bước 2: Chạy pacstrap

```bash
pacstrap -K /mnt base linux linux-firmware intel-ucode sof-firmware \
  vim sudo networkmanager git base-devel btrfs-progs
```

Giải thích:
- `-K`: Initialize keyring trong hệ thống mới.
- `/mnt`: Thư mục đích (nơi các partition đã mount).
- Các gói: danh sách như trên, cách nhau bởi space.

### Bước 3: Chờ quá trình cài đặt

Pacstrap sẽ:
1. Tải gói từ mirror (khoảng 500MB-1GB).
2. Giải nén vào `/mnt`.
3. Chạy post-install scripts.

Thời gian: 2-10 phút tùy tốc độ mạng.

### Bước 4: Kiểm tra cài đặt

```bash
ls /mnt/bin/bash
ls /mnt/usr/bin/pacman
ls /mnt/usr/bin/btrfs
```

Kiểm tra dung lượng:
```bash
df -h /mnt
```

Kiểm tra kernel:
```bash
ls /mnt/boot/vmlinuz-linux
```

Nếu tất cả đều tồn tại → cài đặt thành công.

### Bước 5: Sau pacstrap — các bước tiếp theo

Sau khi pacstrap hoàn tất, chúng ta cần:
1. Sinh fstab (`genfstab -U /mnt >> /mnt/etc/fstab`).
2. Chroot vào hệ thống mới (`arch-chroot /mnt`).
3. Cấu hình locale, timezone, hostname.
4. Cấu hình network (enable NetworkManager).
5. Tạo user.
6. Cài GRUB bootloader.

---

## Cách B: Dùng Archinstall

### Bước 1: Chạy archinstall

```bash
archinstall
```

### Bước 2: Cấu hình packages trong archinstall

Khi đến mục `Packages`:
- Mặc định archinstall cài: `base`, `linux`, `linux-firmware`, ... (tương tự).
- **Thêm gói**: gõ tên gói cách nhau bằng space hoặc dấu phẩy.
- Danh sách gợi ý thêm cho Lenovo LOQ:
  ```
  intel-ucode sof-firmware vim sudo networkmanager git base-devel btrfs-progs
  ```

### Bước 3: Lưu ý

Archinstall không cài `base-devel` và `btrfs-progs` theo mặc định. Bạn phải thêm
thủ công trong menu `Packages`.

### Bước 4: Nếu dùng archinstall command line

```bash
archinstall --packages "base linux linux-firmware intel-ucode sof-firmware vim sudo networkmanager git base-devel btrfs-progs"
```

---

## Xử lý lỗi

### "Failed to install packages to /mnt"

Nguyên nhân thường gặp:
1. **Thiếu mount**: `lsblk` kiểm tra `/mnt` đã có filesystem chưa.
2. **Mất mạng**: `ping -c 3 archlinux.org` — nếu không ping được, dùng tethering.
3. **Keyring cũ**: `pacman -Sy archlinux-keyring` rồi thử lại.
4. **Mirror lỗi**: Kiểm tra `/etc/pacman.d/mirrorlist` — có thể mirror bị chết.
   ```bash
   reflector --latest 10 --protocol https --sort rate --save /etc/pacman.d/mirrorlist
   ```

### "Signature from X is unknown trust"

```bash
pacman -Sy archlinux-keyring
# Sau đó chạy lại pacstrap
```

### "error: failed to commit transaction (conflicting files)"

Xung đột file:
```bash
pacstrap -K /mnt base linux linux-firmware --overwrite "*"
# --overwrite "*" ghi đè tất cả file xung đột
# Chỉ dùng khi thực sự cần
```

### Hết dung lượng trong live environment

```bash
# Xóa cache pacman
pacman -Scc

# Kiểm tra dung lượng
df -h /mnt
df -h /
```

Nếu `/mnt` đầy:
- Subvolume chưa mount đúng (kiểm tra `mount | grep /mnt`).
- Dung lượng partition BTRFS không đủ (kiểm tra partition table).

---

## Tổng kết

- Base system đã được cài vào `/mnt`.
- Các gói quan trọng cho Lenovo LOQ đã được thêm: `sof-firmware`, `intel-ucode`,
  `btrfs-progs`, `base-devel`.
- Sẵn sàng cấu hình fstab, locale, network, user, và bootloader.
