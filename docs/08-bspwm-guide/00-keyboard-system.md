# Hệ thống Bàn phím & Keybinding trong Linux (Lenovo LOQ)

Tài liệu giải thích cách thức hoạt động của bàn phím trên Linux, hành vi phím Fn trên laptop Lenovo LOQ, và cách xây dựng hệ thống keybinding hoàn chỉnh trong bspwm.

---

## 1. Tổng quan về hệ thống xử lý phím trong Linux

Mỗi lần nhấn phím trải qua các tầng sau:

```
Phím vật lý → BIOS → Kernel (evdev) → X11 / Wayland → Window Manager → Ứng dụng
```

| Tầng | Vai trò |
|------|---------|
| **BIOS/EC** | Nhận tín hiệu phím vật lý, quyết định gửi keycode nào xuống OS. Có thể biến đổi tín hiệu (vd: Fn + F1 → volume down) |
| **Kernel (evdev)** | Nhận scancode từ bàn phím, map thành keycode Linux. Tạo event qua `/dev/input/event*` |
| **X11 (X server)** | Nhận event từ kernel, map keycode thành keysym (tên symbolic) dùng XKB. Phát ra XF86 keysym cho phím đặc biệt |
| **Window Manager** | sxhkd (trong bspwm) lắng nghe X11 event, so khớp với config, thực thi lệnh |
| **Ứng dụng** | Nhận key event nếu không bị WM chặn |

**Key point:** Phím vật lý và hành động phần mềm là hai việc hoàn toàn khác nhau. BIOS quyết định gửi tín hiệu gì; Linux quyết định xử lý tín hiệu đó ra sao.

---

## 2. Hành vi phím Fn trên Lenovo LOQ

### 2.1. Chế độ Hotkey trong BIOS

Lenovo LOQ có tùy chọn trong BIOS (F2 khi boot → **Configuration** → **Hotkey Mode**):

| Chế độ | Hành vi F1-F12 mặc định | Cách dùng Fn |
|--------|------------------------|--------------|
| **Enabled** (mặc định) | Multimedia action (volume, brightness, etc.) | Fn + F1-F12 mới ra F1-F12 |
| **Disabled** | F1-F12 tiêu chuẩn | Fn + F1-F12 mới ra multimedia action |

**Tác dụng duy nhất của BIOS:** Thay đổi tín hiệu gửi xuống OS. BIOS **không** gán chức năng. Nếu Hotkey Mode = Enabled, khi nhấn F2, BIOS gửi `XF86MonBrightnessDown` thay vì `F2`.

### 2.2. Phím Fn + Q (Performance Mode)

Phím Fn + Q trên Lenovo LOQ gửi tín hiệu ACPI đặc biệt (WMI event) chứ không phải keycode X11 thông thường. Linux mặc định **không** tự động chuyển chế độ hiệu năng khi nhấn tổ hợp này.

### 2.3. Các phím đặc biệt trên LOQ

| Phím | Tín hiệu (Hotkey ON) | Keysym X11 |
|------|---------------------|------------|
| F1 | Mic mute | `XF86AudioMicMute` |
| F2 | Giảm độ sáng | `XF86MonBrightnessDown` |
| F3 | Tăng độ sáng | `XF86MonBrightnessUp` |
| F4 | Chuyển màn hình | `XF86Display` |
| F5 | Chụp màn hình | (Print / `XF86ScreenSaver`) |
| F6 | Giảm âm lượng | `XF86AudioLowerVolume` |
| F7 | Tăng âm lượng | `XF86AudioRaiseVolume` |
| F8 | Tắt tiếng | `XF86AudioMute` |
| F9 | Play/Pause | `XF86AudioPlay` |
| F10 | Previous | `XF86AudioPrev` |
| F11 | Next | `XF86AudioNext` |
| F12 | Máy bay | `XF86WLAN` |

> Ghi chú: Các tín hiệu trên chỉ đúng khi Hotkey Mode = Enabled. Khi tắt, các phím F1-F12 gửi keycode số (F1 = 67, F2 = 68,...) như bàn phím thường, và Fn+... mới gửi các tín hiệu trên.

---

## 3. Keycode Linux và tầng Event

### 3.1. XF86 keysym

Các phím đa phương tiện (volume, brightness, media) được X11 gán keysym bắt đầu bằng `XF86`. Một số keysym phổ biến:

| Keysym | Mục đích | Ghi chú |
|--------|----------|---------|
| `XF86AudioRaiseVolume` | Tăng âm lượng | Thường dùng +pamixer |
| `XF86AudioLowerVolume` | Giảm âm lượng | |
| `XF86AudioMute` | Tắt/bật tiếng | |
| `XF86AudioPlay` | Play/Pause | Cần playerctl |
| `XF86AudioNext` | Bài tiếp | |
| `XF86AudioPrev` | Bài trước | |
| `XF86MonBrightnessUp` | Tăng sáng | Cần brightnessctl |
| `XF86MonBrightnessDown` | Giảm sáng | |
| `XF86AudioMicMute` | Tắt mic | |
| `XF86Display` | Chuyển màn hình | |
| `XF86WLAN` | Bật/tắt Wi-Fi | |

### 3.2. Phát hiện keycode với xev

Dùng để kiểm tra phím nào đang gửi keysym gì:

```bash
xev | grep keysym
```

Nhấn phím cần kiểm tra, output sẽ hiện:

```
state 0x0, keycode 122 (keysym 0x1008ff13, XF86AudioLowerVolume), same_screen YES
```

### 3.3. Phát hiện event với evtest

Dùng để debug ở tầng kernel, đặc biệt hữu ích khi X11 không nhận được event:

```bash
sudo pacman -S evtest
sudo evtest
```

Chọn thiết bị bàn phím, nhấn phím và xem raw scancode. Hữu ích khi phím hoạt động ở kernel level nhưng không lên X11.

### 3.4. Vai trò của ACPI / WMI

Một số phím đặc biệt (Fn+Q, Fn+ F4, Fn+F12) không gửi keycode X11 mà gửi qua ACPI event. Có thể xem bằng:

```bash
acpi_listen
```

Nếu phím gửi ACPI event thay vì X11 keysym, cần tool chuyên biệt (ví dụ `systemd-logind` hoặc script riêng) để xử lý, không thể dùng sxhkd thuần túy.

---

## 4. Hệ thống Keybinding trong bspwm

### 4.1. Kiến trúc

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
- **sxhkd** là hotkey daemon riêng biệt, lắng nghe X11 event, so khớp với config, thực thi lệnh.
- Hai process độc lập: bspwm không biết về keybinding; sxhkd không biết về window management.

### 4.2. File cấu hình

| File | Đường dẫn | Vai trò |
|------|-----------|---------|
| sxhkdrc | `~/.config/sxhkd/sxhkdrc` | Định nghĩa tất cả keybinding |
| bspwmrc | `~/.config/bspwm/bspwmrc` | Cấu hình bspwm + khởi động các chương trình |

### 4.3. Cấu trúc sxhkdrc

```
# Comment
modifier + key
    command

# Ví dụ
super + Return
    alacritty

super + {h,j,k,l}
    bspc node -f {west,south,north,east}
```

Cú pháp mở rộng:
- `{a,b,c}` — batch, sinh nhiều binding từ một dòng
- `{1-9}` — range, tương tự
- `;` — chain (binding nối tiếp)
- `~` — grab release (ít dùng)

### 4.4. Nạp lại cấu hình

Sau khi sửa `sxhkdrc`:

```bash
pkill -USR1 -x sxhkd
```

hoặc binding trong sxhkdrc:

```
super + Escape
    pkill -USR1 -x sxhkd && bspc wm -r
```

---

## 5. Ví dụ Keybinding thực tế

### 5.1. Volume control

```bash
# Yêu cầu: pamixer (hoặc pipewire-pulse)
sudo pacman -S pamixer
```

```
XF86AudioLowerVolume
    pamixer --decrease 5
XF86AudioRaiseVolume
    pamixer --increase 5
XF86AudioMute
    pamixer --toggle-mute
```

### 5.2. Brightness control

```bash
# Yêu cầu: brightnessctl
sudo pacman -S brightnessctl
```

```
XF86MonBrightnessDown
    brightnessctl set 5%-
XF86MonBrightnessUp
    brightnessctl set +5%
```

### 5.3. Media control

```bash
# Yêu cầu: playerctl
sudo pacman -S playerctl
```

```
XF86AudioPlay
    playerctl play-pause
XF86AudioNext
    playerctl next
XF86AudioPrev
    playerctl previous
```

### 5.4. Screenshot

```bash
# Yêu cầu: flameshot hoặc maim
sudo pacman -S flameshot
```

```
Print
    flameshot full -p ~/Pictures/screenshots
super + Print
    flameshot gui -p ~/Pictures/screenshots
```

### 5.5. App launcher

```bash
# Yêu cầu: rofi
sudo pacman -S rofi
```

```
super + d
    rofi -show drun
super + Shift + d
    rofi -show run
```

### 5.6. Window management (bspc)

```
# Focus
super + {h,j,k,l}
    bspc node -f {west,south,north,east}

# Move window sang hướng khác
super + Shift + {h,j,k,l}
    bspc node -s {west,south,north,east}

# Close
super + q
    bspc node -c

# Fullscreen toggle
super + f
    bspc node -t ~fullscreen

# Float toggle
super + space
    bspc node -t floating; bspc node -g sticky
```

### 5.7. Workspace switching

```
# Chuyển workspace
super + {1-9}
    bspc desktop -f '^{1-9}'

# Gửi window sang workspace khác
super + Shift + {1-9}
    bspc node -d '^{1-9}'
```

---

## 6. Xử lý Fn+Q (Performance Mode)

### 6.1. Vấn đề

Fn+Q gửi ACPI WMI event, **không** gửi X11 keysym. Do đó, `sxhkd` không thể bắt được phím này.

### 6.2. Kiểm tra

Trong terminal chạy:

```bash
acpi_listen
```

Nhấn Fn+Q. Nếu thấy output như `ibm/hotkey HKEY 00000080 00006010` thì đây là ACPI event.

### 6.3. Giải pháp: power-profiles-daemon

Cách đơn giản nhất để có chức năng chuyển chế độ hiệu năng:

```bash
sudo pacman -S power-profiles-daemon
sudo systemctl enable --now power-profiles-daemon
```

Kiểm tra:

```bash
powerprofilesctl list
```

Output:

```
Platform Driver: intel_pstate
Performance:
    Driver: intel_pstate
Balanced:
    Driver: intel_pstate
Power Saver:
    Driver: intel_pstate
```

Sau đó, để bắt Fn+Q và chuyển profile, dùng script kết hợp `acpi_listen` và `powerprofilesctl`.

Tạo file `/usr/local/bin/fnq-handler.sh`:

```bash
#!/bin/bash
acpi_listen | while read -r event; do
    case "$event" in
        *ibm/hotkey*6010*)
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

Thêm vào bspwmrc:

```bash
/usr/local/bin/fnq-handler.sh &
```

### 6.4. Tool Lenovo-specific (tùy chọn)

```bash
yay -S lenovo-legion-linux-git
```

Tool này cung cấp `legion-cli` để điều khiển Fn+Q, LED, và các tính năng Lenovo khác. Tuy nhiên, `power-profiles-daemon` là giải pháp đơn giản và đủ dùng cho đa số người dùng.

---

## 7. Tích hợp Power Management

### 7.1. TLP — Tối ưu pin laptop

```bash
sudo pacman -S tlp
sudo systemctl enable --now tlp
```

TLP tự động quản lý các thông số tiết kiệm pin: CPU scaling governor, disk power management, PCI Express ASPM, USB autosuspend.

**Ngưỡng sạc 80% (giống Lenovo Vantage Conservation Mode):**

Tạo `/etc/tlp.d/01-charge-thresholds.conf`:

```
START_CHARGE_THRESH_BAT0=0
STOP_CHARGE_THRESH_BAT0=80
```

Áp dụng ngay:

```bash
sudo tlp start
```

Kiểm tra:

```bash
tlp-stat -b
```

### 7.2. power-profiles-daemon — Chuyển đổi hiệu năng

| Profile | Use case | Tương ứng Lenovo Vantage |
|---------|----------|--------------------------|
| `power-saver` | Tiết kiệm pin, làm việc nhẹ | Quiet Mode (Fn+Q xanh) |
| `balanced` | Cân bằng, mặc định | Balanced Mode (Fn+Q trắng) |
| `performance` | CPU max, tản nhiệt max | Performance Mode (Fn+Q đỏ) |

Cài đặt:

```bash
sudo pacman -S power-profiles-daemon
sudo systemctl enable --now power-profiles-daemon
```

### 7.3. TLP + power-profiles-daemon có xung đột không?

Có, nếu cả hai cùng quản lý CPU scaling governor. Giải pháp:

- **Dùng TLP** nếu muốn kiểm soát sâu toàn bộ hệ thống (cả disk, USB, PCIe).
- **Dùng power-profiles-daemon** nếu chỉ cần chuyển profile đơn giản + Fn+Q support.
- Không chạy cả hai cùng lúc. Nếu dùng TLP, tắt power-profiles-daemon: `sudo systemctl disable --now power-profiles-daemon`.

### 7.4. Lưu ý

- Trên Intel (Alder Lake / Raptor Lake), `intel_pstate` driver hỗ trợ active mode, power-profiles-daemon có thể điều khiển trực tiếp.
- Kiểm tra driver: `cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_driver`.

---

## 8. Keybinding Stack đề xuất

### 8.1. Danh sách gói

| Công cụ | Mục đích | Cài đặt |
|---------|----------|---------|
| bspwm | Window manager | `pacman -S bspwm` |
| sxhkd | Hotkey daemon | `pacman -S sxhkd` |
| rofi | App launcher, power menu | `pacman -S rofi` |
| polybar | Status bar | `pacman -S polybar` |
| brightnessctl | Điều chỉnh độ sáng | `pacman -S brightnessctl` |
| pamixer | Volume CLI | `pacman -S pamixer` |
| playerctl | Media control | `pacman -S playerctl` |
| flameshot | Chụp màn hình | `pacman -S flameshot` |
| maim | Chụp màn hình (nhẹ hơn) | `pacman -S maim` |
| acpi_listen | Debug ACPI event | `pacman -S acpid` |
| power-profiles-daemon | Chuyển đổi hiệu năng | `pacman -S power-profiles-daemon` |

### 8.2. Mối quan hệ giữa các thành phần

```
  Người dùng nhấn phím
       │
       ▼
  ┌───────────┐
  │  BIOS/EC  │  Quyết định gửi XF86 keysym hay keycode thường
  └─────┬─────┘
        │
  ┌─────▼─────┐
  │  X Server │  Map thành keysym X11
  └─────┬─────┘
        │
  ┌─────▼─────┐
  │   sxhkd   │  So khớp keysym với sxhkdrc
  └─────┬─────┘
        │
  ┌─────▼─────────┐
  │  Command exec  │  bspc | pamixer | brightnessctl | playerctl ...
  └───────────────┘
```

---

## 9. Cấu hình mẫu tối thiểu

### 9.1. sxhkdrc tối thiểu

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
    flameshot full -p ~/Pictures/screenshots
super + Print
    flameshot gui -p ~/Pictures/screenshots

# Reload sxhkd
super + Escape
    pkill -USR1 -x sxhkd
```

### 9.2. bspwmrc tối thiểu

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
bspc config normal_border_color   "#45475a"
bspc config presel_feedback_color "#cba6f7"

# Rules
bspc rule -a Rofi:*:rofi floating on
bspc rule -a Polybar:*:polybar floating on
bspc rule -a Firefox:^2 desktop='^2'
bspc rule -a Alacritty:*:alacritty desktop='^1'

# Launch
sxhkd &
polybar main &
nitrogen --restore &
picom --config ~/.config/picom/picom.conf &
/usr/local/bin/fnq-handler.sh &
```

Chạy:

```bash
chmod +x ~/.config/bspwm/bspwmrc
```

---

## 10. Tóm tắt

| Khái niệm | Giải thích |
|-----------|-----------|
| **Vai trò BIOS** | Chỉ quyết định gửi keycode nào. Không gán chức năng. Hotkey Mode = Enabled gửi XF86 keysym; = Disabled gửi F1-F12 thường |
| **Vai trò Linux** | Nhận keycode, map thành keysym. Mọi hành động (volume, brightness, workspace) đều do phần mềm quyết định, không có gì tự động |
| **Phím Fn** | Không khác phím thường ở tầng OS. BIOS quyết định Fn+F1 là F1 hay XF86AudioMicMute |
| **sxhkd** | Hotkey daemon, so khớp keysym X11 với config, thực thi lệnh. Không thể bắt ACPI event (Fn+Q) |
| **ACPI event** | Fn+Q gửi ACPI WMI event. Cần `acpi_listen` + script riêng + `power-profiles-daemon` |
| **TLP** | Thay thế Lenovo Vantage cho quản lý pin và ngưỡng sạc 80% |
| **Tại sao cần cấu hình thủ công** | Linux không gán hành động mặc định cho phím chức năng. Người dùng phải tự gắn `brightnessctl`, `pamixer`, `playerctl` vào từng phím |
