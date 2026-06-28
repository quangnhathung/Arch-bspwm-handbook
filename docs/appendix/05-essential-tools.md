# Công Cụ Thiết Yếu — Arch Linux + bspwm (Lenovo LOQ 15IAX9)

**Ngày: 25/06/2026 — Kernel 7.x**

Tổng hợp các công cụ cần thiết cho lập trình viên dùng Arch Linux với bspwm. Mỗi mục gồm: mục đích, lệnh cài đặt, và ví dụ sử dụng nhanh.

---

## 1. Hệ thống (System Core)

### pacman
Trình quản lý gói mặc định của Arch.
```bash
sudo pacman -Syu                       # Cập nhật hệ thống
sudo pacman -S <gói>                   # Cài gói
```

### yay / yay-bin
AUR helper. `yay` biên dịch từ nguồn (chậm); `yay-bin` là pre-compiled (nhanh).
```bash
git clone https://aur.archlinux.org/yay-bin.git
cd yay-bin && makepkg -si
yay -S <gói>                           # Cài từ AUR
```

### base-devel
Bộ công cụ biên dịch (gcc, make, autoconf, ...).
```bash
sudo pacman -S base-devel
```

### git
Quản lý phiên bản.
```bash
sudo pacman -S git
git clone <url>
```

### curl / wget
Truyền tải HTTP / tải file.
```bash
sudo pacman -S curl wget
curl -O https://example.com/file
wget https://example.com/file
```

### pacman-contrib
Cung cấp `paccache` (dọn cache pacman) và các tiện ích khác.
```bash
sudo pacman -S pacman-contrib
paccache -rk1                          # Giữ 1 bản cache
```

---

## 2. Soạn thảo văn bản (Text Editors)

### neovim
Fork của Vim, hỗ trợ Lua API, plugin phong phú.
```bash
sudo pacman -S neovim
nvim file.py
```

### vim
Trình soạn thảo modal cổ điển.
```bash
sudo pacman -S vim
vim file.txt
```

### Visual Studio Code
Editor GUI đầy đủ tính năng (cài từ AUR).
```bash
yay -S visual-studio-code-bin
code .
```

---

## 3. Quản lý nguồn (Power Management)

### tlp
Tự động tối ưu năng lượng cho laptop. Có thể cấu hình ngưỡng sạc pin.
```bash
sudo pacman -S tlp
sudo systemctl enable --now tlp
tlp-stat                                  # Xem trạng thái
```

### power-profiles-daemon
Quản lý profile nguồn (power-saver / balanced / performance).
```bash
sudo pacman -S power-profiles-daemon
powerprofilesctl set performance
```

### upower
Xem thông tin pin và thiết bị.
```bash
sudo pacman -S upower
upower -i /org/freedesktop/UPower/devices/battery_BAT0
```

### lm_sensors
Đọc nhiệt độ CPU, tốc độ quạt, điện áp.
```bash
sudo pacman -S lm_sensors
sudo sensors-detect                     # Cấu hình (chọn YES cho tất cả)
sensors                                 # Xem nhiệt độ
```

---

## 4. Giám sát hệ thống (Monitoring)

### htop
Trình xem tiến trình tương tác, hỗ trợ chuột.
```bash
sudo pacman -S htop
htop
```

### btop
Monitor hiện đại, hỗ trợ GPU, theme, chuột.
```bash
sudo pacman -S btop
btop
```

### nvtop
Giám sát GPU (NVIDIA / Intel).
```bash
sudo pacman -S nvtop
nvtop
```

---

## 5. Window Manager Stack

### bspwm
Tiling window manager điều khiển qua IPC (bspc).
```bash
sudo pacman -S bspwm
bspc wm -r                               # Reload
```

### sxhkd
Hotkey daemon, kết nối phím tắt với lệnh.
```bash
sudo pacman -S sxhkd
pkill -USR1 -x sxhkd                     # Reload config
```

### rofi
Application launcher (thay thế dmenu).
```bash
sudo pacman -S rofi
rofi -show drun                          # Launcher
```

### polybar
Status bar với module workspace, network, pin, đồng hồ.
```bash
sudo pacman -S polybar
polybar example                          # Chạy với config mẫu
```

### picom (fork)
Compositor cho X11 (tránh screen tearing). Dùng fork `picom` (không dùng `compton`).
```bash
sudo pacman -S picom
picom --config ~/.config/picom/picom.conf
```

---

## 6. Chụp màn hình (Screenshot)

### flameshot
GUI chụp màn hình có chú thích (mũi tên, text, highlight).
```bash
sudo pacman -S flameshot
flameshot gui                            # Chụp có GUI
```

### maim
Chụp màn hình CLI nhẹ (có thể dùng kèm slop để chọn vùng).
```bash
sudo pacman -S maim slop
maim -s screenshot.png                   # Chọn vùng chụp
```

---

## 7. Xem ảnh (Image)

### feh
Xem ảnh, đặt wallpaper.
```bash
sudo pacman -S feh
feh --bg-fill ~/wallpaper.jpg            # Đặt wallpaper
feh ~/anh.png                            # Xem ảnh
```

### imagemagick
Xử lý ảnh CLI (resize, convert, crop).
```bash
sudo pacman -S imagemagick
convert input.jpg -resize 50% output.jpg
identify image.png                       # Xem thông tin ảnh
```

---

## 8. Đa phương tiện (Media)

### mpv
Player video/audio tối giản, GPU acceleration.
```bash
sudo pacman -S mpv
mpv video.mp4
mpv --shuffle ~/music/                   # Phát nhạc ngẫu nhiên
```

### vlc
Player đa nền tảng, codec sẵn.
```bash
sudo pacman -S vlc
vlc video.mp4
```

### yt-dlp
Tải video từ YouTube và hàng trăm site khác.
```bash
sudo pacman -S yt-dlp
yt-dlp https://youtube.com/watch?v=...
yt-dlp -f bestaudio --extract-audio URL  # Tải audio
```

### ffmpeg
Chuyển đổi, ghi âm, stream, xử lý media.
```bash
sudo pacman -S ffmpeg
ffmpeg -i input.mp4 output.avi
ffmpeg -i video.mp4 -ss 00:01:00 -t 10 clip.mp4  # Cắt clip
```

---

## 9. Mạng (Network)

### networkmanager + nmcli
Quản lý mạng (Wi-Fi, Ethernet, VPN).
```bash
sudo pacman -S networkmanager
sudo systemctl enable --now NetworkManager
nmcli device wifi connect "SSID" password "pass"
```

### iwd
Wi-Fi daemon nhẹ hơn (có thể thay wpa_supplicant).
```bash
sudo pacman -S iwd
iwctl                                       # Giao diện CLI
```

---

## 10. Quản lý file (File Managers)

### pcmanfm
File manager GUI nhẹ (GTK).
```bash
sudo pacman -S pcmanfm
pcmanfm
```

### ranger
File manager trong terminal, phím tắt kiểu Vim, xem trước file.
```bash
sudo pacman -S ranger
ranger
```

### lf
File manager terminal nhẹ hơn ranger (viết bằng Go).
```bash
sudo pacman -S lf
lf
```

---

## 11. Nén (Compression)

### tar
Nén / giải nén (có sẵn trên Arch).
```bash
tar -czf archive.tar.gz /path            # Nén
tar -xzf archive.tar.gz                  # Giải nén
```

### unzip
Giải nén ZIP.
```bash
sudo pacman -S unzip
unzip file.zip
```

### p7zip
Hỗ trợ 7z, các định dạng nén mạnh.
```bash
sudo pacman -S p7zip
7z x file.7z                             # Giải nén
7z a archive.7z /path                    # Nén
```

---

## 12. Bảo mật (Security)

### openssh
SSH client/server cho truy cập từ xa.
```bash
sudo pacman -S openssh
sudo systemctl enable --now sshd
ssh user@host
```

### gnupg
Mã hoá và ký số.
```bash
sudo pacman -S gnupg
gpg --gen-key                            # Tạo key
gpg -c file.txt                          # Mã hoá đối xứng
```

### ufw
Uncomplicated Firewall — frontend iptables dễ dùng.
```bash
sudo pacman -S ufw
sudo ufw enable
sudo ufw status verbose
sudo ufw allow 22/tcp                    # Cho phép SSH
```

---

## 13. Phát triển (Development)

### python
Python 3 + pip.
```bash
sudo pacman -S python python-pip
python -m venv venv
```

### nodejs
JavaScript runtime + npm.
```bash
sudo pacman -S nodejs npm
node -e "console.log('hello')"
```

### docker
Container runtime.
```bash
sudo pacman -S docker
sudo systemctl enable --now docker
sudo docker run hello-world
# Thêm user vào group docker:
sudo usermod -aG docker $USER
# Logout rồi login lại.
```

### go
Ngôn ngữ Go.
```bash
sudo pacman -S go
go version
```

---

## 14. Custom Scripts (tự động hóa)

Các script shell tự viết, đặt trong `~/.local/bin/` và `~/.config/polybar/scripts/`.

### wallpaper.sh — Quản lý hình nền

```bash
~/.local/bin/wallpapers/            # Đặt ảnh từ thư mục ~/images/Wallpapers/
~/.local/bin/wallpaper.sh prev      # Ảnh trước
~/.local/bin/wallpaper.sh next      # Ảnh tiếp theo
~/.local/bin/wallpaper.sh random    # Ảnh ngẫu nhiên
```

Dùng `feh --bg-fill` để đặt wallpaper. Ghi nhớ ảnh hiện tại qua `~/.cache/wallpaper_index`.

### brightness.sh — Điều chỉnh độ sáng

Thiết bị backlight đặc thù trên NVIDIA laptop: `nvidia_wmi_ec_backlight`.

```bash
~/.local/bin/brightness.sh up      # +5%
~/.local/bin/brightness.sh down    # -5%
```

Kèm Dunst notification với progress bar.

### volume.sh — Điều chỉnh âm lượng

Dùng `wpctl` (PipeWire) thay vì `pamixer`. Gửi notification qua Dunst.

```bash
~/.local/bin/volume.sh up          # +5%
~/.local/bin/volume.sh down        # -5%
~/.local/bin/volume.sh mute        # Toggle mute
```

Icon thay đổi theo mức: 󰕿 (nhỏ) → 󰖀 (vừa) → 󰕾 (to).

### polybar/scripts/gpu.sh — GPU monitoring

Đọc Intel iGPU frequency (`/sys/class/drm/card*/gt_cur_freq_mhz`) + NVIDIA utilization (`nvidia-smi`).

### polybar/scripts/mic.sh — Mic recording detection

Kiểm tra `pactl list source-outputs` — hiện icon đỏ khi có app đang dùng mic.

---

## 15. Terminal Multiplexer — tmux

### tmux
Terminal multiplexer: quản lý nhiều phiên terminal trong một cửa sổ, giữ phiên làm việc sống ngay cả khi đóng terminal.

```bash
sudo pacman -S tmux
tmux new -s work              # Tạo session mới tên "work"
tmux attach -t work           # Đính kèm lại session "work"
tmux ls                       # Liệt kê session
```

### Cấu hình tmux hiện tại

File: `~/.tmux.conf`

| Tính năng | Giá trị |
|-----------|---------|
| Prefix key | `Ctrl + a` (thay vì `Ctrl + b` mặc định) |
| Base index | Window/Pane bắt đầu từ 1 |
| Mouse | Bật — scroll, click chọn pane, kéo thả resize |
| History limit | 10,000 dòng |
| Reload config | `Prefix + r` |

### Plugin đã cài (TPM — Tmux Plugin Manager)

| Plugin | Chức năng |
|--------|-----------|
| `tmux-plugins/tpm` | Trình quản lý plugin |
| `tmux-plugins/tmux-sensible` | Thiết lập mặc định tối ưu |
| `tmux-plugins/tmux-resurrect` | Lưu/khôi phục trạng thái session (cửa sổ, pane, đường dẫn) |
| `tmux-plugins/tmux-continuum` | Tự động lưu mỗi 15 phút + tự khôi phục khi mở tmux |
| `catppuccin/tmux` | Theme Catppuccin Mocha |

### Cài plugin

```bash
# Trong tmux: Prefix + I (chữ I hoa) → cài tất cả plugin
# Cập nhật plugin: Prefix + U
# Xoá plugin thừa: Prefix + M-u
```

### Resurrect — lưu và khôi phục

```bash
# Lưu thủ công:      Prefix + Ctrl + s
# Khôi phục:         Prefix + Ctrl + r
# Tự động lưu:       continuum cứ 15 phút lưu 1 lần
# Tự động khôi phục: continuum-restore = on → tự restore khi mở tmux
```

### Phím tắt tmux cơ bản

| Phím | Chức năng |
|------|-----------|
| `Prefix + c` | Tạo window mới |
| `Prefix + ,` | Đổi tên window |
| `Prefix + &` | Đóng window |
| `Prefix + w` | Chọn window bằng danh sách |
| `Prefix + n/p` | Window tiếp theo / trước |
| `Prefix + số` | Chuyển đến window số đó |
| `Prefix + %` | Chia pane dọc |
| `Prefix + "` | Chia pane ngang |
| `Prefix + o` | Di chuyển qua các pane |
| `Prefix + x` | Đóng pane |
| `Prefix + z` | Zoom pane (full màn hình) |
| `Prefix + {` | Swap pane với pane trên |
| `Prefix + }` | Swap pane với pane dưới |
| `Prefix + [` | Vào chế độ copy (cuộn log, dùng `vi` keys) |
| `Prefix + d` | Detach session |
| `Prefix + :` | Command mode (gõ lệnh tmux) |
| `Prefix + ?` | Danh sách tất cả phím tắt |

### ⚠️ Lưu ý xung đột phím tắt

- `Prefix` là `Ctrl + a`: không xung đột với bspwm/sxhkd (sxhkd không dùng `Ctrl + a`).
- `Prefix + r` reload tmux config — không xung đột với sxhkd (`Super + Escape` reload sxhkd).
- `Prefix + Ctrl + s` (tmux-resurrect save) — không xung đột.
- Scroll bằng chuột được bật (không cần dùng phím).

---

## Tổng hợp — Cài đặt nhanh

```bash
# === Cài tất cả công cụ thiết yếu ===
sudo pacman -S --needed \
  base-devel git curl wget pacman-contrib \
  neovim \
  bspwm sxhkd rofi polybar picom \
  flameshot maim slop \
  feh imagemagick \
  mpv vlc yt-dlp ffmpeg \
  networkmanager iwd \
  pcmanfm ranger lf \
  tmux \
  unzip p7zip \
  openssh gnupg ufw \
  python python-pip nodejs npm docker go \
  htop btop nvtop \
  tlp power-profiles-daemon upower lm_sensors

# === Bật services ===
sudo systemctl enable --now NetworkManager tlp sshd ufw

# === Cài từ AUR ===
yay -S visual-studio-code-bin
```
