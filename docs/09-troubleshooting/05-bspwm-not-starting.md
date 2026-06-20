# bspwm không khởi động

## Symptoms

- Chạy `startx` → màn hình đen, không thấy bar.
- Quay lại terminal với error message.
- Màn hình treo ở trạng thái X nhưng không có WM.

## Cause

1. **bspwmrc chưa có execute permission**.
2. **sxhkd chưa được cài**.
3. **Lỗi syntax trong bspwmrc hoặc sxhkdrc**.
4. **Thiếu chương trình gọi trong bspwmrc** (polybar, picom, v.v.).
5. **~/.xinitrc chưa đúng**.
6. **Xorg không được cài đúng**.

## Diagnosis

```bash
# Kiểm tra permission
ls -la ~/.config/bspwm/bspwmrc
# Phải có -rwxr-xr-x (quyền 755)

# Kiểm tra xinitrc
cat ~/.xinitrc

# Kiểm tra log Xorg
cat ~/.local/share/xorg/Xorg.0.log
# Hoặc
cat /var/log/Xorg.0.log

# Chạy bspwm thủ công để xem lỗi
bspwm 2>&1

# Chạy sxhkd foreground
sxhkd -t 5
```

## Fix

### Fix 1: Cấp execute permission

```bash
chmod +x ~/.config/bspwm/bspwmrc
```

### Fix 2: Cài sxhkd

```bash
sudo pacman -S sxhkd
```

### Fix 3: Sửa lỗi syntax trong bspwmrc

```bash
# Check syntax
bash -n ~/.config/bspwm/bspwmrc

# Nếu có lỗi → sửa
```

### Fix 4: Comment dòng chương trình lỗi

```bash
# Trong bspwmrc, comment từng dòng để xác định chương trình nào gây lỗi
# polybar main &
# picom --config ... &
```

### Fix 5: Sửa .xinitrc

```bash
echo "exec bspwm" > ~/.xinitrc
```

### Fix 6: Chạy từng bước thủ công

```bash
# Bước 1: Khởi động X thuần
xinit

# Nếu X chạy được, thoat (Ctrl+Alt+Backspace hoặc Ctrl+C)
# Bước 2: Chạy bspwm từ terminal (không xinit)
X &
sleep 1
DISPLAY=:0 bspwm &
DISPLAY=:0 sxhkd &
DISPLAY=:0 alacritty &

# Xem có lỗi gì không
```

## Prevention

1. **Kiểm tra permission bspwmrc** ngay sau khi tạo.
2. **Chạy `bash -n bspwmrc`** sau mỗi lần sửa.
3. **Test cấu hình trước khi reboot**:

```bash
startx
# Nếu lỗi → Ctrl+Alt+Backspace để kill X
```

4. **Luôn cài sxhkd cùng bspwm**.
5. **Kiểm tra log Xorg** khi gặp lỗi.
