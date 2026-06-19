# Không có âm thanh

## Symptoms

- Không có tiếng từ loa trong.
- Tai nghe không hoạt động.
- HDMI audio không có.
- Volume icon báo muted hoặc không thấy thiết bị.

## Cause

1. **PipeWire chưa chạy** hoặc chưa được enable.
2. **ALSA card sai** — card mặc định là HDMI thay vì analog.
3. **SOF firmware thiếu** — Intel SST không hoạt động.
4. **Muted trong alsamixer**.
5. **WirePlumber chưa chạy**.
6. **Module sai** — pipewire-alsa hoặc pipewire-pulse thiếu.

## Diagnosis

```bash
# Kiểm tra PipeWire
systemctl --user status pipewire
systemctl --user status pipewire-pulse
systemctl --user status wireplumber

# Kiểm tra ALSA cards
cat /proc/asound/cards

# Kiểm tra output devices
pactl list sinks short

# Kiểm tra SOF firmware
dmesg | grep -i sof

# Kiểm tra alsamixer
alsamixer
# Nhấn F6 để chọn card, kiểm tra Master và PCM
```

## Fix

### Fix 1: Start PipeWire

```bash
systemctl --user enable --now pipewire pipewire-pulse wireplumber
```

### Fix 2: Kiểm tra và chọn đúng card

```bash
# Xem card
aplay -l

# Set card mặc định
cat /etc/asound.conf

# Hoặc set bằng pactl
pactl set-default-sink <tên-sink>
```

### Fix 3: Cài lại SOF firmware

```bash
sudo pacman -S sof-firmware alsa-firmware
sudo reboot
```

### Fix 4: Unmute trong alsamixer

```bash
alsamixer
# F6 → chọn card HDA Intel PCH
# Phím m → mute/unmute
# Đảm bảo Master, PCM, Headphone không có chữ MM
# ↑ để tăng volume
```

### Fix 5: Kiểm tra kernel module

```bash
lsmod | grep snd_hda_intel
lsmod | grep snd_sof

# Nếu thiếu
sudo modprobe snd_hda_intel
```

### Fix 6: Cài PipeWire components đầy đủ

```bash
sudo pacman -S pipewire pipewire-alsa pipewire-pulse wireplumber
```

### Fix 7: Restart âm thanh

```bash
systemctl --user restart pipewire pipewire-pulse wireplumber
```

## Prevention

1. **Cài đầy đủ PipeWire + WirePlumber ngay từ đầu**.
2. **Cài sof-firmware trong pacstrap** (đã làm).
3. **Kiểm tra alsamixer sau mỗi lần update kernel**.
4. **Enable user services** ngay sau khi cài:

```bash
systemctl --user enable pipewire pipewire-pulse wireplumber
```
