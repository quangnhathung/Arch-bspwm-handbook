# Essential Tools on Arch Linux

Tài liệu tham khảo các công cụ thiết yếu cho lập trình viên dùng Arch Linux + bspwm. Tập trung vào năng suất, kiểm soát hệ thống và phát triển phần mềm.

---

## 1. System Core Tools

- **pacman** — Trình quản lý gói mặc định. `sudo pacman -Syu` để đồng bộ và nâng cấp.
- **yay** — AUR helper. `yay -S gói` để cài từ AUR. Cài đặt: `sudo pacman -S --needed git base-devel && git clone https://aur.archlinux.org/yay.git && cd yay && makepkg -si`.
- **base-devel** — Meta-package gồm gcc, make, autoconf,... `sudo pacman -S base-devel`.
- **git** — Quản lý phiên bản. `sudo pacman -S git`.
- **curl** — Công cụ truyền tải HTTP. `sudo pacman -S curl`.
- **wget** — Tải file với hỗ trợ đệ quy. `sudo pacman -S wget`.
- **unzip** — Giải nén ZIP. `sudo pacman -S unzip`.
- **tar** — Công cụ nén (có sẵn trên Arch).

---

## 2. Time & System Management

- **timedatectl** — Quản lý thời gian và ngày giờ qua systemd.
  ```bash
  timedatectl set-ntp true             # bật NTP
  timedatectl set-timezone Asia/Ho_Chi_Minh  # ví dụ
  timedatectl status
  ```

---

## 3. Text Editors

**CLI**

- **neovim** — Fork của Vim với Lua API, hỗ trợ plugin mạnh mẽ. `sudo pacman -S neovim`.
- **vim** — Trình soạn thảo modal cổ điển. `sudo pacman -S vim`.

**GUI**

- **Visual Studio Code** — Editor đầy đủ tính năng, terminal tích hợp, debugger. `sudo pacman -S code` hoặc `yay -S visual-studio-code-bin`.
- **Sublime Text** — Editor GUI nhanh, nhẹ. `yay -S sublime-text-4`.

---

## 4. Battery & Power Management

- **tlp** — Quản lý nguồn cho laptop. Tự động cấu hình ngưỡng sạc (đặt stop tại 80% qua `/etc/tlp.d/`). Cài: `sudo pacman -S tlp && sudo systemctl enable --now tlp`.
- **upower** — Báo cáo pin và thiết bị. `upower -i /org/freedesktop/UPower/devices/battery_BAT0`.
- **lm_sensors** — Giám sát phần cứng (nhiệt độ CPU, tốc độ quạt). `sudo pacman -S lm_sensors && sudo sensors-detect && sensors`.

---

## 5. System Monitoring

- **htop** — Trình xem tiến trình tương tác, hỗ trợ cây và chuột. `sudo pacman -S htop`.
- **btop** — Monitor tài nguyên hiện đại, hỗ trợ GPU và themes. `sudo pacman -S btop`.

---

## 6. Window Manager Stack (bspwm)

- **bspwm** — Tiling window manager điều khiển qua IPC. `sudo pacman -S bspwm`.
- **sxhkd** — Hotkey daemon đơn giản, kết nối keybinding với lệnh bspwm. `sudo pacman -S sxhkd`.
- **rofi** — Application launcher, thay thế dmenu. Hỗ trợ drun, cửa sổ, script. `sudo pacman -S rofi`.
- **polybar** — Status bar tùy biến cao với module workspace, network, pin, đồng hồ. `sudo pacman -S polybar`.

---

## 7. Screenshot & Screen Tools

- **flameshot** — Chụp màn hình GUI có chú thích. Hoạt động trên X11 và Wayland (qua XWayland). `sudo pacman -S flameshot`.
- **grim** — Chụp màn hình gốc Wayland (output/region). `sudo pacman -S grim`.
- **slurp** — Chọn vùng màn hình trên Wayland, dùng kèm grim. `sudo pacman -S slurp`.

---

## 8. Image Tools

- **feh** -- Xem ảnh và đặt wallpaper nhẹ. `feh --bg-fill ~/wallpaper.jpg`. `sudo pacman -S feh`.
- **imagemagick** — Bộ công cụ xử lý ảnh CLI. `sudo pacman -S imagemagick`.

---

## 9. Media Tools

- **mpv** -- Player video/audio tối giản, GPU acceleration, hỗ trợ script. `sudo pacman -S mpv`.
- **vlc** -- Player đa nền tảng, hỗ trợ codec sẵn. `sudo pacman -S vlc`.
- **yt-dlp** -- Tải video từ YouTube và các site khác (fork của youtube-dl). `sudo pacman -S yt-dlp`.
- **ffmpeg** -- Chuyển đổi, streaming, ghi âm/video đa năng. `sudo pacman -S ffmpeg`.

---

## 10. Network Tools

- **networkmanager** -- Quản lý mạng. `sudo pacman -S networkmanager && sudo systemctl enable --now NetworkManager`.
- **nmcli** -- CLI frontend cho NetworkManager. Hỗ trợ VPN, Wi-Fi, ethernet.
- **ping** -- Kiểm tra ICMP (thuộc `iputils`). `sudo pacman -S iputils`.
- **traceroute** -- Chẩn đoán đường đi mạng. `sudo pacman -S traceroute`.
- **dig / nslookup** -- Truy vấn DNS (thuộc `bind`). `sudo pacman -S bind`.

---

## 11. Development Tools

- **gcc** -- Trình biên dịch GNU C/C++. `sudo pacman -S gcc`.
- **clang** -- Trình biên dịch LLVM C/C++/ObjC. `sudo pacman -S clang`.
- **make** -- Build automation. `sudo pacman -S make`.
- **docker** -- Container runtime. `sudo pacman -S docker && sudo systemctl enable --now docker`.
- **nodejs** -- Runtime JavaScript/TypeScript. `sudo pacman -S nodejs npm`.
- **python** -- Python 3. `sudo pacman -S python python-pip`.
- **go** -- Ngôn ngữ Go. `sudo pacman -S go`.

---

## 12. File Managers

- **ranger** -- File manager trong terminal, phím tắt kiểu Vim, xem trước file. `sudo pacman -S ranger`.
- **thunar** -- File manager GTK nhẹ (XFCE). `sudo pacman -S thunar`.
- **nautilus** -- File manager GNOME đầy đủ tính năng. `sudo pacman -S nautilus`.

---

## 13. Clipboard & Productivity Tools

- **xclip** -- Clipboard CLI cho X11. `echo "text" | xclip -selection c`. `sudo pacman -S xclip`.
- **wl-clipboard** -- Clipboard CLI cho Wayland. `sudo pacman -S wl-clipboard`.
- **fzf** -- Fuzzy finder cho file, lịch sử lệnh, process,... `sudo pacman -S fzf`.
- **ripgrep (rg)** -- Tìm kiếm text đệ quy tốc độ cao. `sudo pacman -S ripgrep`.
- **bat** -- Thay thế `cat` với syntax highlight và git integration. `sudo pacman -S bat`.
- **eza** -- Thay thế `ls` hiện đại, có icon, màu sắc, tree view. `sudo pacman -S eza`.

---

## 14. Security Tools

- **openssh** -- SSH client/server cho truy cập từ xa bảo mật. `sudo pacman -S openssh && sudo systemctl enable --now sshd`.
- **ufw** -- Uncomplicated Firewall (frontend iptables/nftables). `sudo pacman -S ufw && sudo ufw enable`.
- **firewalld** -- Dynamic firewall với zone-based management. `sudo pacman -S firewalld && sudo systemctl enable --now firewalld`.
- **gnupg** -- Mã hóa và ký số (GNU Privacy Guard). `sudo pacman -S gnupg`.

---

## 15. Recommended Minimal Setup Command

Lệnh cài đặt tối thiểu cho môi trường dev + bspwm:

```bash
sudo pacman -S --needed \
  base-devel git curl wget unzip \
  neovim \
  bspwm sxhkd rofi polybar \
  flameshot \
  feh \
  mpv vlc ffmpeg yt-dlp \
  networkmanager htop btop \
  gcc clang make nodejs npm python python-pip go \
  ranger thunar \
  xclip fzf ripgrep bat eza \
  openssh ufw gnupg \
  tlp upower lm_sensors
```

Bật services:

```bash
sudo systemctl enable --now NetworkManager tlp sshd ufw
```

Cài thêm từ AUR (VS Code, Sublime Text):

```bash
yay -S visual-studio-code-bin
```
