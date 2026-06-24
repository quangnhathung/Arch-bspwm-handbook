# Không có âm thanh

*Áp dụng cho Lenovo LOQ 15IAX9 — Intel SST + Realtek ALC — Kernel 7.x — 25/06/2026*

## Triệu chứng (Symptoms)

- Loa trong không phát tiếng, tai nghe 3.5mm không hoạt động.
- HDMI/DisplayPort audio không có (khi cắm màn hình ngoài).
- `pactl list sinks` không hiển thị thiết bị đầu ra.
- Biểu tượng âm thanh trên polybar báo muted hoặc "Dummy Output".

## Nguyên nhân (Causes)

1. **PipeWire chưa được khởi động** — `systemctl --user status pipewire` không active.
2. **WirePlumber (session manager) chưa chạy** — PipeWire cần WirePlumber để quản lý thiết bị.
3. **SOF (Sound Open Firmware) thiếu** — Intel SST (Smart Sound Technology) trên LOQ 15IAX9 yêu cầu `sof-firmware`.
4. **ALSA card mặc định sai** — card ưu tiên là HDMI (NVIDIA) thay vì HDA Intel PCH (analog).
5. **Alsamixer muted** — Master hoặc Headphone bị mute (chữ `MM`).
6. **PipeWire components thiếu** — `pipewire-alsa` hoặc `pipewire-pulse` chưa cài.

## Chẩn đoán (Diagnosis)

```bash
# 1. PipeWire services
systemctl --user status pipewire wireplumber pipewire-pulse

# 2. ALSA cards
cat /proc/asound/cards

# 3. PulseAudio devices (qua PipeWire)
pactl list sinks short

# 4. SOF firmware
dmesg | grep -iE "sof|snd_soc_skl"

# 5. Kernel modules
lsmod | grep -iE "snd_hda_intel|snd_sof"

# 6. Kiểm tra mute
alsamixer
# F6 → chọn card → Master / Headphone có chữ MM?
```

## Khắc phục (Fix)

### Fix 1: Khởi động PipeWire + WirePlumber

```bash
systemctl --user enable --now pipewire pipewire-pulse wireplumber

# Kiểm tra lại
systemctl --user status pipewire
pactl info
```

### Fix 2: Cài đầy đủ PipeWire components

```bash
sudo pacman -S pipewire pipewire-alsa pipewire-pulse wireplumber
```

### Fix 3: Cài SOF firmware (cho Intel SST)

```bash
sudo pacman -S sof-firmware alsa-firmware
sudo reboot
```

Sau reboot, kiểm tra:
```bash
dmesg | grep sof
# Phải thấy "sof-audio-pci ... bound to ..."
```

### Fix 4: Chọn đúng ALSA card mặc định

```bash
# Xem card analog (thường là card 0 hoặc 1)
cat /proc/asound/cards

# Nếu card sai → tạo /etc/asound.conf
echo 'defaults.pcm.card 0' | sudo tee -a /etc/asound.conf
echo 'defaults.ctl.card 0' | sudo tee -a /etc/asound.conf

# Hoặc dùng pipewire config
pactl set-default-sink alsa_output.pci-0000_00_1f.3.analog-stereo
```

### Fix 5: Unmute trong alsamixer

```bash
alsamixer
# F6 → chọn "HDA Intel PCH"
# ↑ ↓ để chọn kênh
# m → mute/unmute
# Đảm bảo: Master, PCM, Headphone (không có MM)
# ↑ để tăng volume lên ~80%
```

### Fix 6: Restart audio stack

```bash
systemctl --user restart pipewire wireplumber
# Nếu không được → reboot
sudo reboot
```

## Phòng ngừa (Prevention)

1. **Cài `sof-firmware` và `alsa-firmware` ngay trong `pacstrap`.**
2. **Enable user services ngay sau khi cài:**

```bash
systemctl --user enable pipewire pipewire-pulse wireplumber
```

3. **Kiểm tra `dmesg | grep sof` sau mỗi lần cập nhật kernel.**
4. **Kiểm tra `alsamixer` sau khi cài lại alsa-utils.**
