# Window Management — Quản lý cửa sổ

Ngày: 25/06/2026

## Các khái niệm cơ bản

### Node

Node là đơn vị cơ bản trong cấu trúc cây (tree) của bspwm. Mỗi node chứa một cửa sổ. Node có thể được chia thành hai node con (split).

### Container

Container là node không chứa cửa sổ, chỉ chứa node khác. Khi một node bị chia, nó trở thành container.

### Tree (cấu trúc cây)

```
Root (màn hình)
├── Node A (cửa sổ 1)
└── Container (chia dọc)
    ├── Node B (cửa sổ 2)
    └── Node C (cửa sổ 3)
```

bspwm dùng BSP (Binary Space Partition) — mỗi container chia không gian làm hai phần, liên tục đệ quy.

---

## Focus

Di chuyển con trỏ chủ động đến cửa sổ lân cận:

| Phím | Hướng | bspc |
|------|-------|------|
| `Super + h` | Sang trái (west) | `bspc node -f west` |
| `Super + j` | Xuống dưới (south) | `bspc node -f south` |
| `Super + k` | Lên trên (north) | `bspc node -f north` |
| `Super + l` | Sang phải (east) | `bspc node -f east` |

```bash
# Focus cửa sổ cuối cùng
bspc node -f last

# Focus cửa sổ trước đó (quay lại)
bspc node -f prev.local

# Focus theo chiều kim đồng hồ
bspc node -f next.local
```

---

## Move / Swap

Đẩy cửa sổ hiện tại đến vị trí của cửa sổ lân cận (swap):

| Phím | Hướng | bspc |
|------|-------|------|
| `Super + Shift + h` | Sang trái | `bspc node -s west` |
| `Super + Shift + j` | Xuống dưới | `bspc node -s south` |
| `Super + Shift + k` | Lên trên | `bspc node -s north` |
| `Super + Shift + l` | Sang phải | `bspc node -s east` |

```bash
# Di chuyển sang monitor khác
bspc node -m next

# Di chuyển đến workspace khác
bspc node -d 5
```

---

## Close vs Kill

| Phím | Lệnh | Cơ chế |
|------|------|--------|
| `Super + q` | `bspc node -c` | Gửi tín hiệu đóng (SIGTERM). Ứng dụng có thể hỏi "Save?" |
| `Super + Shift + q` | `bspc node -k` | Kill force (SIGKILL). Mất dữ liệu chưa save. |

Dùng `Super + q` cho hầu hết trường hợp. Chỉ dùng `Super + Shift + q` khi ứng dụng bị treo.

---

## Node states

Mỗi cửa sổ có thể ở một trong các state sau:

| State | Mô tả | Phím |
|-------|-------|------|
| **tiled** | Mặc định. Tự động sắp xếp trong lưới BSP. | `Super + t` (toggle) |
| **floating** | Tự do, có thể kéo thả bằng chuột. | `Super + t` (toggle) |
| **fullscreen** | Chiếm toàn màn hình. Ẩn bar + border. | `Super + f` (toggle) |
| **pseudo_tiled** | Giống tiled nhưng giữ kích thước gốc của cửa sổ. | `Super + Shift + t` |

```bash
# Toggle tiled/floating
bspc node -t tiled  # hoặc
bspc node -t floating

# Set fullscreen
bspc node -t fullscreen

# Set pseudo_tiled
bspc node -t pseudo_tiled
```

### Tiled (mặc định)

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

Cửa sổ floating nằm trên layer riêng, có thể di chuyển và resize tự do (chuột hoặc bspc). Dùng cho dialog, popup, Rofi, Polybar.

```bash
# Kéo thả bằng chuột: giữ Super và drag
```

### Fullscreen

Chiếm toàn bộ màn hình, ẩn border và bar. Dùng cho video, game, trình chiếu.

---

## Preselect (split direction)

Preselect cho phép chọn hướng chia **trước khi** mở cửa sổ mới.

| Phím | Hướng |
|------|-------|
| `Super + Ctrl + h` | Chia trái |
| `Super + Ctrl + j` | Chia dưới |
| `Super + Ctrl + k` | Chia trên |
| `Super + Ctrl + l` | Chia phải |
| `Super + Ctrl + space` | Cancel preselect |

Sau khi preselect, một đường màu xuất hiện ở cạnh được chọn. Khi mở cửa sổ mới, nó sẽ xuất hiện ở phía đó.

---

## Split ratio

Tỉ lệ chia mặc định: 50/50 (`split_ratio = 0.50`).

```bash
# Cấu hình mặc định
bspc config split_ratio 0.50

# Điều chỉnh trong lúc chạy
bspc node -i  # tăng kích thước node hiện tại
bspc node -o  # giảm kích thước node hiện tại
```

---

## Resize

| Phím | Tác dụng |
|------|----------|
| `Super + Alt + h` | Thu hẹp sang trái |
| `Super + Alt + j` | Mở rộng xuống dưới |
| `Super + Alt + k` | Mở rộng lên trên |
| `Super + Alt + l` | Mở rộng sang phải |

Resize thay đổi split ratio giữa node hiện tại và node lân cận.

---

## Flags

### Sticky

Cửa sổ luôn hiện trên tất cả workspace.

```bash
bspc node -g sticky
# Kết hợp với floating:
bspc node -t floating; bspc node -g sticky
```

Dùng cho: đồng hồ, nhạc, chat nhỏ.

### Locked

Ngăn thao tác với cửa sổ (focus, close, move).

```bash
bspc node -g locked
```

### Private

Ẩn cửa sổ khỏi window switcher (Rofi window).

```bash
bspc node -g private
```

### Above

Cửa sổ luôn ở trên cùng (always on top).

```bash
bspc node -g above
```

---

## Các lệnh CLI hữu ích

```bash
# Đóng / Kill
bspc node -c                    # close
bspc node -k                    # kill

# State
bspc node -t floating           # set floating
bspc node -t fullscreen         # set fullscreen
bspc node -t tiled              # set tiled

# Di chuyển
bspc node -s east               # swap với cửa sổ bên phải
bspc node -m next               # chuyển sang monitor kế
bspc node -d 5                  # gửi đến workspace 5
bspc node -d 5 --follow         # gửi và follow

# Flag
bspc node -g sticky             # toggle sticky
bspc node -g locked             # toggle locked
bspc node -g private            # toggle private
bspc node -g above              # toggle above

# Resize
bspc node -i                    # expand
bspc node -o                    # shrink
```

---

## Best practices

1. **Giữ 2-4 cửa sổ mỗi workspace.** Quá nhiều → dùng monocle hoặc chia workspace.
2. **Dùng floating cho dialog và popup.** bspwm tự động làm điều này với rule.
3. **Dùng fullscreen cho video, game, trình chiếu.**
4. **Preselect trước khi mở cửa sổ mới** để kiểm soát layout chính xác.
5. **Dùng sticky cho ứng dụng cần theo dõi liên tục** (đồng hồ, nhạc).

---

## Troubleshooting

### Cửa sổ không tile được

- Kiểm tra state: `Super + t` để chuyển về tiled.
- Rule trong `bspwmrc` có thể set floating cho ứng dụng đó.

### Không di chuyển được cửa sổ floating

- `Super + Shift + h/j/k/l` chỉ hoạt động với tiled. Với floating, dùng chuột kéo hoặc `bspc node -m`.

### Cửa sổ bị "kẹt" sau khi chuyển workspace

```bash
bspc node -d <workspace>   # đẩy đi
bspc node -f any           # tìm và focus
```
