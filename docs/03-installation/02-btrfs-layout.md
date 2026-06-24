# BTRFS Layout và Subvolume

## Mục tiêu

Định dạng partition BTRFS và tạo cấu trúc subvolume chuẩn cho hệ thống.

## Kiến thức nền: BTRFS là gì?

BTRFS (B-tree File System, thường đọc là "Butter FS") là filesystem hiện đại của
Linux với các tính năng vượt trội:

### Copy-on-Write (CoW)

Khi bạn sửa một file, BTRFS không ghi đè lên dữ liệu cũ ngay lập tức. Nó tạo một
bản sao mới của các block bị thay đổi. Điều này cho phép:
- **Snapshot tức thì**: Chụp trạng thái hệ thống tại một thời điểm mà không tốn
  thêm dung lượng (chỉ lưu các block thay đổi).
- **Rollback nhanh**: Quay lại trạng thái cũ nếu cập nhật có lỗi.
- **Chống mất dữ liệu**: Nếu hệ thống crash trong khi ghi, dữ liệu cũ vẫn còn.

### Subvolume

Subvolume là một "thư mục logic" bên trong BTRFS có thể:
- Mount riêng biệt với mount options riêng.
- Snapshot độc lập với các subvolume khác.
- Có quota riêng (nếu cần).

Không giống như partition, subvolume **chia sẻ không gian trống** — không cần
cấp phát trước dung lượng.

### Snapshot

Snapshot là ảnh chụp trạng thái của một subvolume tại một thời điểm. Ban đầu
snapshot không tốn thêm dung lượng (chỉ metadata). Dung lượng chỉ tăng khi dữ
liệu thay đổi.

### Compression (Nén)

BTRFS hỗ trợ nén dữ liệu trong suốt. Chúng ta dùng `zstd` (chuẩn nén hiện đại,
tỉ lệ nén cao, tốc độ nhanh). Lợi ích:
- Tiết kiệm dung lượng ổ cứng.
- Giảm lượng ghi vào NVMe (kéo dài tuổi thọ).
- Tốc độ đọc/ghi có thể nhanh hơn (vì đọc/ghi ít dữ liệu hơn).

### Checksum (Tổng kiểm)

BTRFS tự động tính checksum cho mọi block dữ liệu và metadata. Khi đọc, nó kiểm
tra checksum — nếu lỗi (bit rot), nó tự động sửa nếu có bản sao (RAID1) hoặc báo
lỗi.

## Tại sao dùng BTRFS thay vì ext4?

| Tính năng | BTRFS | ext4 |
|---|---|---|
| Snapshot | Có (tức thì, gần như không tốn dung lượng) | Không |
| Subvolume | Có (mount riêng, snapshot riêng) | Không |
| Nén trong suốt (zstd) | Có | Không |
| Checksum data + metadata | Có | Chỉ metadata |
| Tự động sửa lỗi (self-heal) | Có (với RAID) | Không |
| Thay đổi kích thước online | Có (grow/shrink) | Có (grow, shrink khó) |
| Thêm/bớt thiết bị online | Có | Không |
| Deduplication | Có (bằng công cụ bên ngoài) | Không |
| Độ phức tạp | Cao hơn | Thấp |
| Độ chín muồi | Rất ổn định (từ kernel 6.x+) | Rất ổn định |

**Kết luận**: Với desktop cá nhân, BTRFS cho phép snapshot trước khi cập nhật
lớn (`pacman -Syu`) → rollback nhanh nếu có lỗi. ext4 không làm được điều này.

## Cấu trúc subvolume

Chúng ta tạo **5 subvolume** riêng:

```
BTRFS partition (/dev/nvme0n1p2)
├── @               → /                    (root filesystem)
├── @home           → /home                (dữ liệu người dùng)
├── @log            → /var/log             (log hệ thống)
├── @pkg            → /var/cache/pacman/pkg (cache gói pacman)
└── @swap           → /swap                (swapfile)
```

### Giải thích từng subvolume

#### `@` — Root filesystem

Chứa toàn bộ hệ thống trừ các thư mục đã được tách riêng.
- Khi snapshot `@`, bạn chụp toàn bộ hệ thống (cấu hình, ứng dụng đã cài).
- **Không chứa**: `/home`, `/var/log`, `/var/cache/pacman/pkg`, `/swap`.

#### `@home` — Home directory

Chứa dữ liệu cá nhân (Desktop, Documents, Downloads, `.config`, `.local`, v.v.).
Tách riêng vì:
- Snapshot root không ảnh hưởng đến dữ liệu cá nhân.
- Có thể cài lại hệ thống (xóa `@`) mà giữ nguyên `/home`.
- Dữ liệu cá nhân thay đổi liên tục — không muốn snapshot root phình to.

#### `@log` — System logs

Chứa log hệ thống tại `/var/log`.
- Log thay đổi liên tục (journald ghi rất nhiều).
- Tách riêng để snapshot root gọn nhẹ hơn.
- Thường không cần snapshot log.

#### `@pkg` — Pacman cache

Chứa file `.pkg.tar.zst` đã tải tại `/var/cache/pacman/pkg`.
- Cache có thể rất lớn (nhiều GB) sau nhiều lần cập nhật.
- Tách riêng để không ảnh hưởng snapshot root.
- Dễ dọn dẹp (`pacman -Sc`) mà không ảnh hưởng hệ thống.

#### `@swap` — Swapfile

Chứa file swap duy nhất tại `/swap/swapfile`.
- **Swap và CoW không tương thích**: Nếu swapfile nằm trong subvolume có CoW,
  kernel sẽ cảnh báo và từ chối dùng.
- Subvolume `@swap` được mount với option `nodatacow` để tắt CoW.
- Tách riêng để các snapshot không chứa swapfile (swapfile rất lớn, thay đổi
  liên tục).

## Mount options cho laptop NVMe

Các option tối ưu cho Lenovo LOQ (NVMe + laptop):

| Option | Giá trị | Ý nghĩa |
|---|---|---|
| `compress` | `zstd` | Nén dữ liệu với zstd. Tiết kiệm dung lượng, giảm ghi NVMe, kéo dài tuổi thọ ổ. |
| `noatime` | — | Không cập nhật access time khi đọc file. Giảm ghi NVMe đáng kể. |
| `space_cache` | `v2` | Cache không gian trống trên BTRFS. Phiên bản 2 (v2) ổn định và hiệu quả hơn. |
| `autodefrag` | — | Tự động chống phân mảnh cho file nhỏ. Hữu ích cho database, file cấu hình. |
| `nodatacow` | — | Tắt Copy-on-Write. Chỉ dùng cho `@swap` (swap không tương thích CoW). |

---

## Cách A: Cài thủ công

### Bước 1: Format partition BTRFS

```bash
mkfs.btrfs -f /dev/nvme0n1p2
```

Giải thích:
- `mkfs.btrfs`: Tạo filesystem BTRFS.
- `-f` (force): Ghi đè nếu partition đã có dữ liệu hoặc filesystem cũ.
- `/dev/nvme0n1p2`: Partition Linux đã tạo ở bước trước.

Kiểm tra:
```bash
blkid /dev/nvme0n1p2  # TYPE="btrfs"
```

### Bước 2: Mount tạm để tạo subvolume

```bash
mount /dev/nvme0n1p2 /mnt
```

Cần mount partition BTRFS trước để có thể tạo subvolume bên trong.

### Bước 3: Tạo các subvolume

```bash
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@log
btrfs subvolume create /mnt/@pkg
btrfs subvolume create /mnt/@swap
```

Giải thích: `btrfs subvolume create <path>` tạo một subvolume tại path đó.

### Bước 4: Kiểm tra subvolume

```bash
btrfs subvolume list /mnt
```

Output mong đợi:
```
ID 256 gen 3 top level 5 path @
ID 257 gen 4 top level 5 path @home
ID 258 gen 5 top level 5 path @log
ID 259 gen 6 top level 5 path @pkg
ID 260 gen 7 top level 5 path @swap
```

### Bước 5: Unmount

```bash
umount /mnt
```

---

## Cách B: Dùng Archinstall

### Nếu dùng Archinstall với Manual partitioning:

Archinstall không tự động tạo subvolume tùy chỉnh (`@`, `@home`, ...). Nó chỉ tạo
một subvolume mặc định (thường là `@`) hoặc mount trực tiếp vào root.

**Sau khi Archinstall hoàn tất**, bạn có thể cấu hình thêm subvolume thủ công
(nếu cần). Tuy nhiên, với người dùng Archinstall, cấu trúc subvolume mặc định
của Archinstall là đủ dùng.

### Để có subvolume tùy chỉnh với Archinstall:

1. Chạy `archinstall` → chọn `Manual partitioning`.
2. Chọn ổ `/dev/nvme0n1` → partition `nvme0n1p2` → filesystem `btrfs`.
3. Subvolume: Archinstall cho phép thêm subvolume trong menu partition.
   - Nhập tên subvolume (vd: `@`, `@home`, `@log`, `@pkg`, `@swap`).
   - Chỉ định mountpoint tương ứng.
4. Nếu archinstall không hỗ trợ subvolume tùy chỉnh trong giao diện,
   **cài thủ công các bước còn lại sau khi chạy archinstall**.

### Giải pháp Hybrid:

Dùng Archinstall để phân vùng và cài base, sau đó tự sửa subvolume:

1. Chạy `archinstall` với BTRFS (tự động tạo `@` và `@home`).
2. Sau khi cài xong, boot vào hệ thống mới.
3. Tạo thêm subvolume `@log`, `@pkg`, `@swap` bằng lệnh `btrfs subvolume create`.
4. Sửa `/etc/fstab` để mount chúng.

---

## Cấu trúc cuối cùng

```
/dev/nvme0n1p2 (BTRFS, 475.9G)
├── @               → /                    (root)
├── @home           → /home                (dữ liệu user)
├── @log            → /var/log             (log)
├── @pkg            → /var/cache/pacman/pkg (cache)
└── @swap           → /swap                (swapfile, nodatacow)
```

## Lưu ý về swap

Không tạo swapfile ở bước này. File swap sẽ được tạo sau khi đã chroot vào hệ
thống mới, vì lúc đó mới có các công cụ cần thiết (`dd`, `mkswap`, `swapon`).
Subvolume `@swap` sẽ được mount với option `nodatacow`.

## Tổng kết

- Partition BTRFS đã được format.
- 5 subvolume đã được tạo với cấu trúc rõ ràng.
- Sẵn sàng để mount đúng vị trí và cài base system.
