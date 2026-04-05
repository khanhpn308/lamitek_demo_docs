# Tài Liệu API (Diễn Giải Cho Dev/QA)

- **Nguồn chân lý hợp đồng API**: `docs/api/openapi-like.yaml`
- **Base URL**: `http://localhost:8000/api`
- **Cơ chế auth**: JWT Bearer

## 1. Quy ước chung

- Content-Type: `application/json`.
- Endpoint cần xác thực phải gửi `Authorization: Bearer <token>`.
- Trường lỗi ưu tiên lấy từ `detail` trong response backend.

## 2. Nhóm Auth

### POST `/auth/login`
- Mục đích: đăng nhập.
- Thành công: trả `access_token` và `user`.
- Lỗi:
  - `401`: sai username/password.
  - `403`: tài khoản bị vô hiệu hóa.

### POST `/auth/register` (admin)
- Mục đích: tạo user mới.
- Lỗi thường gặp:
  - `400`: username hoặc CCCD đã tồn tại.
  - `403`: không có quyền admin.

### POST `/auth/bootstrap`
- Chỉ dùng lần đầu khi hệ thống chưa có user.

### GET `/auth/me`
- Lấy profile người dùng hiện tại theo token.

### POST `/auth/recover-password`
- Xác minh username + CCCD.
- Thành công trả mật khẩu tạm.

### POST `/auth/change-password`
- Đổi mật khẩu tài khoản hiện tại.

## 3. Nhóm Users (admin)

### GET `/users`
- Trả danh sách user.

### PATCH `/users/{user_id}`
- Cập nhật `status` (`active`/`deactive`).

### DELETE `/users/{user_id}`
- Xóa user.
- Không được xóa chính mình.

## 4. Nhóm Devices

### GET `/devices` (admin)
- Lấy toàn bộ thiết bị.

### POST `/devices` (admin)
- Tạo mới thiết bị.

### GET `/devices/my`
- User thường lấy danh sách thiết bị đã được cấp quyền còn hiệu lực.

### GET `/devices/{device_id}`
- Admin xem tất cả.
- User thường chỉ xem thiết bị có quyền.

## 5. Nhóm Authorizations (admin)

### GET `/authorizations?user_id={id}`
- Lấy danh sách thiết bị đã cấp cho user.

### POST `/authorizations`
- Tạo phân quyền user-thiết bị.
- Trùng cặp `device_id + user_id` trả `409`.

## 6. Nhóm Health/MQTT

### GET `/health`
- Kiểm tra API sống.

### GET `/health/db`
- Kiểm tra DB.

### GET `/mqtt/status`
- Trạng thái MQTT subscriber.

### GET `/mqtt/messages?limit=50`
- Lấy message MQTT gần nhất.

## 7. Ma trận mã lỗi

- `200`: Thành công.
- `201`: Tạo mới thành công.
- `204`: Xóa thành công, không trả body.
- `400`: Dữ liệu không hợp lệ hoặc vi phạm rule nghiệp vụ.
- `401`: Chưa xác thực hoặc token sai/hết hạn.
- `403`: Không có quyền.
- `404`: Không tìm thấy.
- `409`: Dữ liệu trùng/đã tồn tại.
- `503`: Dịch vụ phụ thuộc chưa sẵn sàng.
