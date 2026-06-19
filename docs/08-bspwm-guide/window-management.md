# Window Management — Quản lý cửa sổ

## Mục tiêu

Hiểu và làm chủ các thao tác quản lý cửa sổ trong bspwm.

## Các khái niệm cơ bản

### Node

Node là đơn vị cơ bản trong cấu trúc cây của bspwm. Mỗi node chứa một cửa sổ.
Node có thể được chia thành hai node con (split).

### Container

Container là node chứa node khác (không phải cửa sổ). Khi chia một node,
nó trở thành container.

### Tree (Cấu trúc cây)

```
Root (màn hình)
├── Node A (cửa sổ 1)
└── Container (chia dọc)
    ├── Node B (cửa sổ 2)
    └── Node C (cửa sổ 3)
```

## Các thao tác cơ bản

### Focus (chọn cửa sổ)

```
Super + h   → focus sang trái
Super + j   → focus xuống dưới
Super + k   → focus lên trên
Super + l   → focus sang phải
```

Focus di chuyển con trỏ chủ động đến cửa sổ lân cận theo hướng chỉ định.

### Move (di chuyển cửa sổ)

```
Super + Shift + h → đẩy cửa sổ sang trái
Super + Shift + j → đẩy cửa sổ xuống dưới
Super + Shift + k → đẩy cửa sổ lên trên
Super + Shift + l → đẩy cửa sổ sang phải
```

Di chuyển swap vị trí của cửa sổ hiện tại với cửa sổ lân cận.

### Close (đóng cửa sổ)

```
Super + q   → đóng cửa sổ (gửi tín hiệu đóng)
Super + Shift + q → kill ứng dụng (buộc dừng)
```

- `Super + q` gửi yêu cầu đóng → ứng dụng có thể hỏi "Save?".
- `Super + Shift + q` dùng SIGKILL → ứng dụng tắt ngay, mất dữ liệu chưa save.

### Fullscreen

```
Super + Shift + o → toggle fullscreen
```

Khi fullscreen, cửa sổ chiếm toàn màn hình, không thấy border, không thấy bar.

## Node states

Mỗi cửa sổ có thể ở một trong các state sau:

| State | Mô tả | Phím tắt |
|---|---|---|
| **tiled** | Mặc định, cửa sổ tự động sắp xếp | `Super + t` |
| **floating** | Cửa sổ tự do, có thể kéo thả | `Super + t` (toggle) |
| **fullscreen** | Chiếm toàn màn hình | `Super + Shift + o` |
| **pseudo_tiled** | Tiled nhưng giữ kích thước gốc | `Super + Shift + t` |

### Tiled

Mặc định. Cửa sổ tự động chia màn hình thành các ô.
Khi mở thêm cửa sổ, node hiện tại bị chia làm hai.

```
┌──────────┬──────────┐
│          │          │
│  Term 1  │  Term 2  │
│          │          │
├──────────┼──────────┤
│          │          │
│  Term 3  │  Term 4  │
│          │          │
└──────────┴──────────┘
```

### Floating

Cửa sổ không bị tile, có thể di chuyển và resize tự do.
Dùng cho: dialogs, popups, Rofi, Polybar.

Chuyển một cửa sổ từ tiled sang floating:

```bash
# Toggle tiled/floating
Super + t

# Set floating
bspc node -t floating
```

Để kéo thả cửa sổ floating:

```bash
# Giữ Super và kéo chuột
# (cần cấu hình thêm nếu muốn)
```

### Pseudo_tiled

Cửa sổ nằm trong lưới tile nhưng giữ kích thước ưa thích (như floating
nhưng vẫn ở trong layout).

### Fullscreen

Cửa sổ chiếm toàn bộ màn hình. Không thấy bar, không thấy border.

## Split (chia cửa sổ)

### Preselect

Preselect cho phép bạn chọn hướng chia trước khi mở cửa sổ mới.

```
Super + Ctrl + h → preselect split trái
Super + Ctrl + j → preselect split dưới
Super + Ctrl + k → preselect split trên
Super + Ctrl + l → preselect split phải
```

Sau khi preselect, một đường màu đỏ xuất hiện ở cạnh được chọn.
Khi mở cửa sổ mới, nó sẽ xuất hiện ở phía đó.

### Split ratio

Tỉ lệ chia mặc định là 0.5 (50/50). Có thể thay đổi:

```bash
# Thay đổi ratio cho lần chia tiếp theo
bspc node -i  # tăng kích thước node hiện tại (mở rộng)
bspc node -o  # giảm kích thước node hiện tại (thu hẹp)
```

Cấu hình mặc định trong bspwmrc:

```bash
bspc config split_ratio 0.50
```

### Cancel preselect

```
Super + Ctrl + Space → cancel preselect
```

## Resize (thay đổi kích thước)

```
Super + Alt + h → thu hẹp sang trái (-20px)
Super + Alt + j → mở rộng xuống dưới (+20px)
Super + Alt + k → mở rộng lên trên (-20px)
Super + Alt + l → mở rộng sang phải (+20px)
```

Resize thay đổi tỉ lệ giữa node hiện tại và node lân cận.

## Sticky

Sticky là trạng thái cửa sổ luôn hiện trên tất cả workspace.

```bash
bspc node -g sticky
```

Dùng cho: đồng hồ, nhạc, chat nhỏ.

## Locked

Locked ngăn không cho thao tác với cửa sổ (focus, close, move).

```bash
bspc node -g locked
```

## Private

Private ẩn cửa sổ khỏi danh sách window switcher (Rofi window).

```bash
bspc node -g private
```

## Các lệnh hữu ích

```bash
# Đóng cửa sổ
bspc node -c

# Kill ứng dụng
bspc node -k

# Fullscreen toggle
bspc node -t fullscreen

# Toggle tiled/floating
bspc node -t tiled
bspc node -t floating

# Di chuyển cửa sổ đến hướng
bspc node -s east

# Di chuyển cửa sổ đến monitor khác
bspc node -m next

# Gán cửa sổ luôn ở trên (above)
bspc node -g above

# Toggle sticky
bspc node -g sticky
```

## Best practices

1. **Giữ số lượng cửa sổ mỗi workspace vừa phải** (2-4 cửa sổ).
2. **Dùng floating cho dialog và popup** (đã có rule trong bspwmrc).
3. **Dùng fullscreen cho ứng dụng toàn màn hình** (video, game).
4. **Preselect trước khi mở cửa sổ mới** để kiểm soát layout.

## Troubleshooting

### Cửa sổ không tile được

- Kiểm tra state: `Super + t` để chuyển về tiled.
- Rule trong bspwmrc có thể set floating.

### Không di chuyển được cửa sổ floating

- Dùng Super + Shift + h/j/k/l chỉ hoạt động với tiled.
- Với floating, cần cấu hình mouse binding hoặc dùng bspc resize.

### Cửa sổ bị mất sau khi chuyển workspace

- Nếu cửa sổ bị "kẹt", dùng `bspc node -d <workspace>` để đẩy đi.

## Tổng kết

- bspwm quản lý cửa sổ qua cấu trúc cây (tree).
- Mỗi cửa sổ có state: tiled, floating, fullscreen, pseudo_tiled.
- Focus, move, resize, split, close đều qua phím tắt.
- Sticky, locked, private là các flag đặc biệt.
- Preselect cho phép kiểm soát layout trước khi mở cửa sổ.
