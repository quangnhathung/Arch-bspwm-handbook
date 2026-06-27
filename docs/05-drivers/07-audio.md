# Audio — PipeWire

## Mục tiêu

Cài đặt và cấu hình âm thanh với PipeWire trên Intel SST cho Lenovo LOQ 15IAX9.

## Kiến thức nền

### Intel Smart Sound Technology (SST)

Intel SST là công nghệ âm thanh kỹ thuật số tích hợp trên chipset Intel, sử dụng DSP (Digital Signal Processor). Trên Linux, Intel SST hoạt động qua Sound Open Firmware (SOF).

Các thành phần cần thiết:

- `sof-firmware`: Firmware SOF — đã cài trong pacstrap
- Kernel 6.x+ (hiện tại kernel 7.x — hỗ trợ tốt)
- ALSA + PipeWire + WirePlumber

### ALSA vs PipeWire vs PulseAudio

| Tính năng | ALSA | PulseAudio | PipeWire |
|---|---|---|---|
| Cấp | Low-level | High-level | High-level |
| Mixer | Có | Có | Có |
| Network audio | Không | Có | Có |
| Bluetooth | Không | Có | Có (tốt hơn) |
| Latency | Thấp | Trung bình | Thấp |
| Security (sandbox) | Không | Không | Có |

Chúng ta dùng **PipeWire** — chuẩn âm thanh hiện đại của Linux.

### PipeWire stack

```
Application → PipeWire → ALSA (kernel) → Hardware
                  ↕
            WirePlumber (session & policy manager)
                  ↕
            pipewire-pulse (PulseAudio compat)
                  ↕
            Ứng dụng PulseAudio cũ
```

**WirePlumber** là session manager được khuyến nghị (thay thế `pipewire-media-session` cũ).

## Các bước thực hiện

### Bước 1: Cài PipeWire và WirePlumber

```bash
pacman -S pipewire pipewire-alsa pipewire-pulse wireplumber
```

| Gói | Vai trò |
|---|---|
| `pipewire` | Core audio server |
| `pipewire-alsa` | ALSA compatibility layer |
| `pipewire-pulse` | PulseAudio compatibility (cho ứng dụng cũ) |
| `wireplumber` | Session & policy manager (quản lý thiết bị, Bluetooth, profile) |

### Bước 2: Enable user service

PipeWire chạy ở **user level** (không cần root):

```bash
systemctl --user enable pipewire pipewire-pulse wireplumber
```

Khởi động ngay:

```bash
systemctl --user start pipewire pipewire-pulse wireplumber
```

### Bước 3: Kiểm tra card âm thanh

```bash
# Liệt kê card
cat /proc/asound/cards

# Dùng aplay
aplay -l
```

Output mong đợi:

```
**** List of PLAYBACK Hardware Devices ****
card 0: PCH [HDA Intel PCH], device 0: ALC257 Analog [ALC257 Analog]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 0: PCH [HDA Intel PCH], device 3: HDMI 0 [HDMI 0]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 1: HDMI [HDA NVidia], device 3: HDMI 0 [HDMI 0]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```

### Bước 4: Kiểm tra output

```bash
# Phát thử âm thanh
speaker-test -l1 -c2

# Hoặc phát file WAV
speaker-test -t wav -c 2
```

Nhấn Ctrl+C để dừng.

### Bước 5: Cài công cụ quản lý

```bash
pacman -S pavucontrol pamixer
```

| Gói | Vai trò |
|---|---|
| `pavucontrol` | GUI mixer (điều chỉnh volume từng ứng dụng) |
| `pamixer` | CLI mixer (dùng trong keybinding sxhkd) |

### Bước 6: Kiểm tra PipeWire

```bash
pactl info
```

Output:

```
Server Name: PulseAudio (on PipeWire)
...
```

```bash
# Liệt kê sinks (thiết bị output)
pactl list sinks short

# Liệt kê sources (thiết bị input)
pactl list sources short
```

### Bước 7: Cấu hình volume trong sxhkd

Trong `~/.config/sxhkd/sxhkdrc` (dùng wrapper script `volume.sh`):

```bash
XF86AudioRaiseVolume
    ~/.local/bin/volume.sh up
XF86AudioLowerVolume
    ~/.local/bin/volume.sh down
XF86AudioMute
    ~/.local/bin/volume.sh mute
```

> Script wrapper có thể sử dụng `pamixer` hoặc `wpctl` bên trong. Xem nội dung script để biết chi tiết.

## Cấu hình nâng cao

### Chọn output device mặc định

```bash
# Liệt kê sinks
pactl list sinks short

# Set sink mặc định (dùng tên)
pactl set-default-sink alsa_output.pci-0000_00_1f.3.analog-stereo
```

### Điều chỉnh phần trăm volume

Sửa trong sxhkdrc:

```bash
XF86AudioRaiseVolume
    pamixer -i 3
XF86AudioLowerVolume
    pamixer -d 3
```

## Xác minh SOF firmware

```bash
dmesg | grep -i sof
```

Output mong đợi:

```
sof-audio-pci-intel-tgl 0000:00:1f.3: sof firmware version 2.x.x
```

## Bluetooth audio

PipeWire + WirePlumber tự động hỗ trợ Bluetooth A2DP (âm thanh chất lượng cao). Chi tiết xem bài `06-bluetooth.md`.

## Troubleshooting

### Không có âm thanh

**Diagnosis**:

```bash
# Kiểm tra PipeWire
systemctl --user status pipewire
systemctl --user status wireplumber

# Kiểm tra ALSA
aplay -l

# Kiểm tra module SOF
lsmod | grep snd_sof
```

**Fix**:

```bash
# Restart PipeWire
systemctl --user restart pipewire pipewire-pulse wireplumber

# Kiểm tra alsamixer
alsamixer
# F6 → chọn card, đảm bảo Master và PCM không bị muted (MM → nhấn m)
```

### "sof-audio-pci-intel-tgl: error: failed to load firmware"

```bash
# SOF firmware thiếu hoặc lỗi
pacman -S sof-firmware
reboot
```

### Không có HDMI audio

```bash
aplay -l | grep HDMI

# Set default sink cho HDMI
pactl set-default-sink alsa_output.pci-0000_01_00.1.hdmi-stereo
```

### Độ trễ âm thanh

```bash
# Cấu hình PipeWire
vim /etc/pipewire/pipewire.conf

# Sửa:
default.clock.rate = 48000
default.clock.allowed-rates = [ 44100 48000 ]
```

### Loa trong không hoạt động, tai nghe có

```bash
# Auto-mute của codec âm thanh
alsamixer
# F6 → chọn HDA Intel PCH
# Tìm "Auto-Mute Mode" → Disable (mũi tên xuống)
```

### Âm thanh quá nhỏ

```bash
# Tăng giới hạn gain
pamixer --set-limit 150
pamixer -i 20
```

## Tổng kết

- PipeWire + WirePlumber đã cài và enable (user service)
- SOF firmware cho Intel SST hoạt động
- pavucontrol (GUI) + pamixer (CLI) quản lý âm lượng
- Volume keys hoạt động qua sxhkd
- Bluetooth audio qua PipeWire A2DP
- Troubleshooting: alsamixer, firmware, PipeWire status
