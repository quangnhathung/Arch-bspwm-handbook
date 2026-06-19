# Command Cheatsheet

## Mục tiêu

Tổng hợp các lệnh hữu ích cho quản trị và sử dụng Arch Linux + bspwm.

## Pacman

| Lệnh | Chức năng |
|---|---|
| `pacman -Syu` | Cập nhật toàn bộ hệ thống |
| `pacman -S gói` | Cài gói |
| `pacman -Rs gói` | Xóa gói + dependencies không dùng |
| `pacman -Rns gói` | Xóa gói + config + dependencies |
| `pacman -Ss từ` | Tìm gói trong database |
| `pacman -Qs từ` | Tìm gói đã cài |
| `pacman -Q` | Liệt kê tất cả gói đã cài |
| `pacman -Qi gói` | Thông tin gói đã cài |
| `pacman -Si gói` | Thông tin gói trong repo |
| `pacman -Ql gói` | File của gói đã cài |
| `pacman -Qo file` | Gói nào sở hữu file |
| `pacman -F file` | Tìm gói chứa file |
| `pacman -Qtdq` | Liệt kê orphan packages |
| `pacman -U file.pkg.tar.zst` | Cài từ file |
| `paccache -r` | Dọn cache (giữ 3 bản) |
| `paccache -rk 1` | Dọn cache (giữ 1 bản) |

## Yay

| Lệnh | Chức năng |
|---|---|
| `yay -Syu` | Cập nhật tất cả (AUR + chính thức) |
| `yay -S gói` | Cài gói (AUR hoặc chính thức) |
| `yay -Sa gói` | Cài gói từ AUR |
| `yay -Ss từ` | Tìm gói (AUR + chính thức) |
| `yay -Ssa từ` | Tìm gói (chỉ AUR) |
| `yay -Sc` | Dọn cache yay |

## Systemctl

| Lệnh | Chức năng |
|---|---|
| `systemctl start service` | Start dịch vụ |
| `systemctl stop service` | Stop dịch vụ |
| `systemctl restart service` | Restart dịch vụ |
| `systemctl enable service` | Bật tự động khởi động |
| `systemctl disable service` | Tắt tự động khởi động |
| `systemctl status service` | Xem trạng thái |
| `systemctl --failed` | Liệt kê dịch vụ lỗi |
| `systemctl list-units` | Liệt kê tất cả unit |
| `systemctl --user ...` | User service (thay system) |
| `systemctl enable --now ...` | Enable + start ngay |

## Journalctl

| Lệnh | Chức năng |
|---|---|
| `journalctl -p 3 -xb` | Log lỗi (priority 3+) |
| `journalctl -u service` | Log của service cụ thể |
| `journalctl -f` | Follow log (real-time) |
| `journalctl --list-boots` | Danh sách boot sessions |
| `journalctl -b -1` | Log của boot trước |
| `journalctl --disk-usage` | Dung lượng journal |
| `journalctl --vacuum-size=100M` | Dọn journal giữ 100MB |

## IP / Network

| Lệnh | Chức năng |
|---|---|
| `ip addr` | Xem IP address |
| `ip link show` | Xem interfaces |
| `ip route` | Xem routing table |
| `ping -c 3 host` | Kiểm tra kết nối |
| `ss -tuln` | Xem port đang listen |
| `ss -tup` | Xem kết nối đang hoạt động |
| `nmcli device status` | Xem trạng thái thiết bị mạng |
| `nmcli device wifi list` | Quét Wi-Fi |
| `nmcli device wifi connect SSID password P` | Kết nối Wi-Fi |
| `nmtui` | Giao diện text NetworkManager |
| `iwctl` | iwd CLI (nếu dùng iwd) |
| `rfkill list` | Xem RF kill status |
| `rfkill unblock wifi` | Unblock Wi-Fi |

## Find / Grep

| Lệnh | Chức năng |
|---|---|
| `find /path -name "*.ext"` | Tìm file theo tên |
| `find /path -type f -size +100M` | Tìm file lớn |
| `find /path -mtime -7` | File thay đổi trong 7 ngày |
| `grep -r "pattern" /path` | Tìm text trong file |
| `grep -rn "pattern" /path` | Tìm + số dòng |
| `grep -rl "pattern" /path` | Chỉ hiện tên file |
| `grep -i "pattern"` | Case-insensitive |

## Rsync

| Lệnh | Chức năng |
|---|---|
| `rsync -avh src/ dst/` | Sync thư mục |
| `rsync -avh --delete src/ dst/` | Sync + xóa file thừa |
| `rsync -avh --exclude '.cache' src/ dst/` | Sync loại trừ |

## BTRFS

| Lệnh | Chức năng |
|---|---|
| `btrfs subvolume list /` | Liệt kê subvolume |
| `btrfs subvolume create /path` | Tạo subvolume |
| `btrfs subvolume delete /path` | Xóa subvolume |
| `btrfs subvolume snapshot -r src dst` | Tạo snapshot read-only |
| `btrfs filesystem usage /` | Xem dung lượng |
| `btrfs filesystem show` | Xem device |
| `btrfs device stats /` | Xem thống kê lỗi |
| `btrfs scrub start /` | Kiểm tra lỗi filesystem |

## GRUB

| Lệnh | Chức năng |
|---|---|
| `grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB` | Cài GRUB |
| `grub-mkconfig -o /boot/grub/grub.cfg` | Sinh config |
| `efibootmgr -v` | Xem boot entries |
| `efibootmgr -c -d /dev/nvme0n1 -p 1 -L GRUB -l \\EFI\\GRUB\\grubx64.efi` | Thêm entry |

## NVIDIA

| Lệnh | Chức năng |
|---|---|
| `nvidia-smi` | Xem GPU status |
| `nvidia-smi -q` | Thông tin chi tiết |
| `glxinfo \| grep renderer` | Xem OpenGL renderer |
| `prime-run lệnh` | Chạy lệnh trên NVIDIA |

## PipeWire

| Lệnh | Chức năng |
|---|---|
| `pactl info` | Xem thông tin audio server |
| `pactl list sinks short` | Liệt kê output devices |
| `pactl list sources short` | Liệt kê input devices |
| `pactl set-default-sink name` | Set output mặc định |
| `pamixer -i 5` | Tăng volume |
| `pamixer -d 5` | Giảm volume |
| `pamixer -t` | Mute toggle |
| `pavucontrol` | GUI mixer |

## Bspwm (CLI)

| Lệnh | Chức năng |
|---|---|
| `bspc node -c` | Đóng cửa sổ |
| `bspc node -k` | Kill cửa sổ |
| `bspc node -f west/south/north/east` | Focus |
| `bspc node -s west/south/north/east` | Swap |
| `bspc node -d 1-9` | Gửi đến workspace |
| `bspc desktop -f 1-9` | Chuyển workspace |
| `bspc node -t tiled/floating/fullscreen` | Set state |
| `bspc node -g sticky/private/locked` | Set flag |
| `bspc node -m next` | Di chuyển sang monitor khác |
| `bspc monitor -f next` | Focus monitor khác |
| `bspc desktop -l next` | Đổi layout |
| `bspc wm -r` | Reload bspwm |
| `bspc query -D` | Query workspaces |
| `bspc query -N` | Query nodes |
| `bspc query -M` | Query monitors |

## Sxhkd

| Lệnh | Chức năng |
|---|---|
| `pkill -USR1 -x sxhkd` | Reload config |
| `pgrep -x sxhkd` | Kiểm tra sxhkd đang chạy |
| `sxhkd -t 5` | Chạy foreground 5s để debug |

## System Info

| Lệnh | Chức năng |
|---|---|
| `uname -a` | Kernel info |
| `uname -r` | Kernel version |
| `lscpu` | CPU info |
| `lsblk` | Disk info |
| `lspci` | PCI devices |
| `lsusb` | USB devices |
| `free -h` | RAM usage |
| `df -h` | Disk usage |
| `du -sh /*` | Dung lượng thư mục |
| `uptime` | Thời gian hoạt động |
| `dmesg \| tail -20` | Kernel log |
| `timedatectl status` | Thời gian |
| `hostnamectl` | System info |
