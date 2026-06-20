# Phân vùng ổ đĩa

## Mục tiêu

Xóa sạch toàn bộ dữ liệu cũ trên ổ NVMe, tạo phân vùng mới với GPT cho UEFI.

## Điều kiện tiên quyết

- Đã boot vào Arch live environment.
- Đã xác định đúng tên ổ cứng (thường là `/dev/nvme0n1`).
- Không còn dữ liệu quan trọng trên máy.

## CẢNH BÁO NGHIÊM TRỌNG

> **Thao tác dưới đây sẽ xóa SẠCH toàn bộ dữ liệu trên ổ cứng.**
>
> - Windows, tất cả ứng dụng, tài liệu, nhạc, video, ảnh → mất hết.
> - Không thể khôi phục.
> - Kiểm tra kỹ bạn đang thao tác trên Ổ NÀO.
> - Nếu có nhiều ổ cứng (vd: ổ C: + ổ D:), kiểm tra thật kỹ.

## Giải thích kiến thức nền

### Partition Table: GPT vs MBR

| | GPT | MBR |
|---|---|---|
| Đời | Mới (UEFI cần GPT) | Cũ (Legacy BIOS) |
| Ổ >2TB | Có | Không |
| Số partition | 128 | 4 primary |
| Redundancy | Backup GPT ở cuối ổ | Không |

Chúng ta dùng **GPT** vì máy dùng UEFI.

### EFI System Partition (ESP)

Phân vùng đặc biệt chứa bootloader (GRUB). Yêu cầu:
- FAT32
- Size: 1GB (dư dùng, có thể 500MB nhưng 1GB an toàn hơn)
- Type: EFI System (C12A7328-F81F-11D2-BA4B-00A0C93EC93B)

### Linux Filesystem Partition

Phân vùng chính chứa hệ thống. Format: BTRFS.

### Swapfile

Chúng ta dùng swapfile (file trong BTRFS) thay vì swap partition.
Linh hoạt hơn, dễ resize.

## Sơ đồ phân vùng cuối cùng

```
nvme0n1          (512GB NVMe)
├─ nvme0n1p1     (1GB, EFI System — FAT32)     → /efi
└─ nvme0n1p2     (511GB, Linux — BTRFS)         → /
```

Swap sẽ là file `/swap/swapfile` bên trong BTRFS.

## Các bước thực hiện

### Bước 1: Xác định ổ cứng

```bash
lsblk
```

Tìm ổ có kích thước ~512GB. Thường là `nvme0n1`.

Kiểm tra thêm:

```bash
ls -la /dev/nvme*
```

### Bước 2: Xóa sạch partition cũ

Mở công cụ phân vùng:

```bash
cfdisk /dev/nvme0n1
```

**Nếu dùng cfdisk:**
1. Nếu thấy "Select label type" → chọn `gpt`.
2. Nếu đã có partition → chọn từng partition → `Delete` → xóa hết.
3. Khi còn `Free Space` duy nhất → xong.

**Cảnh báo**: Nếu lỡ xóa nhầm ổ, dừng lại ngay, không save.

### Bước 3: Tạo EFI System Partition

Trong cfdisk:
1. Chọn `Free Space` → `New` → Partition size: `1G` → Enter.
2. Type: `EFI System`.

Hoặc dùng sgdisk (thay thế cfdisk):

```bash
# Xóa tất cả partition
sgdisk --zap-all /dev/nvme0n1
# Tạo GPT mới
sgdisk --clear /dev/nvme0n1
# Tạo ESP 1GB
sgdisk --new=1:0:+1G --typecode=1:ef00 /dev/nvme0n1
# Tạo partition BTRFS chiếm phần còn lại
sgdisk --new=2:0:0 --typecode=2:8300 /dev/nvme0n1
```

### Bước 4: Tạo BTRFS partition

Trong cfdisk:
1. Chọn `Free Space` còn lại → `New` → `Enter` (dùng hết dung lượng còn lại).
2. Type: `Linux filesystem` (mặc định).

### Bước 5: Ghi và thoát

Trong cfdisk:
1. Chọn `Write` → gõ `yes` → Enter.
2. Select `Quit`.

### Bước 6: Kiểm tra kết quả

```bash
lsblk
```

Kết quả mong đợi:

```
nvme0n1     259:0    0 476.9G  0 disk
├─nvme0n1p1 259:1    0     1G  0 part
└─nvme0n1p2 259:2    0 475.9G  0 part
```

## Xác minh partition type

```bash
blkid /dev/nvme0n1p1
blkid /dev/nvme0n1p2
```

`nvme0n1p1` sẽ có `PARTUUID` và chưa có filesystem (vì chưa format).

## Nếu có lỗi

### "Device or resource busy"

- Ổ đã được mount → unmount trước: `umount -R /mnt`.
- Hoặc reboot live environment.

### "Partition table cannot be written"

- Ổ đang được sử dụng bởi process nào đó.
- Dùng `fdisk -l` để xem, reboot nếu cần.

## Tổng kết

- Ổ cứng đã được xóa sạch và phân vùng lại.
- 1 partition ESP (1GB FAT32) cho UEFI.
- 1 partition BTRFS cho toàn bộ phần còn lại.
- Sẵn sàng để format và mount.
