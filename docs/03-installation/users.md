# Tạo User và Cấp quyền Sudo

## Mục tiêu

Tạo user thường (không phải root), cấp quyền sudo, và thiết lập password.

## Kiến thức nền

### Tại sao không dùng root hàng ngày?

- **Bảo mật**: Nếu dùng root, mọi ứng dụng đều có toàn quyền.
- **An toàn**: Lỡ gõ `rm -rf /` sẽ xóa toàn bộ hệ thống. Với user thường,
  chỉ xóa được file của mình.
- **Best practice**: Unix truyền thống dùng root chỉ cho quản trị.

### Root vs User thường

| Đặc điểm | Root | User thường |
|---|---|---|
| Quyền | Vô hạn | Giới hạn |
| Sudo | Không cần | Cần password |
| Home | /root | /home/username |
| Shell mặc định | /bin/bash | /bin/bash |

## Các bước thực hiện

### Bước 1: Đặt password cho root (không bắt buộc)

```bash
passwd
```

Nhập password cho root. Có thể để trống nếu không có nhu cầu dùng root trực tiếp.
Tuy nhiên, nên đặt phòng trường hợp sudo hỏng.

### Bước 2: Tạo user mới

```bash
useradd -m -G wheel -s /bin/bash archuser
```

Giải thích:
- `useradd`: Lệnh tạo user mới.
- `-m`: Tạo home directory (`/home/archuser`).
- `-G wheel`: Thêm user vào group `wheel`. Group wheel được sudo cho phép
  chạy lệnh với quyền root.
- `-s /bin/bash`: Shell mặc định là bash.
- `archuser`: Tên user. Bạn có thể đổi tên khác.

### Bước 3: Đặt password cho user

```bash
passwd archuser
```

Nhập password (xác nhận 2 lần). Lưu ý: khi gõ password, không có gì hiện trên màn hình
(đây là tính năng bảo mật mặc định).

### Bước 4: Cấp quyền sudo

```bash
EDITOR=vim visudo
```

Tìm và bỏ comment dòng (xóa dấu `#` ở đầu):

```
%wheel ALL=(ALL:ALL) ALL
```

Giải thích:
- `EDITOR=vim`: Dùng vim để edit (mặc định có thể là nano).
- `visudo`: Lệnh an toàn để edit sudoers file. Nó kiểm tra syntax trước khi save,
  tránh lock mình khỏi sudo.
- `%wheel`: Group wheel.
- `ALL=(ALL:ALL) ALL`: Cho phép chạy mọi lệnh với tư cách mọi user.

Cuối cùng, dòng sẽ là:

```
%wheel ALL=(ALL:ALL) ALL
```

**Lưu ý**: Nếu muốn sudo không cần password (tiện nhưng kém an toàn hơn):

```
%wheel ALL=(ALL:ALL) NOPASSWD: ALL
```

### Bước 5: Kiểm tra

```bash
# Kiểm tra user đã được tạo
id archuser

# Kiểm tra group
groups archuser

# Output phải bao gồm "wheel"
```

## Các lệnh hữu ích liên quan đến user

```bash
# Đổi tên user
usermod -l newname oldname

# Thêm user vào group khác
usermod -aG video,audio,storage,optical archuser

# Xóa user
userdel -r archuser
```

## Nếu có lỗi

### "useradd: command not found"

Trong chroot, useradd phải có sẵn. Nếu không, kiểm tra base package.

### "visudo: syntax error"

Nếu lỡ làm hỏng sudoers, fix bằng cách:

```bash
# Chạy với tư cách root
pkexec visudo
# Hoặc boot vào live environment, mount và sửa trực tiếp
```

## Tổng kết

- User `archuser` đã được tạo với home directory.
- User có quyền sudo qua group wheel.
- Password đã được đặt.

Sẵn sàng cài bootloader.
