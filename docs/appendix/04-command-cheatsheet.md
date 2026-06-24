# Command Cheatsheet — Arch Linux + bspwm

**Ngày: 25/06/2026 — Kernel 7.x** — Lenovo LOQ 15IAX9 (Intel + RTX 4050)

---

## Pacman

| Lệnh | Chức năng |
|---|---|
| `pacman -Syu` | Cập nhật toàn bộ hệ thống |
| `pacman -S <gói>` | Cài gói |
| `pacman -Rs <gói>` | Xoá gói + dependencies không dùng |
| `pacman -Rns <gói>` | Xoá gói + config + dependencies |
| `pacman -Ss <từ>` | Tìm gói trong repository |
| `pacman -Qs <từ>` | Tìm gói đã cài |
| `pacman -Q` | Liệt kê tất cả gói đã cài |
| `pacman -Qi <gói>` | Chi tiết gói đã cài |
| `pacman -Si <gói>` | Chi tiết gói trong repo |
| `pacman -Ql <gói>` | Danh sách file của gói |
| `pacman -Qo <file>` | Gói nào sở hữu file này |
| `pacman -F <file>` | Tìm gói chứa file |
| `pacman -Qtdq` | Liệt kê orphan packages |
| `pacman -U <file>.pkg.tar.zst` | Cài từ file `.pkg.tar.zst` |
| `paccache -r` | Dọn cache giữ 3 bản gần nhất |
| `paccache -rk1` | Dọn cache chỉ giữ 1 bản |

> `paccache` cần gói `pacman-contrib`: `sudo pacman -S pacman-contrib`

## Yay

| Lệnh | Chức năng |
|---|---|
| `yay -Syu` | Cập nhật cả AUR + chính thức |
| `yay -S <gói>` | Cài gói (AUR hoặc chính thức) |
| `yay -Sa <gói>` | Cài gói từ AUR |
| `yay -Ss <từ>` | Tìm gói (AUR + chính thức) |
| `yay -Ssa <từ>` | Tìm gói (chỉ AUR) |
| `yay -Sc` | Dọn cache yay |

> `yay` biên dịch từ nguồn (chậm). `yay-bin` là pre-compiled (nhanh hơn).

## Systemctl

| Lệnh | Chức năng |
|---|---|
| `systemctl start <dịch-vụ>` | Bắt đầu dịch vụ |
| `systemctl stop <dịch-vụ>` | Dừng dịch vụ |
| `systemctl restart <dịch-vụ>` | Khởi động lại dịch vụ |
| `systemctl enable <dịch-vụ>` | Tự động khởi động cùng hệ thống |
| `systemctl disable <dịch-vụ>` | Tắt tự động khởi động |
| `systemctl status <dịch-vụ>` | Xem trạng thái |
| `systemctl --failed` | Liệt kê dịch vụ bị lỗi |
| `systemctl list-units` | Liệt kê tất cả unit |
| `systemctl --user ...` | Dịch vụ user (thay vì system) |
| `systemctl enable --now ...` | Enable + start ngay lập tức |

## Journalctl

| Lệnh | Chức năng |
|---|---|
| `journalctl -p 3 -xb` | Log lỗi (priority 3 trở lên) |
| `journalctl -u <service>` | Log của dịch vụ cụ thể |
| `journalctl -f` | Theo dõi log real-time |
| `journalctl --list-boots` | Danh sách các lần boot |
| `journalctl -b -1` | Log của lần boot trước |
| `journalctl --disk-usage` | Dung lượng journal đang chiếm |
| `sudo journalctl --vacuum-size=100M` | Dọn journal còn 100MB |

> `journalctl --vacuum-size` cần `sudo`.

## Mạng (Network)

| Lệnh | Chức năng |
|---|---|
| `ip addr` | Xem địa chỉ IP |
| `ip link show` | Xem interfaces |
| `ip route` | Xem bảng định tuyến |
| `ping -c 3 <host>` | Kiểm tra kết nối |
| `ss -tuln` | Xem port đang listen |
| `ss -tup` | Xem kết nối đang hoạt động |
| `nmcli device status` | Xem trạng thái thiết bị mạng |
| `nmcli device wifi list` | Quét Wi-Fi |
| `nmcli device wifi connect <SSID> password <pass>` | Kết nối Wi-Fi |
| `nmtui` | Giao diện text NetworkManager |
| `rfkill list` | Xem trạng thái RF kill |
| `rfkill unblock wifi` | Bỏ chặn Wi-Fi |

## Find / Grep / Ripgrep

| Lệnh | Chức năng |
|---|---|
| `find <path> -name "*.ext"` | Tìm file theo tên |
| `find <path> -type f -size +100M` | Tìm file lớn hơn 100MB |
| `find <path> -mtime -7` | File thay đổi trong 7 ngày |
| `grep -r "pattern" <path>` | Tìm text trong thư mục |
| `grep -rn "pattern" <path>` | Tìm + hiển thị số dòng |
| `grep -rl "pattern" <path>` | Chỉ hiển thị tên file |
| `grep -i "pattern"` | Tìm không phân biệt hoa thường |
| `rg "pattern"` | Tìm nhanh với ripgrep (nếu đã cài) |

## Rsync

| Lệnh | Chức năng |
|---|---|
| `rsync -avh <src>/ <dst>/` | Đồng bộ thư mục |
| `rsync -avh --delete <src>/ <dst>/` | Đồng bộ + xoá file thừa ở đích |
| `rsync -avh --exclude '.cache' <src>/ <dst>/` | Đồng bộ loại trừ thư mục |

## BTRFS

| Lệnh | Chức năng |
|---|---|
| `sudo btrfs subvolume list /` | Liệt kê subvolume |
| `sudo btrfs subvolume create <path>` | Tạo subvolume |
| `sudo btrfs subvolume delete <path>` | Xoá subvolume |
| `sudo btrfs subvolume snapshot -r <src> <dst>` | Tạo snapshot read-only |
| `sudo btrfs filesystem usage /` | Xem dung lượng chi tiết |
| `sudo btrfs filesystem show` | Xem thiết bị BTRFS |
| `sudo btrfs device stats /` | Xem thống kê lỗi thiết bị |
| `sudo btrfs scrub start /` | Quét kiểm tra lỗi filesystem |

## GRUB

| Lệnh | Chức năng |
|---|---|
| `sudo grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB` | Cài GRUB |
| `sudo grub-mkconfig -o /boot/grub/grub.cfg` | Sinh lại cấu hình GRUB |
| `efibootmgr -v` | Xem boot entries UEFI |
| `efibootmgr -c -d /dev/nvme0n1 -p 1 -L GRUB -l \\EFI\\GRUB\\grubx64.efi` | Thêm boot entry |

## NVIDIA (RTX 4050 — `nvidia-open`)

| Lệnh | Chức năng |
|---|---|
| `nvidia-smi` | Xem trạng thái GPU |
| `nvidia-smi -q` | Thông tin chi tiết GPU |
| `glxinfo \| grep renderer` | Xem OpenGL renderer đang dùng |
| `prime-run <lệnh>` | Chạy lệnh trên GPU NVIDIA (nếu có prime) |
| `cat /proc/driver/nvidia/version` | Phiên bản driver |

## PipeWire (Âm thanh)

| Lệnh | Chức năng |
|---|---|
| `pactl info` | Thông tin audio server |
| `pactl list sinks short` | Liệt kê output devices |
| `pactl list sources short` | Liệt kê input devices |
| `pactl set-default-sink <tên>` | Đặt output mặc định |
| `pamixer -i 5` | Tăng volume 5% |
| `pamixer -d 5` | Giảm volume 5% |
| `pamixer -t` | Tắt/bật âm thanh (mute toggle) |
| `pavucontrol` | Giao diện GUI mixer |

## Bspwm (bspc)

| Lệnh | Chức năng |
|---|---|
| `bspc node -c` | Đóng cửa sổ |
| `bspc node -k` | Kill cửa sổ |
| `bspc node -f west\|south\|north\|east` | Focus theo hướng |
| `bspc node -s west\|south\|north\|east` | Swap vị trí |
| `bspc node -d <số>` | Gửi cửa sổ sang workspace |
| `bspc desktop -f <số>` | Chuyển workspace |
| `bspc node -t tiled\|floating\|fullscreen` | Đổi chế độ hiển thị |
| `bspc node -g sticky\|private\|locked` | Gắn flag cho cửa sổ |
| `bspc node -m next` | Di chuyển sang màn hình khác |
| `bspc monitor -f next` | Focus màn hình khác |
| `bspc desktop -l next` | Đổi layout |
| `bspc wm -r` | Reload bspwm |
| `bspc query -D` | Liệt kê workspaces |
| `bspc query -N` | Liệt kê nodes |
| `bspc query -M` | Liệt kê monitors |

## Sxhkd

| Lệnh | Chức năng |
|---|---|
| `pkill -USR1 -x sxhkd` | Reload cấu hình sxhkd |
| `pgrep -x sxhkd` | Kiểm tra sxhkd đang chạy |
| `sxhkd -t 5` | Chạy foreground 5 giây để debug |

## Thông tin hệ thống

| Lệnh | Chức năng |
|---|---|
| `uname -a` | Kernel info đầy đủ |
| `uname -r` | Kernel version |
| `lscpu` | Thông tin CPU |
| `lsblk` | Thông tin ổ đĩa |
| `lspci -k` | PCI devices + kernel modules |
| `lsusb` | USB devices |
| `free -h` | Dung lượng RAM |
| `df -h` | Dung lượng ổ đĩa |
| `du -sh ~/*` | Dung lượng từng thư mục |
| `uptime` | Thời gian hệ thống đã chạy |
| `dmesg \| tail -30` | Kernel log (30 dòng cuối) |
| `timedatectl status` | Thời gian và múi giờ |
| `hostnamectl` | Thông tin hệ thống |
