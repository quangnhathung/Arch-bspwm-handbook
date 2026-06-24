# Hệ thống Bàn phím & Keybinding trong Linux (Lenovo LOQ)

Ngày: 25/06/2026

Tài liệu giải thích cách thức hoạt động của bàn phím trên Linux, hành vi phím Fn trên laptop Lenovo LOQ, và cách xây dựng hệ thống keybinding hoàn chỉnh trong bspwm.

---

## 1. Tổng quan về hệ thống xử lý phím trong Linux

Mỗi lần nhấn phím trải qua các tầng sau:

```
Phím vật lý → BIOS → Kernel (evdev) → X11 → sxhkd → command
```

| Tầng | Vai trò |
|------|---------|
| **BIOS/EC** | Nhận tín hiệu phím vật lý, quyết định gửi keycode nào xuống OS. Có thể biến đổi tín hiệu (vd: Fn + F1 → volume down) |
| **Kernel (evdev)** | Nhận scancode từ bàn phím, map thành keycode Linux. Tạo event qua `/dev/input/event*` |
| **X11 (X server)** | Nhận event từ kernel, map keycode thành keysym (tên symbolic) dùng XKB. Phát ra XF86 keysym cho phím đặc biệt |
| **sxhkd** | Lắng nghe X11 event, so khớp với config, thực thi lệnh |
| **Command** | Lệnh shell được thực thi (bspc, pamixer, brightnessctl, ...) |

**Key point:** Phím vật lý và hành động phần mềm là hai việc hoàn toàn khác nhau. BIOS quyết định gửi tín hiệu gì; Linux quyết định xử lý tín hiệu đó ra sao.

---

## 2. Hành vi phím Fn trên Lenovo LOQ

### 2.1. Chế độ Hotkey trong BIOS

Lenovo LOQ có tùy chọn trong BIOS (F2 khi boot → **Configuration** → **Hotkey Mode**):

| Chế độ | Hành vi F1-F12 mặc định | Cách dùng Fn |
|--------|------------------------|--------------|
| **Enabled** (mặc định) | Multimedia action (volume, brightness, ...) | Fn + F1-F12 mới ra F1-F12 |
| **Disabled** | F1-F12 tiêu chuẩn | Fn + F1-F12 mới ra multimedia action |

Tác dụng duy nhất của BIOS: thay đổi tín hiệu gửi xuống OS. BIOS **không** gán chức năng. Nếu Hotkey Mode = Enabled, khi nhấn F2, BIOS gửi `XF86MonBrightnessDown` thay vì `F2`.

### 2.2. Phím Fn + Q (Performance Mode)

Phím Fn + Q trên Lenovo LOQ gửi tín hiệu ACPI WMI event, **không** phải keycode X11 thông thường. Do đó `sxhkd` không thể bắt được phím này. Cần `acpi_listen` + script riêng + `power-profiles-daemon`.

### 2.3. Các phím đặc biệt trên LOQ

| Phím | Tín hiệu (Hotkey ON) | Keysym X11 |
|------|---------------------|------------|
| F1 | Mic mute | `XF86AudioMicMute` |
| F2 | Giảm độ sáng | `XF86MonBrightnessDown` |
| F3 | Tăng độ sáng | `XF86MonBrightnessUp` |
| F4 | Chuyển màn hình | `XF86Display` |
| F5 | Chụp màn hình | Print |
| F6 | Giảm âm lượng | `XF86AudioLowerVolume` |
| F7 | Tăng âm lượng | `XF86AudioRaiseVolume` |
| F8 | Tắt tiếng | `XF86AudioMute` |
| F9 | Play/Pause | `XF86AudioPlay` |
| F10 | Previous | `XF86AudioPrev` |
| F11 | Next | `XF86AudioNext` |
| F12 | Máy bay | `XF86WLAN` |

> Các tín hiệu trên chỉ đúng khi Hotkey Mode = Enabled. Khi tắt, các phím F1-F12 gửi keycode số thường, và Fn + Fn mới gửi tín hiệu trên.

---

## 3. Debug phím

### 3.1. xev — kiểm tra keysym X11

Gói: `xorg-xev`

```bash
sudo pacman -S xorg-xev
xev | grep keysym
```

Nhấn phím cần kiểm tra, output:

```
state 0x0, keycode 122 (keysym 0x1008ff13, XF86AudioLowerVolume), same_screen YES
```

Dùng `xev` khi cần biết keysym chính xác để đặt trong `sxhkdrc`.

### 3.2. evtest — debug tầng kernel

Gói: `evtest`

```bash
sudo pacman -S evtest
sudo evtest
```

Chọn thiết bị bàn phím, nhấn phím và xem raw scancode. Hữu ích khi phím hoạt động ở kernel level nhưng không lên X11 (ví dụ Fn+Q).

### 3.3. acpi_listen — ACPI event

```bash
sudo pacman -S acpid
acpi_listen
```

Dùng để bắt ACPI WMI event từ Fn+Q và các phím đặc biệt không gửi X11 keysym.

---

## 4. Kiến trúc sxhkd

sxhkd là hotkey daemon riêng biệt, **hoàn toàn độc lập** với bspwm:

```
bspwm (window manager)                     sxhkd (hotkey daemon)
       │                                          │
       │ IPC: bspc node -f west                    │ exec: bspc node -f west
       │                                          │
       └─────────────── bspwmrc ───────────────────┘
                            │
                 Khởi động cả hai trong .xinitrc
```

- **bspwm** quản lý cửa sổ, lắng nghe IPC từ `bspc`.
- **sxhkd** là hotkey daemon, lắng nghe X11 event, so khớp với `sxhkdrc`, thực thi lệnh.
- Hai process độc lập: bspwm không biết về keybinding; sxhkd không biết về window management.

### File cấu hình

| File | Đường dẫn | Vai trò |
|------|-----------|---------|
| sxhkdrc | `~/.config/sxhkd/sxhkdrc` | Định nghĩa tất cả keybinding |
| bspwmrc | `~/.config/bspwm/bspwmrc` | Cấu hình bspwm + khởi động chương trình |

### Cấu trúc sxhkdrc

```
# Comment
modifier + key
<TAB>command

# Ví dụ
super + Return
<TAB>alacritty

super + {h,j,k,l}
<TAB>bspc node -f {west,south,north,east}
```

> **IMPORTANT:** sxhkd yêu cầu TAB indentation. Dùng spaces sẽ không hoạt động.

Cú pháp mở rộng:
- `{a,b,c}` — batch, sinh nhiều binding từ một dòng
- `{1-9}` — range, tương tự
- `;` — chain (binding nối tiếp)
- `~` — grab release

### Nạp lại cấu hình

```bash
pkill -USR1 -x sxhkd
```

Hoặc binding trong sxhkdrc:

```
super + Escape
<TAB>pkill -USR1 -x sxhkd && bspc wm -r
```

---

## 5. Power Management & Xung đột

### 5.1. power-profiles-daemon — chuyển đổi hiệu năng

```bash
sudo pacman -S power-profiles-daemon
sudo systemctl enable --now power-profiles-daemon
```

| Profile | Use case |
|---------|----------|
| `power-saver` | Tiết kiệm pin, làm việc nhẹ |
| `balanced` | Cân bằng, mặc định |
| `performance` | CPU max, tản nhiệt max |

Bắt Fn+Q bằng script:

Tạo `/usr/local/bin/fnq-handler.sh`:

```bash
#!/bin/bash
acpi_listen | while read -r event; do
    case "$event" in
        *ibm/hotkey*6010*|*PNP0C14*6010*)
            current=$(powerprofilesctl get)
            case "$current" in
                performance)
                    powerprofilesctl set balanced
                    notify-send "Power Profile" "Balanced" ;;
                balanced)
                    powerprofilesctl set power-saver
                    notify-send "Power Profile" "Power Saver" ;;
                power-saver)
                    powerprofilesctl set performance
                    notify-send "Power Profile" "Performance" ;;
            esac
            ;;
    esac
done
```

```bash
sudo chmod +x /usr/local/bin/fnq-handler.sh
```

Thêm vào `bspwmrc`:

```bash
/usr/local/bin/fnq-handler.sh &
```

### 5.2. TLP — tối ưu pin (thay thế Lenovo Vantage)

```bash
sudo pacman -S tlp
sudo systemctl enable --now tlp
```

Ngưỡng sạc 80% (giống Conservation Mode):

Tạo `/etc/tlp.d/01-charge-thresholds.conf`:

```
START_CHARGE_THRESH_BAT0=0
STOP_CHARGE_THRESH_BAT0=80
```

```bash
sudo tlp start
tlp-stat -b
```

### 5.3. ⚠️ Xung đột TLP + power-profiles-daemon

Cả hai cùng quản lý CPU scaling governor. **Không chạy cùng lúc.**

| Tình huống | Giải pháp |
|------------|-----------|
| Dùng TLP (kiểm soát sâu: disk, USB, PCIe) | `sudo systemctl disable --now power-profiles-daemon` |
| Dùng power-profiles-daemon (đơn giản, có Fn+Q) | `sudo systemctl disable --now tlp` |

### 5.4. Biến môi trường

> `/etc/environment` được khuyến cáo không dùng trên systemd hiện đại.

Thay vào đó, dùng:
- `~/.config/environment.d/*.conf` — systemd user session
- Shell profile: `~/.bash_profile`, `~/.zprofile`, `~/.xinitrc`

Ví dụ `~/.config/environment.d/env.conf`:

```
PATH=${PATH}:$HOME/.local/bin
EDITOR=nvim
BROWSER=firefox
```

---

## 6. Cấu hình mẫu tối thiểu

### 6.1. sxhkdrc tối thiểu

```bash
# ~/.config/sxhkd/sxhkdrc

# Terminal
super + Return
	alacritty

# Launcher
super + d
	rofi -show drun

# Close window
super + q
	bspc node -c

# Focus direction
super + {h,j,k,l}
	bspc node -f {west,south,north,east}

# Workspace
super + {1-9}
	bspc desktop -f '^{1-9}'
super + Shift + {1-9}
	bspc node -d '^{1-9}'

# Volume
XF86AudioLowerVolume
	pamixer --decrease 5
XF86AudioRaiseVolume
	pamixer --increase 5
XF86AudioMute
	pamixer --toggle-mute

# Brightness
XF86MonBrightnessDown
	brightnessctl set 5%-
XF86MonBrightnessUp
	brightnessctl set +5%

# Media
XF86AudioPlay
	playerctl play-pause
XF86AudioNext
	playerctl next
XF86AudioPrev
	playerctl previous

# Screenshot
Print
	maim -u ~/Pictures/screenshots/$(date +%Y%m%d-%H%M%S).png
super + Print
	maim -su ~/Pictures/screenshots/$(date +%Y%m%d-%H%M%S).png

# Reload sxhkd
super + Escape
	pkill -USR1 -x sxhkd
```

### 6.2. bspwmrc tối thiểu

```bash
#!/bin/bash
# ~/.config/bspwm/bspwmrc

# Monitor
xrandr --auto

# Workspaces
bspc monitor -d I II III IV V VI VII VIII IX

# Config
bspc config border_width         2
bspc config window_gap           8
bspc config split_ratio          0.50
bspc config focused_border_color "#cba6f7"
bspc config normal_border_color  "#45475a"
bspc config presel_feedback_color "#cba6f7"

# Rules
bspc rule -a Rofi:* floating on
bspc rule -a Polybar:* floating on
bspc rule -a Firefox desktop='^2'
bspc rule -a Alacritty desktop='^1'

# Launch
sxhkd &
polybar main &
nitrogen --restore &
picom --config ~/.config/picom/picom.conf &
/usr/local/bin/fnq-handler.sh &
```

```bash
chmod +x ~/.config/bspwm/bspwmrc
```

---

## 7. Tóm tắt

| Khái niệm | Giải thích |
|-----------|-----------|
| **Vai trò BIOS** | Chỉ quyết định gửi keycode nào. Không gán chức năng. |
| **Vai trò Linux** | Nhận keycode, map thành keysym. Mọi hành động đều do phần mềm quyết định. |
| **Phím Fn** | Không khác phím thường ở tầng OS. BIOS quyết định tín hiệu gửi đi. |
| **sxhkd** | Hotkey daemon, so khớp keysym X11 với config. Không thể bắt ACPI event. |
| **ACPI event** | Fn+Q gửi ACPI WMI event. Cần `acpi_listen` + script + `power-profiles-daemon`. |
| **TLP** | Thay thế Lenovo Vantage cho quản lý pin và ngưỡng sạc 80%. |
| **power-profiles-daemon** | Chuyển đổi profile hiệu năng. Xung đột với TLP. |
| **/etc/environment** | deprecated — dùng `~/.config/environment.d/*.conf` hoặc shell profile. |
