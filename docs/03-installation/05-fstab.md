# Fstab (File System Table)

## Mục tiêu

Tạo file `/etc/fstab` để hệ thống tự động mount các partition và subvolume khi khởi động.

## Điều kiện tiên quyết

- Đã cài base system bằng pacstrap.
- Các partition/subvolume đã mount đúng vào `/mnt`.
- **Chưa** chroot.

## Kiến thức nền

### fstab là gì?

`/etc/fstab` (File System Table) là file cấu hình của Linux chứa danh sách các
filesystem sẽ được tự động mount khi boot.

Mỗi dòng trong fstab có 6 trường:

```
<device>    <mountpoint>    <fstype>    <options>    <dump>    <pass>
```

| Trường | Ý nghĩa |
|---|---|
| device | Thiết bị — dùng `UUID=` (không dùng `/dev/nvme0n1p1` vì có thể thay đổi) |
| mountpoint | Thư mục sẽ mount (vd: `/`, `/home`, `/efi`) |
| fstype | Loại filesystem: `btrfs`, `vfat`, `swap` |
| options | Mount options: `compress=zstd`, `noatime`, `subvol=@`, ... |
| dump | Có backup bằng `dump` hay không (0 = không) |
| pass | Thứ tự kiểm tra fsck (0 = không check, 1 = root, 2 = others) |

### Tại sao dùng UUID?

UUID (Universally Unique Identifier) là mã định danh duy nhất cho mỗi partition.
- **Không thay đổi** khi thêm/bớt ổ cứng.
- **Không phụ thuộc vào tên device** (`/dev/nvme0n1` có thể thay đổi).
- **Duy nhất toàn cầu** — không có hai partition trùng UUID.

Lấy UUID:
```bash
blkid /dev/nvme0n1p1
blkid /dev/nvme0n1p2
```

### Mount options cho BTRFS

| Option | Subvolume | Ý nghĩa |
|---|---|---|
| `compress=zstd` | `@`, `@home`, `@log`, `@pkg` | Nén zstd |
| `noatime` | Tất cả | Tắt access time |
| `space_cache=v2` | Tất cả | Cache không gian trống |
| `nodatacow` | `@swap` | Tắt CoW cho swap |

### Mount options cho ESP (FAT32)

| Option | Ý nghĩa |
|---|---|
| `fmask=0022` | File permission mask (file = 755) |
| `dmask=0022` | Directory permission mask (dir = 755) |
| `codepage=437` | Bảng mã ký tự cho FAT |
| `iocharset=iso8859-1` | Character set |
| `shortname=mixed` | Cách xử lý tên file 8.3 |
| `errors=remount-ro` | Nếu lỗi, remount readonly |

---

## Cách A: Cài thủ công

### Bước 1: Sinh fstab tự động

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

Giải thích:
- `genfstab`: Script sinh fstab dựa trên mount hiện tại.
- `-U`: Dùng UUID (thay vì device path như `/dev/nvme0n1p2`).
- `/mnt`: Thư mục gốc của hệ thống mới.
- `>>`: Append vào file (không ghi đè).

### Bước 2: Kiểm tra nội dung fstab

```bash
cat /mnt/etc/fstab
```

Output mong đợi:
```
# /dev/nvme0n1p2
UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx  /  btrfs  rw,noatime,compress=zstd,space_cache=v2,autodefrag,subvol=@  0  0

# /dev/nvme0n1p2
UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx  /home  btrfs  rw,noatime,compress=zstd,space_cache=v2,autodefrag,subvol=@home  0  0

# /dev/nvme0n1p2
UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx  /var/log  btrfs  rw,noatime,compress=zstd,space_cache=v2,autodefrag,subvol=@log  0  0

# /dev/nvme0n1p2
UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx  /var/cache/pacman/pkg  btrfs  rw,noatime,compress=zstd,space_cache=v2,autodefrag,subvol=@pkg  0  0

# /dev/nvme0n1p2
UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx  /swap  btrfs  rw,noatime,space_cache=v2,nodatacow,subvol=@swap  0  0

# /dev/nvme0n1p1
UUID=XXXX-XXXX  /efi  vfat  rw,noatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro  0  2
```

Các UUID (`xxxxxxxx-xxxx-...` và `XXXX-XXXX`) là khác nhau trên mỗi máy.

### Bước 3: Kiểm tra UUID với blkid

```bash
blkid
```

So UUID trong fstab với output của blkid. Phải khớp chính xác.

Ví dụ output blkid:
```
/dev/nvme0n1p1: UUID="A8B6-4C2D" BLOCK_SIZE="512" TYPE="vfat" PARTUUID="..."
/dev/nvme0n1p2: UUID="a1b2c3d4-e5f6-7890-abcd-ef1234567890" UUID_SUB="..." TYPE="btrfs" PARTUUID="..."
```

### Bước 4: Kiểm tra mount options

Dùng `vim` để kiểm tra và sửa nếu cần:
```bash
vim /mnt/etc/fstab
```

Đảm bảo:
- `compress=zstd`: Có trên `@`, `@home`, `@log`, `@pkg`.
- **Không** có `compress=zstd` trên `@swap`.
- `nodatacow`: Có trên `@swap`.
- `subvol=@...`: Mỗi dòng đúng subvolume tương ứng.
- `fmask=0022,dmask=0022`: Có trên ESP.
- Dòng ESP có `pass = 2` (để fsck kiểm tra FAT32).

### Bước 5: Thêm swapfile entry (thủ công)

Swapfile chưa tồn tại (sẽ tạo sau khi chroot). Nhưng fstab cần có dòng cho nó:

```bash
echo "/swap/swapfile  none  swap  sw  0  0" >> /mnt/etc/fstab
```

### Bước 6: Xác minh fstab

```bash
# Kiểm tra cú pháp (không mount thật)
mount -a --fake -R /mnt

# Nếu không có lỗi → fstab hợp lệ
# Nếu có lỗi → kiểm tra từng dòng
```

### Bước 7: Kiểm tra file fstab hoàn chỉnh

```bash
cat /mnt/etc/fstab
```

Đảm bảo có đủ 7 dòng (6 subvolume/partition + 1 swap).

---

## Cách B: Dùng Archinstall

Archinstall tự động sinh fstab trong quá trình cài đặt. Bạn không cần chạy
`genfstab` thủ công.

### Archinstall xử lý fstab như thế nào?

1. Dựa trên mount points bạn đã cấu hình trong `Manual partitioning`.
2. Tự động sinh fstab với UUID.
3. Lưu vào `/mnt/etc/fstab`.

### Kiểm tra fstab sau archinstall

Sau khi archinstall hoàn tất (trước khi reboot):

```bash
cat /mnt/etc/fstab
```

Kiểm tra:
- Các mount options có chính xác không?
- Swapfile có được thêm không? (Archinstall không tự thêm swapfile entry —
  bạn phải thêm thủ công sau khi boot).
- Subvolume `@swap` có `nodatacow` không?

### Nếu cần sửa fstab sau archinstall

```bash
# Chroot vào hệ thống mới
arch-chroot /mnt

# Sửa fstab
vim /etc/fstab

# Kiểm tra
mount -a --fake
```

---

## File fstab mẫu (với UUID giả — không copy trực tiếp)

```
# /etc/fstab: static file system information
# <file system> <dir> <type> <options> <dump> <pass>

# BTRFS root
UUID=a1b2c3d4-e5f6-7890-abcd-ef1234567890  /        btrfs  rw,noatime,compress=zstd,space_cache=v2,autodefrag,subvol=@           0  0

# BTRFS home
UUID=a1b2c3d4-e5f6-7890-abcd-ef1234567890  /home    btrfs  rw,noatime,compress=zstd,space_cache=v2,autodefrag,subvol=@home       0  0

# BTRFS log
UUID=a1b2c3d4-e5f6-7890-abcd-ef1234567890  /var/log btrfs  rw,noatime,compress=zstd,space_cache=v2,autodefrag,subvol=@log         0  0

# BTRFS pkg cache
UUID=a1b2c3d4-e5f6-7890-abcd-ef1234567890  /var/cache/pacman/pkg  btrfs  rw,noatime,compress=zstd,space_cache=v2,autodefrag,subvol=@pkg  0  0

# BTRFS swap (nodatacow)
UUID=a1b2c3d4-e5f6-7890-abcd-ef1234567890  /swap    btrfs  rw,noatime,space_cache=v2,nodatacow,subvol=@swap                     0  0

# EFI System Partition
UUID=A8B6-4C2D  /efi  vfat  rw,noatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro  0  2

# Swapfile
/swap/swapfile  none  swap  sw  0  0
```

> **CẢNH BÁO**: UUID trong mẫu trên là giả. **Không copy**. Mỗi máy có UUID riêng.
> Chạy `blkid` để lấy UUID thật.

---

## Xác minh và sửa lỗi

### Lệnh kiểm tra tổng hợp

```bash
# Xem fstab
cat /mnt/etc/fstab

# Kiểm tra UUID
blkid | grep -E "nvme0n1p[12]"

# Xác minh mount options
mount -a --fake -R /mnt 2>&1

# Kiểm tra syntax (nếu có lỗi)
findmnt --fstab -R /mnt
```

### Lỗi thường gặp

#### genfstab không ra đúng subvol option

Genfstab đôi khi không thêm đúng option `subvol=@`. Kiểm tra và sửa thủ công
bằng vim.

Cần sửa: `subvol=/@` thành `subvol=@` (xóa dấu `/` nếu có).

#### Thiếu dòng mount

Nếu thiếu dòng cho `/var/log` hoặc `/var/cache/pacman/pkg`:
- Chưa mount đúng lúc chạy genfstab.
- Thêm thủ công bằng vim, copy từ dòng tương tự.

#### Sai UUID

- Kiểm tra `blkid` và sửa UUID trong fstab.
- Lưu ý: BTRFS partition có cả `UUID` và `UUID_SUB`. Dùng `UUID` (không phải
  `UUID_SUB`) trong fstab.

#### Swapfile dòng sai

Dòng swapfile phải là:
```
/swap/swapfile  none  swap  sw  0  0
```

Không dùng UUID cho swapfile (swapfile là file, không phải partition).

---

## Tổng kết

- fstab đã được sinh với UUID và mount options chính xác.
- Swapfile entry đã được thêm thủ công.
- Đã kiểm tra `mount -a --fake` — không lỗi.
- Sẵn sàng chroot vào hệ thống mới để cấu hình locale, network, user, GRUB.
