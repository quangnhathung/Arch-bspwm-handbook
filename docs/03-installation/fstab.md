# Fstab

## Mục tiêu

Tạo file `/etc/fstab` để hệ thống tự động mount các partition và subvolume khi khởi động.

## Kiến thức nền

### fstab là gì?

`/etc/fstab` (File System Table) là file cấu hình của Linux chứa danh sách các
filesystem sẽ được tự động mount khi boot.

Mỗi dòng trong fstab có format:

```
<device>    <mountpoint>    <fstype>    <options>    <dump>    <pass>
```

| Field | Ý nghĩa |
|---|---|
| device | Thiết bị (UUID hoặc /dev/...) |
| mountpoint | Thư mục mount |
| fstype | Loại filesystem (btrfs, vfat, ...) |
| options | Mount options (compress=zstd, noatime, subvol=@, ...) |
| dump | Backup flag (0 = không backup) |
| pass | fsck order (0 = không check, 1 = root, 2 = others) |

### Tại sao dùng UUID?

UUID là mã định danh duy nhất của partition, không thay đổi ngay cả khi
thêm/bớt ổ cứng. Không dùng `/dev/nvme0n1p1` vì có thể thay đổi.

## Các bước thực hiện

### Bước 1: Sinh fstab

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

Giải thích:
- `genfstab`: Script sinh fstab tự động dựa trên mount hiện tại.
- `-U`: Dùng UUID thay vì device path.
- `/mnt`: Thư mục gốc của hệ thống mới.
- `>>`: Append vào file fstab.

### Bước 2: Kiểm tra nội dung

```bash
cat /mnt/etc/fstab
```

Kết quả mong đợi:

```
# /dev/nvme0n1p2
UUID=xxxx-xxxx-xxxx-xxxx-xxxx  /              btrfs   rw,noatime,compress=zstd,space_cache=v2,autodefrag,subvol=@ 0 0

# /dev/nvme0n1p2
UUID=xxxx-xxxx-xxxx-xxxx-xxxx  /home          btrfs   rw,noatime,compress=zstd,space_cache=v2,autodefrag,subvol=@home 0 0

# /dev/nvme0n1p2
UUID=xxxx-xxxx-xxxx-xxxx-xxxx  /var/log       btrfs   rw,noatime,compress=zstd,space_cache=v2,autodefrag,subvol=@log 0 0

# /dev/nvme0n1p2
UUID=xxxx-xxxx-xxxx-xxxx-xxxx  /var/cache/pacman/pkg btrfs rw,noatime,compress=zstd,space_cache=v2,autodefrag,subvol=@pkg 0 0

# /dev/nvme0n1p2
UUID=xxxx-xxxx-xxxx-xxxx-xxxx  /swap          btrfs   rw,noatime,space_cache=v2,nodatacow,subvol=@swap 0 0

# /dev/nvme0n1p1
UUID=yyyy-yyyy-yyyy-yyyy-yyyy  /efi           vfat    rw,noatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro 0 2
```

Kiểm tra:

```bash
vim /mnt/etc/fstab
```

Đảm bảo các mount options chính xác:

- `compress=zstd`: Có trên các subvolume BTRFS (trừ @swap).
- `nodatacow`: Có trên `@swap`.
- `subvol=@...`: Mỗi dòng đúng subvolume.
- `fmask=0022,dmask=0022`: FAT32 mount options chuẩn cho ESP.

### Bước 3: Kiểm tra UUID

```bash
blkid
```

So UUID trong fstab với output của blkid. Phải khớp.

## Swap entry trong fstab

Swap không có trong fstab lúc này. Sau khi tạo swapfile (trong chroot),
chúng ta sẽ thêm thủ công. Swapfile cần entry trong fstab:

```
/swap/swapfile  none  swap  sw  0  0
```

## Xác minh

```bash
# Kiểm tra các mount trong fstab có đúng không
mount -a --fake
# Nếu không có lỗi → fstab hợp lệ
```

## Nếu có lỗi

### genfstab không tạo đúng subvolume

Có thể genfstab không nhận diện đúng subvol option. Kiểm tra và sửa thủ công
bằng vim nếu cần.

### Thiếu dòng

Nếu thiếu dòng mount cho `/var/log` hoặc `/var/cache/pacman/pkg`, thêm bằng tay.

### Sai UUID

Kiểm tra blkid và sửa UUID trong fstab.

## File fstab mẫu hoàn chỉnh

```
# /etc/fstab: static file system information
# <file system> <dir> <type> <options> <dump> <pass>

UUID=ABC / btrfs rw,noatime,compress=zstd,space_cache=v2,autodefrag,subvol=@ 0 0
UUID=ABC /home btrfs rw,noatime,compress=zstd,space_cache=v2,autodefrag,subvol=@home 0 0
UUID=ABC /var/log btrfs rw,noatime,compress=zstd,space_cache=v2,autodefrag,subvol=@log 0 0
UUID=ABC /var/cache/pacman/pkg btrfs rw,noatime,compress=zstd,space_cache=v2,autodefrag,subvol=@pkg 0 0
UUID=ABC /swap btrfs rw,noatime,space_cache=v2,nodatacow,subvol=@swap 0 0
UUID=DEF /efi vfat rw,noatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro 0 2
/swap/swapfile none swap sw 0 0
```

(ABC và DEF là UUID thật — mỗi máy khác nhau, không copy mẫu này.)

## Tổng kết

- fstab đã được sinh với UUID và mount options chính xác.
- Kiểm tra và sửa thủ công nếu cần.
- Sẵn sàng chroot vào hệ thống mới.
