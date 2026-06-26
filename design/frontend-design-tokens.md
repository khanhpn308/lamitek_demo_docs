# Tiêu Chuẩn Design Token Frontend

- Mã tài liệu: FE-DESIGN-001
- Phiên bản: 2.0.0
- Ngày cập nhật: 2026-06-25
- Phạm vi: Frontend trong `app_service`

> **Thay đổi quan trọng (v2.0.0):** Ứng dụng chạy **dark theme cố định** với palette
> slate/blue. Token mặc định sinh từ `generated/webflow.css` (card `#191919`, primary
> `#1a1b1f`/`#84868b`, background `#f5f7fa`) **KHÔNG khớp** theme thực tế, nên `global.css`
> **override toàn bộ** token màu ở `:root` (xem mục 4). Khi đọc giá trị token đang chạy,
> lấy theo bảng "Token override đang áp dụng" — KHÔNG phải bảng token gốc của webflow.

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

## 4. Color token override đang áp dụng (nguồn: `global.css` `:root`)

Đây là **giá trị token thực tế đang chạy** trên toàn ứng dụng. Block override đặt ở cuối
`app_service/src/styles/global.css` để thắng độ ưu tiên so với token gốc từ webflow.

| Semantic token | Value | Tham chiếu Tailwind |
|---|---|---|
| `--background` | `#0f172a` | slate-900 |
| `--foreground` | `#f8fafc` | slate-50 |
| `--card` | `#1e293b` | slate-800 |
| `--card-foreground` | `#f8fafc` | slate-50 |
| `--popover` | `#1e293b` | slate-800 |
| `--popover-foreground` | `#f8fafc` | slate-50 |
| `--primary` | `#2563eb` | blue-600 |
| `--primary-foreground` | `#ffffff` | white |
| `--secondary` | `#334155` | slate-700 |
| `--secondary-foreground` | `#f8fafc` | slate-50 |
| `--muted` | `#334155` | slate-700 |
| `--muted-foreground` | `#94a3b8` | slate-400 |
| `--accent` | `#1d4ed8` | blue-700 |
| `--accent-foreground` | `#ffffff` | white |
| `--destructive` | `#dc2626` | red-600 |
| `--border` | `#334155` | slate-700 |
| `--input` | `#334155` | slate-700 |
| `--ring` | `#3b82f6` | blue-500 |
| `--radius` | `0.75rem` | rounded-xl |

### 4.1 Màu giữ nguyên (KHÔNG token hoá)

Các màu sau mang ý nghĩa ngữ nghĩa, được giữ trực tiếp (không map vào token):

- Trạng thái: online/success = `green-500`, offline/error = `red-500`, cảnh báo = `amber-*`.
- Badge vai trò: admin = `purple-500/20`.
- Màu chuỗi biểu đồ recharts (Current `#3b82f6`, Voltage `#a855f7`, Temperature `#ef4444`,
  Vibration `#10b981`) — recharts cần giá trị hex cụ thể qua prop `fill`/`stroke`.
- Gradient nền trang Login/ForgotPassword: `from-slate-900 via-slate-800 to-slate-900`.

## 5. Token gốc từ webflow (tham chiếu — KHÔNG dùng trực tiếp)

`generated/webflow.css` định nghĩa token light/dark theo design system gốc
(vd dark: `--card #191919`, `--primary #84868b`; light: `--background #f5f7fa`).
Các giá trị này **bị override** ở mục 4 nên không phản ánh giao diện thực tế. Chỉ tham chiếu
khi đồng bộ lại với design system gốc. App hiện không bật class `.dark` — chạy dark theme
cố định qua override `:root`.

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
