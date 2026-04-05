# ADR-0002: Chiến lược xác thực JWT và phân quyền RBAC

- **Trạng thái**: Draff
- **Ngày**: 2026-04-04
- **Người quyết định**: Nhóm kỹ thuật dự án

## Bối cảnh

Hệ thống có nhóm người dùng admin và user thường.
Cần cơ chế bảo vệ API rõ ràng, dễ kiểm thử, dễ mở rộng.

## Quyết định

- Sử dụng JWT Bearer cho xác thực API.
- Phân quyền theo vai trò (`admin`, `user`) ở backend.
- Sử dụng dependency `get_current_user` và `require_admin` để chuẩn hóa kiểm tra quyền.

## Lý do

- JWT phù hợp cho frontend SPA.
- RBAC đơn giản, đáp ứng yêu cầu hiện tại.
- Kiểm tra quyền ở backend tránh rủi ro chỉ kiểm tra ở giao diện.

## Hệ quả

### Tích cực
- Dễ triển khai và tích hợp frontend.
- Quy tắc quyền rõ ràng cho endpoint admin-only.
- Dễ audit endpoint nào yêu cầu quyền gì.

### Tiêu cực
- Cần cơ chế quản trị vòng đời token (hết hạn, refresh nếu cần trong tương lai).
- Cần quản lý chặt JWT secret trong vận hành production.
