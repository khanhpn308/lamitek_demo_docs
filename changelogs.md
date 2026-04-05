# Changelog (docs)

## 2026-04-05

- **Backend**
  - `DELETE /api/devices/{device_id}` (admin): xóa thiết bị và bản ghi `device_authorization` liên quan.
  - `GET /api/devices/{device_id}`: thêm `authorized_users` (RBAC); `user_device_asignment_id` chỉ trả cho admin.
  - `PATCH /api/devices/{device_id}`: hỗ trợ cập nhật `user_device_asignment_id`.
  - `GET /api/users`: mỗi user có `authorized_devices` (thiết bị đã phân quyền).
  - `GET /api/authorizations`: tham số `user_id` hoặc `device_id` (đúng một trong hai).
- **Frontend**
  - Trang Devices: admin có nút xóa thiết bị (xác nhận bằng `OK`).
  - Quản lý người dùng: mỗi thẻ user hiển thị danh sách thiết bị được phân quyền.
  - Chi tiết thiết bị: danh sách user được phân quyền; khối chỉnh sửa `user_device_asignment_id` chỉ admin.

Tài liệu API: `docs/api/openapi-like.yaml`, `docs/api/api-documentation.md`.
