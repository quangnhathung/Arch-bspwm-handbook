# Workspaces — Desktop ảo

## Mục tiêu

Hiểu khái niệm workspace và cách sử dụng chúng trong bspwm.

## Kiến thức nền

### Workspace là gì?

Workspace (còn gọi là desktop ảo) là màn hình ảo riêng biệt.
Mỗi workspace có thể chứa một bộ cửa sổ riêng, không ảnh hưởng đến workspace khác.

```
Workspace 1       Workspace 2       Workspace 3
+-----------+     +-----------+     +-----------+
| Terminal  |     | Browser   |     | Music     |
|           |     |           |     |           |
+-----------+     +-----------+     +-----------+
```

Bạn chỉ thấy một workspace tại một thời điểm. Chuyển workspace bằng phím tắt.

### Tại sao dùng workspace?

- **Tổ chức**: Mỗi không gian cho một công việc riêng.
  - Workspace 1: Terminal + Code
  - Workspace 2: Browser (tra cứu)
  - Workspace 3: Nhạc / Chat
  - Workspace 4: Email
- **Tập trung**: Không bị phân tâm bởi các cửa sổ không liên quan.
- **Đa màn hình**: Mỗi màn hình có bộ workspace riêng.

## Cấu hình workspace

Trong `bspwmrc`:

```bash
bspc monitor -d I II III IV V VI VII VIII IX
```

Dòng này tạo 9 workspace cho monitor đầu tiên (màn hình laptop), đặt tên
là I, II, III, ..., IX.

Có thể dùng tên khác:

```bash
bspc monitor -d term code web chat music files mail sys games
```

Hoặc workspace hệ thống:

```bash
bspc monitor -d 1 2 3 4 5 6 7 8 9
```

## Thao tác với workspace

### Chuyển workspace

| Phím | Chức năng |
|---|---|
| `Super + 1` | Chuyển đến workspace 1 |
| `Super + 2` | Chuyển đến workspace 2 |
| ... | ... |
| `Super + 9` | Chuyển đến workspace 9 |
| `Super + Tab` | Quay lại workspace trước đó |

### Di chuyển cửa sổ đến workspace khác

| Phím | Chức năng |
|---|---|
| `Super + Shift + 1` | Gửi cửa sổ hiện tại đến workspace 1 và chuyển đến đó |
| `Super + Shift + 2` | Gửi cửa sổ hiện tại đến workspace 2 |
| ... | ... |
| `Super + Shift + 9` | Gửi cửa sổ hiện tại đến workspace 9 |

### Lệnh CLI

```bash
# Chuyển đến workspace
bspc desktop -f 5

# Di chuyển cửa sổ đến workspace
bspc node -d 5

# Di chuyển và chuyển đến
bspc node -d 5 --follow
```

## Trạng thái workspace

Polybar hiển thị trạng thái workspace:

| Trạng thái | Hiển thị | Ý nghĩa |
|---|---|---|
| Focused | Màu xanh | Workspace đang dùng |
| Unfocused | Màu xám | Workspace khác có cửa sổ |
| Empty | Không hiện | Workspace trống |
| Urgent | Màu đỏ | Ứng dụng cần chú ý |
| Occupied | Màu nhạt | Có cửa sổ nhưng không focus |

## Gán ứng dụng vào workspace

Trong `bspwmrc`:

```bash
bspc rule -a firefox desktop='^2'
bspc rule -a Alacritty desktop='^1'
bspc rule -a thunderbird desktop='^5'
```

- `desktop='^2'`: Mở trên workspace 2.
- `^` có nghĩa là "workspace số 2 của monitor hiện tại".

## Multi-monitor

Mỗi monitor có bộ workspace riêng.

```
Monitor eDP-1 (laptop)
  Workspace: I II III IV V
Monitor HDMI-A-1 (ngoài)
  Workspace: VI VII VIII IX
```

Chuyển workspace qua lại:

```bash
# Chuyển workspace giữa các màn hình
bspc node -m next
```

Xem thêm bài multi-monitor.md.

## Best practices

1. **Dành workspace cố định cho ứng dụng thường dùng**:
   - WS 1: Terminal
   - WS 2: Trình duyệt
   - WS 3: Chat (Telegram, Discord)
   - WS 4: File manager

2. **Dùng workspace cuối** cho ứng dụng tạm thời (WS 9).

3. **Không nhồi nhét quá nhiều cửa sổ vào một workspace**.
   Nếu cần nhiều cửa sổ, chia ra nhiều workspace.

4. **Dùng `Super + Tab` để quay lại workspace trước** khi cần
   so sánh nhanh.

## Troubleshooting

### Workspace không hiển thị trên Polybar

- Kiểm tra module `bspwm` trong config Polybar.
- Restart Polybar: `killall polybar; polybar main &`.

### Không chuyển workspace được

- Kiểm tra sxhkd có chạy không.
- Kiểm tra phím tắt trong sxhkdrc.

## Tổng kết

- 9 workspace mặc định, có thể đặt tên tùy ý.
- Chuyển bằng `Super + 1-9`.
- Di chuyển cửa sổ bằng `Super + Shift + 1-9`.
- Mỗi monitor có bộ workspace riêng.
- Polybar hiển thị trạng thái workspace.
- Có thể gán ứng dụng vào workspace cụ thể.
