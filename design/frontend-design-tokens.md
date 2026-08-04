# Tiêu Chuẩn Design Token Frontend

- Mã tài liệu: FE-DESIGN-001
- Phiên bản: 1.0.0
- Ngày cập nhật: 2026-04-05
- Phạm vi: Frontend trong `app_service`

## 1. Mục tiêu

Tài liệu này chuẩn hóa typography và color token nhằm:

- Đảm bảo giao diện nhất quán giữa các màn hình.
- Hỗ trợ kiểm soát thay đổi (audit) theo chuẩn doanh nghiệp.
- Tránh hard-code màu sắc và phông chữ trong component.

## 2. Nguồn token chính thức

- `app_service/generated/webflow.css`: token gốc từ hệ thống thiết kế.
- `app_service/generated/fonts.css`: danh sách font import từ công cụ thiết kế.
- `app_service/src/styles/global.css`: lớp ánh xạ token gốc sang token ngữ nghĩa dùng trong ứng dụng.

Nguyên tắc:

1. Không chỉnh sửa trực tiếp file trong `generated/`.
2. Khi cần thay đổi token gốc, cập nhật tại nguồn design system và sinh lại.
3. Các override theo nghiệp vụ phải đặt trong `global.css`.

## 3. Tiêu chuẩn Typography

### 3.1 Font được phê duyệt

- Body: `Roboto, sans-serif`
- Heading: `Roboto, sans-serif`
- Button: `Roboto, sans-serif`

### 3.2 Quy tắc sử dụng

- Văn bản thông thường: dùng `font-body` (hoặc font mặc định của `body`).
- Tiêu đề: dùng `font-heading`.
- Nút bấm/chức năng: dùng `font-button`.
- Không sử dụng font ngoài danh sách phê duyệt nếu chưa có xác nhận của Design/Brand.

## 4. Color token (Light theme)

| Semantic token | Value |
|---|---|
| `--background` | `#f5f7fa` |
| `--foreground` | `#ffffff` |
| `--card` | `#ffffff` |
| `--card-foreground` | `#060606` |
| `--popover` | `#eae9e9` |
| `--popover-foreground` | `#060606` |
| `--primary` | `#1a1b1f` |
| `--primary-foreground` | `#f8f8f9` |
| `--secondary` | `#e9ecfb` |
| `--secondary-foreground` | `#0d0d0f` |
| `--muted` | `#f0f0f0` |
| `--muted-foreground` | `#767273` |
| `--accent` | `#dcdcdc` |
| `--accent-foreground` | `#0d0d0d` |
| `--destructive` | `#d74843` |
| `--border` | `#e4ebf3` |
| `--input` | `#e7eef6` |
| `--ring` | `#1a1b1e` |

## 5. Color token (Dark theme)

| Semantic token | Value |
|---|---|
| `--background` | `#060606` |
| `--foreground` | `#f8f8f8` |
| `--card` | `#191919` |
| `--card-foreground` | `#f8f8f8` |
| `--popover` | `#191919` |
| `--popover-foreground` | `#f8f8f8` |
| `--primary` | `#84868b` |
| `--primary-foreground` | `#060606` |
| `--secondary` | `#171a29` |
| `--secondary-foreground` | `#f7f8fd` |
| `--muted` | `#020202` |
| `--muted-foreground` | `#8d8d8d` |
| `--accent` | `#2e2e2e` |
| `--accent-foreground` | `#f8f8f8` |
| `--destructive` | `#dc4d48` |
| `--border` | `#e4ebf31a` |
| `--input` | `#ffffff26` |
| `--ring` | `#97989c` |

## 6. Ánh xạ vào triển khai

Trong `app_service/src/styles/global.css`, các token ngữ nghĩa được ánh xạ vào utility class:

- `bg-background`, `text-foreground`
- `bg-primary`, `text-primary-foreground`
- `border-border`, `outline-ring/50`

Quy định trong code:

1. Ưu tiên dùng utility class đã ánh xạ từ token.
2. Không đặt mã HEX trực tiếp trong JSX/TSX (trừ trường hợp ngoại lệ có phê duyệt).
3. Khi thêm token mới, bắt buộc cập nhật:
   - `app_service/generated/webflow.css` (nguồn token)
   - `app_service/src/styles/global.css` (ánh xạ)
   - tài liệu này.

## 7. Kiểm soát thay đổi

- Mọi thay đổi typography/color phải qua review của FE Lead và Design Owner.
- Bắt buộc cập nhật phiên bản tài liệu khi token thay đổi.
- Ghi nhận thay đổi token trong release note của kỳ phát hành.
