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

### Bước 3: Cấu hình config.ini — Floating Dynamic Island Bar

```bash
vim /home/archuser/.config/polybar/config.ini
```

**Cấu hình thực tế:** Polybar hoạt động như một **Dynamic Island** kiểu macOS:
bar nổi hoàn toàn (override-redirect), bo tròn 20px, ẩn mặc định, chỉ hiện khi
gọi bằng phím tắt (`Super + Shift + b`).

```ini
[colors]
background = #16181a
foreground = #FFFFFFFF
primary = #89B4FA
border = #30FFFFFF

[bar/island]
width = 78%
height = 60
offset-x = 11%    ; (100 - 78) / 2 = căn giữa
offset-y = 12
radius = 20
fixed-center = true

background = ${colors.background}
foreground = ${colors.foreground}
border-color = ${colors.border}

line-size = 3
border-size = 1
padding-left = 2
padding-right = 3
module-margin = 1

font-0 = "JetBrainsMono Nerd Font:size=11;3"
font-1 = "JetBrainsMono Nerd Font:size=13;4"

enable-ipc = true
override-redirect = true   ; Floating hoàn toàn, không chiếm không gian bspwm

modules-left = power user wifi cpu mic-status
modules-center = bspwm
modules-right = gpu pulseaudio battery date

include-file = ~/.config/polybar/modules/bspwm.ini
include-file = ~/.config/polybar/modules/date.ini
include-file = ~/.config/polybar/modules/battery.ini
include-file = ~/.config/polybar/modules/audio.ini
include-file = ~/.config/polybar/modules/user.ini
include-file = ~/.config/polybar/modules/power.ini
include-file = ~/.config/polybar/modules/wifi.ini
include-file = ~/.config/polybar/modules/hw_mic.ini
```

### Bước 4: Launch script — khởi động + ẩn chờ peek

Tạo `~/.config/polybar/launch.sh`:

```bash
#!/usr/bin/env bash

killall -q polybar
while pgrep -u $UID -x polybar >/dev/null; do sleep 0.1; done

polybar island 2>&1 | tee -a /tmp/polybar.log & disown

# Chờ X11 tạo cửa sổ Polybar → lấy ID → đẩy lên trên cùng
POLYBAR_ID=""
while [ -z "$POLYBAR_ID" ]; do
    POLYBAR_ID=$(xdotool search --class "polybar" | head -n 1)
    sleep 0.1
done

xdotool windowraise $POLYBAR_ID

# Ẩn bar ngay sau khi launch, chờ peek.sh gọi lại
until polybar-msg cmd hide >/dev/null 2>&1; do
    sleep 0.1
done
```

### Bước 5: Peek script — hiện/ẩn Dynamic Island

Tạo `~/.config/polybar/peek.sh`:

```bash
#!/usr/bin/env bash

PIDFILE="/tmp/polybar_peek.pid"

# Nếu bar ĐANG HIỆN → tắt
if [ -f "$PIDFILE" ]; then
    kill $(cat "$PIDFILE") 2>/dev/null
    rm -f "$PIDFILE"
    polybar-msg cmd hide
    exit 0
fi

# Nếu bar ĐANG ẨN → hiện lên
polybar-msg cmd show
xdotool search --class "polybar" windowraise 2>/dev/null

# Tự động ẩn sau 8 giây
(
    sleep 8
    polybar-msg cmd hide
    rm -f "$PIDFILE"
) &

echo $! > "$PIDFILE"
```

Gọi từ sxhkdrc:
```
super + shift + b
    ~/.config/polybar/peek.sh
```

### Bước 6: Modules chi tiết

Các file module được include trong `~/.config/polybar/modules/`:

#### `bspwm.ini` — Workspace dots (pure dot pager)

```ini
[module/bspwm]
type = internal/bspwm
ws-icon-default = ●
label-focused = ●
label-focused-foreground = #fa89b4
label-occupied = ●
label-occupied-foreground = #e1e6f5
label-urgent = ●
label-urgent-foreground = #F38BA8
label-empty =                ; Ẩn workspace trống
```

Workspace hiển thị dưới dạng **dots** (chấm tròn), không hiện tên. Focused = hồng,
occupied = trắng, urgent = đỏ, empty = ẩn.

#### `audio.ini` — PulseAudio volume

```ini
[module/pulseaudio]
type = internal/pulseaudio
format-volume = <ramp-volume> <label-volume>
label-volume = %percentage%%
format-muted = <label-muted>
format-muted-prefix = "󰝟 "
label-muted = Muted
ramp-volume-0 = ""
ramp-volume-1 = ""
ramp-volume-2 = ""
click-right = pavucontrol
```

Icon loa tự động đổi:  (nhỏ) →  (vừa) →  (to). Click phải → pavucontrol.

#### `wifi.ini` — Network SSID + signal

```ini
[module/wifi]
type = internal/network
interface = wlan0
interval = 3.0
format-connected = <ramp-signal> <label-connected>
label-connected = %essid%
ramp-signal-0 = "󰤯"  ; yếu
ramp-signal-1 = "󰤟"
ramp-signal-2 = "󰤢"
ramp-signal-3 = "󰤥"
ramp-signal-4 = "󰤨"  ; mạnh
format-disconnected-prefix = "󰤭 "
label-disconnected = Offline
```

#### `battery.ini` — Pin laptop

```ini
[module/battery]
type = internal/battery
battery = BAT1
adapter = ACAD
label-charging =  %percentage%%
label-discharging =  %percentage%%
```

#### `date.ini` — Đồng hồ

```ini
[module/date]
type = internal/date
interval = 1
date = %d/%m
time = %H:%M
format-prefix = " "
label = %time%  |  %date%
```

#### `user.ini` — Tên user

```ini
[module/user]
type = custom/script
exec = whoami
interval = 1024
format-prefix = " "
```

#### `power.ini` — Nút tắt máy

```ini
[module/power]
type = custom/text
content = ""
content-foreground = #F38BA8
click-left = ~/.local/bin/powermenu.sh
click-right = systemctl reboot
```

#### `hw_mic.ini` — CPU + GPU + Mic status

File này chứa 3 module gộp:
- **cpu** — internal/cpu, interval 2s, prefix ``
- **gpu** — custom script (xem bên dưới), interval 2s
- **mic-status** — custom script (xem bên dưới), interval 1s

### Bước 7: Script tùy chỉnh

#### `scripts/gpu.sh` — GPU monitoring (Intel + NVIDIA)

```bash
#!/usr/bin/env bash
output=""

# Intel iGPU
for dir in /sys/class/drm/card*; do
    if [ -f "$dir/device/vendor" ]; then
        vendor=$(cat "$dir/device/vendor")
        if [ "$vendor" == "0x8086" ]; then
            if [ -f "$dir/gt_cur_freq_mhz" ]; then
                freq=$(cat "$dir/gt_cur_freq_mhz")
                output+="%{F#89B4FA}Intel:%{F-} ${freq}MHz"
            fi
            break
        fi
    fi
done

# NVIDIA dGPU
if command -v nvidia-smi &> /dev/null; then
    nvidia_load=$(nvidia-smi --query-gpu=utilization.gpu --format=csv,noheader,nounits | tr -d ' ')
    if [ -n "$nvidia_load" ]; then
        [ -n "$output" ] && output+="  |  "
        output+="%{F#A6E3A1}NVIDIA:%{F-} ${nvidia_load}%"
    fi
fi

echo "${output:-GPU: N/A}"
```

#### `scripts/mic.sh` — Mic recording detection

```bash
#!/usr/bin/env bash
is_recording=$(pactl list source-outputs | grep -c 'Source Output #')
if [ "$is_recording" -gt 0 ]; then
    echo "%{F#F38BA8}󰍬 %{F-}On"
else
    echo ""              # Output rỗng → module tự ẩn
fi
```

### Bước 8: Cập nhật bspwmrc

```bash
~/.config/polybar/launch.sh &
```

### Bước 9: Cài đặt phụ thuộc

```bash
sudo pacman -S xdotool   # Cần cho launch.sh và peek.sh
```

## Các module trên bar

| Vị trí | Module | Hiển thị |
|--------|--------|----------|
| Trái | `power` | Icon  → click = powermenu |
| Trái | `user` | Tên user |
| Trái | `wifi` | Tên Wi-Fi + vạch sóng |
| Trái | `cpu` | CPU usage % |
| Trái | `mic-status` | Icon mic đỏ khi có app đang ghi âm |
| Giữa | `bspwm` | Dot pager (● = workspace) |
| Phải | `gpu` | Intel MHz + NVIDIA % |
| Phải | `pulseaudio` | Icon loa + volume % |
| Phải | `battery` | % pin + icon sạc |
| Phải | `date` | Đồng hồ HH:MM |

## Tùy chỉnh

### Font

Cần `ttf-jetbrains-mono` và `ttf-nerd-fonts-symbols`:

```bash
sudo pacman -S ttf-jetbrains-mono ttf-nerd-fonts-symbols
```

### Thay đổi thời gian peek

Trong `peek.sh`, sửa dòng `sleep 8` thành số giây mong muốn.

### Thêm module

1. Tạo file `.ini` trong `~/.config/polybar/modules/`.
2. Thêm tên module vào `modules-left/center/right` trong `config.ini`.
3. Thêm `include-file` vào cuối `config.ini`.

## Troubleshooting

### Polybar không hiển thị

```bash
pgrep -x polybar                          # Kiểm tra polybar đã chạy
cat /tmp/polybar.log                      # Xem log lỗi
```

### Workspace dots không hiện

- Module `bspwm` yêu cầu `bspc` trong PATH.
- Polybar phải chạy **sau** bspwm (trong bspwmrc).

### Font/Icon bị ô vuông

```bash
sudo pacman -S ttf-nerd-fonts-symbols
fc-cache -fv
```

### Peek không hoạt động

- Kiểm tra `xdotool` đã cài: `which xdotool`.
- Kiểm tra IPC: `polybar-msg cmd toggle`.

## Tổng kết

- Polybar chạy ở chế độ **Dynamic Island**: floating, bo tròn, ẩn mặc định.
- **Peek mechanism**: `Super + Shift + b` để xem bar trong 8 giây.
- **7 modules** chính: power, user, wifi, cpu, mic-status, bspwm, gpu, audio, battery, date.
- **2 custom scripts**: `gpu.sh` (Intel + NVIDIA) và `mic.sh` (recording detection).
- Launch script dùng `xdotool` để quản lý Z-order.
