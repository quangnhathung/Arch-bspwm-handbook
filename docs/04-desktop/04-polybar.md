# Polybar — Thanh trạng thái

Ngày cập nhật: 25/06/2026

## Mục tiêu

Cài đặt và cấu hình Polybar — thanh trạng thái hiển thị thông tin hệ thống.

## Kiến thức nền

### Polybar là gì?

Polybar là chương trình tạo thanh trạng thái (status bar) cho các window manager.
Nó hiển thị thông tin như:

- Workspace hiện tại
- Thời gian
- Pin (laptop)
- Volume
- Network status
- CPU / RAM / Disk usage

### Cách Polybar hoạt động

1. Đọc file cấu hình (`~/.config/polybar/config.ini`).
2. Tạo bar với các module được cấu hình.
3. Module có thể là built-in (như date, battery) hoặc script tùy chỉnh.

## Các bước thực hiện

### Bước 1: Cài Polybar

```bash
pacman -S polybar
```

### Bước 2: Tạo thư mục và config

```bash
su - archuser
mkdir -p ~/.config/polybar
exit
```

### Bước 3: Cấu hình config.ini

```bash
vim /home/archuser/.config/polybar/config.ini
```

Nội dung (theme Dracula):

```ini
[colors]
background = #282A36
background-alt = #44475A
foreground = #F8F8F2
foreground-alt = #6272A4
primary = #50FA7B
secondary = #FFB86C
alert = #FF5555
disabled = #44475A

[bar/main]
width = 100%
height = 28
offset-x = 0
offset-y = 0
radius = 0
fixed-center = false
background = ${colors.background}
foreground = ${colors.foreground}

line-size = 2
line-color = #f00

padding-left = 2
padding-right = 2

module-margin-left = 1
module-margin-right = 1

font-0 = "JetBrains Mono:size=11;3"
font-1 = "Noto Sans:size=10;3"
font-2 = "Symbols Nerd Font:size=11;3"

modules-left = bspwm
modules-center = date
modules-right = filesystem memory cpu temperature wlan eth battery pulseaudio powermenu

cursor-click = pointer
cursor-scroll = pointer

enable-ipc = true

[module/bspwm]
type = internal/bspwm
format = <label-state> <label-mode>
label-monitor = ${colors.foreground-alt}
label-monitor-foreground = ${colors.foreground-alt}
label-focused = %name%
label-focused-foreground = ${colors.primary}
label-focused-underline = ${colors.primary}
label-focused-padding = 2
label-unfocused = %name%
label-unfocused-foreground = ${colors.foreground-alt}
label-unfocused-padding = 2
label-visible = %name%
label-visible-foreground = ${colors.foreground-alt}
label-visible-padding = 2
label-urgent = %name%
label-urgent-foreground = ${colors.alert}
label-urgent-underline = ${colors.alert}
label-urgent-padding = 2

[module/date]
type = internal/date
interval = 1
date = %Y-%m-%d
date-alt = %A, %d %B %Y
time = %H:%M:%S
time-alt = %H:%M:%S
format = <label>
label = %date% %time%
label-foreground = ${colors.foreground}

[module/filesystem]
type = internal/filesystem
mount-0 = /
label-mounted = %mountpoint%: %percentage_used%%
label-unmounted = %mountpoint%: not mounted
label-unmounted-foreground = ${colors.disabled}
interval = 60

[module/memory]
type = internal/memory
label = RAM: %percentage_used%%
label-foreground = ${colors.secondary}
interval = 5

[module/cpu]
type = internal/cpu
label = CPU: %percentage:2%%
label-foreground = ${colors.secondary}
interval = 2

[module/temperature]
type = internal/temperature
thermal-zone = 0
label = %temperature-c%
label-foreground = ${colors.foreground}
warn-temperature = 80

[module/wlan]
type = internal/network
interface = wlan0
interval = 3
format-connected = <label-connected>
label-connected = %essid% %local_ip%
label-connected-foreground = ${colors.primary}
format-disconnected = <label-disconnected>
label-disconnected = No Wi-Fi
label-disconnected-foreground = ${colors.alert}

[module/eth]
type = internal/network
interface = eth0
interval = 3
format-connected = <label-connected>
label-connected = %local_ip%
label-connected-foreground = ${colors.primary}
format-disconnected =
label-disconnected =

[module/battery]
type = internal/battery
battery = BAT0
adapter = AC0
format-charging = <label-charging>
format-discharging = <label-discharging>
format-full = <label-full>
label-charging = %percentage%%
label-discharging = %percentage%%
label-full = %percentage%%
label-discharging-foreground = ${colors.foreground}
label-charging-foreground = ${colors.primary}
label-full-foreground = ${colors.primary}

[module/pulseaudio]
type = internal/pulseaudio
format-volume = <label-volume>
label-volume = VOL: %percentage%%
label-muted = VOL: MUTED
label-muted-foreground = ${colors.alert}
click-right = pavucontrol

[module/powermenu]
type = custom/menu
expand-right = true
label-open = ⏻
label-close = ⏻
label-separator = |
menu-0-0 = Lock
menu-0-0-exec = betterlockscreen -l
menu-0-1 = Logout
menu-0-1-exec = bspc quit
menu-0-2 = Reboot
menu-0-2-exec = systemctl reboot
menu-0-3 = Shutdown
menu-0-3-exec = systemctl poweroff
```

### Bước 4: Cập nhật bspwmrc để chạy Polybar

Trong `bspwmrc`, đã có dòng:

```bash
polybar main &
```

Nếu bạn đặt tên bar khác, sửa tương ứng.

### Bước 5: Kiểm tra

```bash
# Khởi động Polybar thử
polybar main

# Nếu lỗi, xem log
polybar main 2>&1 | head -20
```

## Multi-monitor support

Nếu bạn có nhiều màn hình, dùng script này thay vì `polybar main &`:

```bash
#!/bin/bash
for m in $(polybar --list-monitors | cut -d: -f1); do
    MONITOR=$m polybar main &
done
```

Lưu thành `~/.config/polybar/launch.sh`, cấp executable, và gọi trong bspwmrc.

## Các module phổ biến

| Module | Hiển thị |
|---|---|
| `bspwm` | Danh sách workspace |
| `date` | Ngày giờ |
| `cpu` | CPU usage |
| `memory` | RAM usage |
| `filesystem` | Dung lượng ổ |
| `network` | IP, SSID |
| `battery` | Pin laptop |
| `pulseaudio` | Volume |
| `temperature` | Nhiệt độ CPU |

## Tùy chỉnh

### Font

Polybar cần Nerd Fonts để hiển thị icon. Cài:

```bash
pacman -S ttf-jetbrains-mono ttf-nerd-fonts-symbols noto-fonts
```

### Click events

Polybar hỗ trợ click chuột trái/phải trên module.
Ví dụ: click volume → mở pavucontrol.

## Troubleshooting

### Polybar không hiển thị

- Kiểm tra `pgrep -x polybar`.
- Kiểm tra config syntax: `polybar main --print-logfile=/tmp/polybar.log`.
- Xem log: `cat /tmp/polybar.log`.

### Workspace không hiển thị đúng

- Module `bspwm` cần `bspc` trong PATH.
- Polybar phải chạy sau bspwm.

### Font hiển thị sai hoặc icon bị ô vuông

- Cài `ttf-nerd-fonts-symbols` và `noto-fonts-emoji`.
- Kiểm tra tên font trong config (dùng `fc-list | grep -i nerd` để xem tên chính xác).

## Tổng kết

- Polybar đã được cấu hình với các module cần thiết.
- Thanh trạng thái hiển thị workspace, CPU, RAM, network, battery, volume.
- Tích hợp click event cho volume và power menu.
