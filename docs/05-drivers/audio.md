# Audio — PipeWire

## Mục tiêu

Cài đặt và cấu hình âm thanh với PipeWire trên Intel SST.

## Kiến thức nền

### Intel Smart Sound Technology (SST)

Intel SST là công nghệ âm thanh kỹ thuật số tích hợp trên chipset Intel.
Nó sử dụng DSP (Digital Signal Processor) để xử lý âm thanh.

Để Intel SST hoạt động trên Linux, cần:

- `sof-firmware`: Firmware cho Sound Open Firmware (SOF) — đã cài trong pacstrap.
- Kernel mới (6.x+) — đã cài linux.
- ALSA + PipeWire.

### ALSA vs PulseAudio vs PipeWire

| | ALSA | PulseAudio | PipeWire |
|---|---|---|---|
| Cấp | Low-level | High-level | High-level |
| Mixer | Có | Có | Có |
| Network audio | Không | Có | Có |
| Bluetooth | Không | Có | Có (tốt hơn) |
| Latency | Thấp | Trung bình | Thấp |
| Modern | Cũ | Đang bị thay thế | Mới, đang phát triển |

Chúng ta dùng **PipeWire** — chuẩn âm thanh mới của Linux, thay thế PulseAudio.

### PipeWire stack

```
Application → PipeWire → ALSA (kernel) → Hardware
                  ↕
            WirePlumber (session manager)
                  ↕
            pipewire-pulse (PulseAudio compat)
                  ↕
            Ứng dụng PulseAudio cũ
```

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
| `wireplumber` | Session & policy manager (quản lý thiết bị) |

### Bước 2: Enable PipeWire service

```bash
systemctl --user enable pipewire pipewire-pulse wireplumber
```

PipeWire chạy ở user level (không cần root).

### Bước 3: Kiểm tra card âm thanh

```bash
# Liệt kê card âm thanh
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

# Hoặc dùng aplay
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
| `pamixer` | CLI mixer (dùng trong keybinding) |

### Bước 6: Kiểm tra PipeWire

```bash
# Kiểm tra trạng thái
pactl info

# Output:
# Server Name: PulseAudio (on PipeWire)
# ...
```

```bash
# Liệt kê sinks (thiết bị output)
pactl list sinks short
```

```bash
# Liệt kê sources (thiết bị input)
pactl list sources short
```

### Bước 7: Cấu hình volume trong sxhkd

Đã cấu hình trong sxhkdrc:

```
XF86AudioRaiseVolume
    pamixer -i 5
XF86AudioLowerVolume
    pamixer -d 5
XF86AudioMute
    pamixer -t
```

## Cấu hình nâng cao

### Chọn output device mặc định

```bash
# Liệt kê sinks
pactl list sinks short

# Set sink mặc định (dùng tên hoặc index)
pactl set-default-sink alsa_output.pci-0000_00_1f.3.analog-stereo
```

### Điều chỉnh phần trăm volume mỗi lần nhấn

Sửa trong sxhkdrc:

```
XF86AudioRaiseVolume
    pamixer -i 3
XF86AudioLowerVolume
    pamixer -d 3
```

## Xác minh SOF firmware

```bash
# Kiểm tra SOF đã được load
dmesg | grep -i sof
```

Output mong đợi:

```
sof-audio-pci-intel-tgl 0000:00:1f.3: sof firmware version 2.x.x
```

## Troubleshooting

### Không có âm thanh

**Symptoms**: speaker-test không ra tiếng.

**Cause**: Card âm thanh sai mặc định, hoặc thiếu firmware, hoặc PipeWire chưa chạy.

**Diagnosis**:

```bash
# Kiểm tra PipeWire
systemctl --user status pipewire
systemctl --user status wireplumber

# Kiểm tra ALSA
aplay -l
cat /proc/asound/cards

# Kiểm tra module
lsmod | grep snd_sof
```

**Fix**:

```bash
# Restart PipeWire
systemctl --user restart pipewire pipewire-pulse wireplumber

# Nếu vẫn không được, kiểm tra alsamixer
alsamixer
# Nhấn F6 để chọn card, đảm bảo Master và PCM không bị muted (MM → nhấn m để unmute)
```

### "sof-audio-pci-intel-tgl: error: failed to load firmware"

```bash
# SOF firmware thiếu → cài lại
pacman -S sof-firmware
```

### Không có HDMI audio

```bash
# Kiểm tra NVIDIA HDMI
aplay -l | grep HDMI

# Nếu có card NVIDIA HDMI, set làm default
pactl set-default-sink alsa_output.pci-0000_01_00.1.hdmi-stereo
```

### Độ trễ âm thanh (audio lag)

PipeWire mặc định có latency thấp. Nếu bị lag:

```bash
# Cấu hình PipeWire
vim /etc/pipewire/pipewire.conf

# Tìm và sửa:
default.clock.rate = 48000
default.clock.allowed-rates = [ 44100 48000 ]
```

### Loa trong không hoạt động nhưng tai nghe có

**Cause**: Auto-mute của codec âm thanh.

**Fix**:

```bash
# Vào alsamixer
alsamixer
# Chọn card HDA Intel PCH
# Nhấn F6 → chọn HDA Intel PCH
# Tìm "Auto-Mute Mode" → Disable
```

### Âm thanh quá nhỏ

```bash
# Tăng gain
pamixer --set-limit 150
pamixer -i 20
```

## Tổng kết

- PipeWire + WirePlumber đã được cài và enable.
- SOF firmware cho Intel SST hoạt động.
- Công cụ pavucontrol và pamixer đã được cài.
- Volume control qua phím chức năng hoạt động.
- Troubleshooting: kiểm tra firmware, alsamixer, PipeWire status.
