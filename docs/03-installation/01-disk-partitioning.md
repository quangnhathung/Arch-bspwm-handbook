# Phân vùng ổ đĩa (Disk Partitioning)

## Mục tiêu

Xóa sạch toàn bộ dữ liệu cũ trên ổ NVMe, tạo phân vùng mới với **GPT (GUID Partition Table)** cho UEFI.

## CẢNH BÁO NGHIÊM TRỌNG — MẤT DỮ LIỆU

> **Thao tác dưới đây sẽ xóa SẠCH toàn bộ dữ liệu trên ổ cứng.**
>
> - Windows, tất cả ứng dụng, tài liệu, nhạc, video, ảnh → mất hết.
> - Không thể khôi phục bằng phần mềm thông thường.
> - Kiểm tra kỹ bạn đang thao tác trên Ổ NÀO (`lsblk`).
> - Nếu có nhiều ổ cứng (vd: ổ C: + ổ D:), kiểm tra thật kỹ.
> - **Sao lưu dữ liệu quan trọng trước khi bắt đầu.**

## Sơ đồ phân vùng cuối cùng

```
nvme0n1          (512GB NVMe)
├─ nvme0n1p1     (1GB, EFI System — FAT32)     → /efi
└─ nvme0n1p2     (~511GB, Linux — BTRFS)        → /
```

Swap là file `/swap/swapfile` bên trong BTRFS (không tạo swap partition riêng).

## Kiến thức nền

### GPT vs MBR

| Tính năng | GPT | MBR |
|---|---|---|
| Đời | Mới (UEFI bắt buộc) | Cũ (Legacy BIOS) |
| Ổ > 2TB | Hỗ trợ | Không |
| Số partition tối đa | 128 | 4 primary |
| Backup partition table | Có (backup GPT ở cuối ổ) | Không |
| CRC32 checksum | Có | Không |

Chúng ta dùng **GPT** vì máy Lenovo LOQ 15IAX9 dùng UEFI.

### EFI System Partition (ESP)

Phân vùng đặc biệt chứa bootloader (GRUB). Yêu cầu:
- **FAT32** — định dạng bắt buộc cho UEFI.
- **Dung lượng**: 1GB (dư dùng, 500MB cũng đủ nhưng 1GB an toàn hơn).
- **Type GUID**: `C12A7328-F81F-11D2-BA4B-00A0C93EC93B` (EFI System).
- **Cờ**: `esp` (hoặc `boot` trong một số công cụ).

### Linux Filesystem Partition

Phân vùng chính chứa toàn bộ hệ thống. Format BTRFS với subvolumes.

### Swapfile thay vì Swap Partition

| Tiêu chí | Swap Partition | Swapfile |
|---|---|---|
| Thay đổi kích thước | Phải xóa partition, tạo lại | Dễ dàng resize |
| Snapshot BTRFS | Không ảnh hưởng | Cần subvolume riêng (`nodatacow`) |
| Linh hoạt | Thấp | Cao |
| Hiệu suất | Tương đương | Tương đương |

---

## Cách A: Cài thủ công (Manual Install)

### Bước 1: Xác định ổ cứng

```bash
lsblk
```

Tìm ổ có kích thước ~512GB, thường là `nvme0n1` trên Lenovo LOQ.

Kiểm tra thêm:
```bash
ls -la /dev/nvme*
cat /sys/block/nvme0n1/device/model
```

Output mẫu:
```
Micron 2400 MTFDKBA512QCH
```

### Bước 2: Xóa sạch ổ cứng

Có 3 công cụ để phân vùng, chọn **một** trong các cách dưới đây:

#### Cách 2a: `cfdisk` (đơn giản, trực quan)

```bash
cfdisk /dev/nvme0n1
```

1. Nếu thấy "Select label type" → chọn `gpt`.
2. Nếu đã có partition cũ → chọn từng partition → `Delete` → xóa hết cho đến khi còn `Free Space` duy nhất.
3. **Chọn `Write` → gõ `yes` → `Quit`.**

#### Cách 2b: `sgdisk` (nhanh, scriptable)

```bash
# Xóa tất cả partition cũ
sgdisk --zap-all /dev/nvme0n1

# Tạo GPT mới
sgdisk --clear /dev/nvme0n1
```

#### Cách 2c: `gdisk` (chi tiết, kiểm soát từng bước)

```bash
gdisk /dev/nvme0n1
```

Trong gdisk:
- `x` → `z` → `y` → `y` (xóa GPT và khởi tạo mới, wipe sạch).
- Sau đó chạy lại `gdisk /dev/nvme0n1` để tạo partition mới.

### Bước 3: Tạo EFI System Partition (ESP) — 1GB

#### Với `cfdisk`:
1. Chọn `Free Space` → `New`.
2. Partition size: `1G` → Enter.
3. Type: `EFI System` (hoặc gõ mã `ef00` nếu được hỏi).

#### Với `sgdisk`:
```bash
sgdisk --new=1:0:+1G --typecode=1:ef00 --change-name=1:"EFI System" /dev/nvme0n1
```

#### Với `gdisk`:
1. `n` → Partition number: `1` → First sector: (Enter để mặc định) → Size in sectors: `+1G` → Hex code: `ef00`.

### Bước 4: Tạo Linux Filesystem Partition — BTRFS (phần còn lại)

#### Với `cfdisk`:
1. Chọn `Free Space` còn lại → `New`.
2. Size: Enter (dùng hết phần còn lại).
3. Type: `Linux filesystem` (mặc định, mã `8300`).

#### Với `sgdisk`:
```bash
sgdisk --new=2:0:0 --typecode=2:8300 --change-name=2:"Linux BTRFS" /dev/nvme0n1
```

#### Với `gdisk`:
1. `n` → Partition number: `2` → First sector: (Enter) → Last sector: (Enter để lấy hết) → Hex code: `8300`.

### Bước 5: Ghi và thoát

- `cfdisk`: Chọn `Write` → `yes` → `Quit`.
- `sgdisk`: Không cần ghi riêng (tự động ghi ngay).
- `gdisk`: `w` → `y`.

### Bước 6: Kiểm tra partition table

```bash
lsblk
```

Kết quả mong đợi:
```
nvme0n1     259:0    0 476.9G  0 disk
├─nvme0n1p1 259:1    0     1G  0 part
└─nvme0n1p2 259:2    0 475.9G  0 part
```

Kiểm tra chi tiết:
```bash
sgdisk --print /dev/nvme0n1
```

Output:
```
Disk /dev/nvme0n1: ... sectors, 476.9 GiB
Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         2099199   1.0 GiB     EF00  EFI System
   2         2099200       1000215182   475.9 GiB  8300  Linux BTRFS
```

---

## Cách B: Dùng Archinstall (Guided Installer)

Archinstall có sẵn trên Arch ISO. Nó tự động xử lý phân vùng.

### Bước 1: Chạy archinstall

```bash
archinstall
```

### Bước 2: Cấu hình disk partitioning trong archinstall

Khi đến mục `Disk configuration`:

1. Chọn **`Manual partitioning`** (không chọn `Wipe all drives` tự động để kiểm soát).
2. Chọn ổ `/dev/nvme0n1`.
3. Tạo partition:
   - **Partition 1**: size `1GiB`, filesystem `fat32`, mountpoint `/boot` (hoặc `/efi`).
   - **Partition 2**: size còn lại (max), filesystem `btrfs`, mountpoint `/`.

> **Lưu ý**: Archinstall mặc định gắn ESP tại `/boot`. Nếu bạn muốn ESP tại `/efi`, có thể chỉnh sau. Trong tài liệu này chúng ta dùng `/efi`.

### Bước 3: Hoặc chọn "Best-effort" (tự động)

Nếu chọn **`Best-effort`** với `btrfs`:
- Archinstall tự động tạo partition ESP 512MB.
- Partition BTRFS chiếm phần còn lại.
- Có thể không đủ 1GB cho ESP và không tạo subvolume tùy chỉnh.
  → **Không khuyến nghị**. Dùng `Manual partitioning` để kiểm soát.

### Bước 4: Tiếp tục archinstall

Sau khi phân vùng xong, archinstall sẽ hỏi các cấu hình khác (mạng, user, desktop).
Bạn có thể thoát archinstall sau bước phân vùng nếu muốn làm thủ công các bước sau.

---

## Xác minh partition table

```bash
# Kiểm tra partition type
blkid /dev/nvme0n1p1
blkid /dev/nvme0n1p2

# Kiểm tra chi tiết partition
fdisk -l /dev/nvme0n1
```

Lúc này `nvme0n1p1` và `nvme0n1p2` chưa có filesystem (chưa format). Chúng ta sẽ format ở các bước sau.

## Xác minh alignment (4K sector)

```bash
sgdisk --verify /dev/nvme0n1
```

Output mong đợi: No problems found.

---

## Xử lý lỗi thường gặp

### "Device or resource busy"

Ổ đã được mount tự động bởi live environment:
```bash
umount -R /mnt 2>/dev/null
# Hoặc
swapoff -a
```

### "Partition table cannot be written"

Ổ đang được sử dụng bởi process nào đó hoặc đã được mount:
```bash
lsof /dev/nvme0n1
# Reboot live environment nếu cần
```

### "GPT PMBR size mismatch"

Dùng `gdisk` để sửa:
```bash
gdisk /dev/nvme0n1
```
- `w` → `y` (ghi lại partition table, tự động sửa mismatch).

---

## Tổng kết

| Hạng mục | Giá trị |
|---|---|
| Partition table | GPT |
| Ổ cứng | `/dev/nvme0n1` (512GB NVMe) |
| ESP | `/dev/nvme0n1p1` — 1GB FAT32 |
| Linux | `/dev/nvme0n1p2` — ~511GB BTRFS |
| Swap | Swapfile (`/swap/swapfile`) — không có swap partition |

Ổ cứng đã sẵn sàng để format và mount.
