# Tạo User và Cấp quyền Sudo

## Mục tiêu

Tạo user thường (không phải root), cấp quyền sudo, thiết lập password.

## Điều kiện tiên quyết

- Đã chroot vào hệ thống mới.
- Đã cấu hình locale và network.

## Kiến thức nền

### Tại sao không dùng root hàng ngày?

| Root | User thường |
|---|---|
| Toàn quyền — cài/gỡ/xóa bất kỳ file nào | Chỉ có quyền trên file của mình |
| `rm -rf /` → xóa toàn bộ hệ thống | `rm -rf /` → permission denied |
| Mọi ứng dụng chạy với quyền tối đa | Ứng dụng chạy với quyền giới hạn |
| Cần login riêng | Dùng `sudo` khi cần quyền root |

**Nguyên tắc an toàn**: Chỉ dùng root khi thực sự cần (cài gói, sửa cấu hình
hệ thống). Dùng user thường cho mọi hoạt động hàng ngày.

### Root vs User thường

| Đặc điểm | Root | User thường |
|---|---|---|
| Home | `/root` | `/home/archuser` |
| Shell mặc định | `/bin/bash` | `/bin/bash` |
| Sudo | Không cần | Cần |
| UID | 0 | 1000+ |

---

## Cách A: Cài thủ công

### Bước 1: Đặt password cho root

```bash
passwd
```

- Nhập password cho root (xác nhận 2 lần).
- Nên đặt phòng trường hợp sudo bị hỏng.
- Gõ password → không có gì hiện trên màn hình (tính năng bảo mật).

### Bước 2: Tạo user mới

```bash
useradd -m -G wheel -s /bin/bash archuser
```

Giải thích:
- `useradd`: Lệnh tạo user mới.
- `-m` (create home): Tạo `/home/archuser`.
- `-G wheel`: Thêm user vào group `wheel` (cần cho sudo).
- `-s /bin/bash`: Shell mặc định là Bash.
- `archuser`: Tên user. **Đổi thành tên bạn muốn** (vd: `hung`, `quang`, ...).

### Bước 3: Đặt password cho user

```bash
passwd archuser
```

- Nhập password (xác nhận 2 lần).
- Password nên mạnh: ít nhất 8 ký tự, có chữ hoa, số, ký tự đặc biệt.

### Bước 4: Cấp quyền sudo

Dùng `visudo` để sửa file `/etc/sudoers` một cách an toàn:

```bash
EDITOR=vim visudo
```

Tìm dòng:
```
# %wheel ALL=(ALL:ALL) ALL
```

Bỏ dấu `#` ở đầu:
```
%wheel ALL=(ALL:ALL) ALL
```

**Giải thích:**
- `EDITOR=vim`: Dùng vim làm editor (mặc định có thể là nano).
- `visudo`: Lệnh an toàn — kiểm tra syntax trước khi save.
- `%wheel`: Group wheel.
- `ALL=(ALL:ALL) ALL`: Cho phép chạy mọi lệnh với tư cách mọi user.

**Lưu ý an toàn**: Không dùng `NOPASSWD` trừ khi bạn hiểu rủi ro.

### Bước 5: Kiểm tra

```bash
# Kiểm tra user
id archuser

# Kiểm tra groups
groups archuser

# Kiểm tra home directory
ls -la /home/archuser

# Kiểm tra sudo syntax (từ root)
visudo -c
```

Output mong đợi:
```
# id archuser
uid=1000(archuser) gid=1000(archuser) groups=1000(archuser),998(wheel)

# groups archuser
archuser wheel

# visudo -c
/etc/sudoers: parsed OK
```

---

## Cách B: Dùng Archinstall

### Trong quá trình archinstall

Khi đến mục **`Users`**:
1. **Root password**: Nhập password cho root.
2. **Add user**: Nhập thông tin:
   - `Username`: `archuser` (hoặc tên bạn muốn).
   - `Password`: Nhập password.
   - `Groups`: Chọn `wheel` (và `audio`, `video`, `storage` nếu muốn).
   - `Shell`: `/bin/bash` (mặc định).

### Archinstall tự động xử lý:

- Tạo user với home directory.
- Thêm user vào group wheel.
- Cấu hình sudo (uncomment `%wheel ALL=(ALL:ALL) ALL`).
- Đặt password.

### Kiểm tra sau archinstall

```bash
arch-chroot /mnt
id archuser
groups archuser
visudo -c
```

---

## Các lệnh quản lý user hữu ích

```bash
# Đổi tên user
usermod -l newname oldname
usermod -d /home/newname -m newname

# Thêm user vào group khác
usermod -aG video,audio,storage,optical archuser

# Xóa user (cẩn thận)
userdel -r archuser

# Khóa/mở user
passwd -l archuser   # lock
passwd -u archuser   # unlock

# Đổi shell
chsh -s /bin/zsh archuser
```

---

## Bảo mật

### Đổi password sau lần đầu login

Sau khi reboot và login lần đầu:
```bash
passwd
```

### Kiểm tra ai có quyền sudo

```bash
cat /etc/sudoers
cat /etc/sudoers.d/* 2>/dev/null
```

### Không dùng root cho SSH

Nếu mở SSH, tắt root login:
```bash
echo "PermitRootLogin no" >> /etc/ssh/sshd_config
```

---

## Tổng kết

- User `archuser` đã được tạo với home directory và shell bash.
- User có quyền sudo qua group wheel.
- Password đã được đặt cho cả root và user.
- Sẵn sàng cài GRUB bootloader và reboot.
