# bspwm không khởi động

*Áp dụng cho Lenovo LOQ 15IAX9 — bspwm + sxhkd — Kernel 7.x — 25/06/2026*

## Triệu chứng (Symptoms)

- `startx` → màn hình đen, không thấy polybar, không thấy cursor.
- Xorg mở ra rồi tắt ngay, quay về terminal với error message.
- Màn hình treo ở trạng thái X server nhưng không có window manager.
- `ps aux | grep bspwm` không thấy process.

## Nguyên nhân (Causes)

1. **`bspwmrc` không có execute permission** (thiếu `chmod + x`).
2. **sxhkd chưa được cài** — không có keybinding → không làm gì được.
3. **Lỗi cú pháp trong `bspwmrc` hoặc `sxhkdrc`** — bash syntax error.
4. **Chương trình trong autostart bị lỗi** — polybar, picom, dunst crash làm hỏng cả session.
5. **`~/.xinitrc` sai** — không có `exec bspwm` hoặc gọi sai.
6. **Xorg không được cài đầy đủ** — thiếu `xorg-xinit`, `xorg-server`.

## Chẩn đoán (Diagnosis)

```bash
# 1. Permission
ls -la ~/.config/bspwm/bspwmrc
# Phải có -rwxr-xr-x (755)

# 2. Kiểm tra xinitrc
cat ~/.xinitrc

# 3. Xorg log
cat ~/.local/share/xorg/Xorg.0.log 2>/dev/null
# Hoặc
cat /var/log/Xorg.0.log | grep -iE "error|fail"

# 4. Chạy bspwm thủ công
bspwm 2>&1

# 5. Chạy sxhkd thủ công (foreground 5s)
sxhkd -t 5

# 6. Kiểm tra syntax
bash -n ~/.config/bspwm/bspwmrc
bash -n ~/.config/sxhkd/sxhkdrc
```

## Khắc phục (Fix)

### Fix 1: Cấp execute permission

```bash
chmod +x ~/.config/bspwm/bspwmrc
ls -la ~/.config/bspwm/bspwmrc
```

### Fix 2: Cài sxhkd (nếu chưa có)

```bash
sudo pacman -S sxhkd

# Kiểm tra
which sxhkd
```

### Fix 3: Sửa lỗi syntax

```bash
# Bắt lỗi bash
bash -n ~/.config/bspwm/bspwmrc
# Nếu có lỗi → sửa dòng báo lỗi

# Kiểm tra sxhkdrc
bash -n ~/.config/sxhkd/sxhkdrc 2>&1 | head
```

### Fix 4: Comment autostart để cô lập lỗi

Trong `bspwmrc`, comment từng dòng một:

```bash
# picom --config ~/.config/picom/picom.conf &
# polybar main &
# dunst &
```

Nếu thấy polybar lỗi → kiểm tra polybar config:
```bash
polybar main
```

### Fix 5: Sửa .xinitrc

```bash
echo 'exec bspwm' > ~/.xinitrc
cat ~/.xinitrc
```

> **Lưu ý:** `xorg-init` không tồn tại — gói đúng là `xorg-xinit`.

### Fix 6: Debug từng bước bằng tay

```bash
# Bước 1: Chạy X server thuần
xinit
# Nếu X chạy → thấy màn hình xám, chuột chữ X
# Thoát: Ctrl+Alt+Backspace hoặc Ctrl+C

# Bước 2: Chạy bspwm + sxhkd từ TTY
startx ~/.config/bspwm/bspwmrc
```

## Phòng ngừa (Prevention)

1. **Kiểm tra permission ngay sau khi tạo bspwmrc:**

```bash
chmod +x ~/.config/bspwm/bspwmrc
```

2. **Chạy `bash -n` sau mỗi lần sửa config:**

```bash
bash -n ~/.config/bspwm/bspwmrc
bash -n ~/.config/sxhkd/sxhkdrc
```

3. **Test trước khi reboot:**

```bash
startx  # Nếu lỗi → Ctrl+Alt+Backspace
```

4. **Luôn cài `sxhkd`, `xorg-xinit`, `xorg-server` cùng lúc với bspwm.**
5. **Giữ bản backup config (git hoặc snapshot).**
