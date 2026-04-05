# Tài Liệu Thiết Kế Giao Diện (UI/UX)

- **Mã tài liệu**: UIUX-IOT-001
- **Phiên bản**: 1.0.0
- **Ngày cập nhật**: 2026-04-04

## 1. Mục tiêu trải nghiệm

- Người dùng đăng nhập và thao tác nhanh với các chức năng cốt lõi.
- Giao diện nhất quán giữa các trang quản trị và giám sát.
- Hiển thị rõ trạng thái dữ liệu: loading, lỗi, rỗng, thành công.
- Tối ưu hành trình riêng cho `admin` và `user`.

## 2. Nguyên tắc thiết kế

- Chủ đề tối (dark theme) nhất quán.
- CTA chính dùng màu xanh dương.
- Trạng thái thành công dùng xanh lá, lỗi dùng đỏ, cảnh báo dùng vàng.
- Thành phần hành động quan trọng có xác nhận (xóa user).

## 3. Bản đồ màn hình

- Public:
  - `/login`
  - `/forgot-password`
- Private:
  - `/home`
  - `/dashboard`
  - `/devices`
  - `/devices/:deviceId`
  - `/change-password`
- Admin:
  - `/user-management`

## 4. Mô tả màn hình chính

### 4.1 Login
- Input: username, password.
- Validate tại phía client.
- Liên kết điều hướng quên mật khẩu.

### 4.2 Forgot Password
- Input: username, CCCD.
- Trả thông báo + mật khẩu tạm khi xác thực thành công.

### 4.3 Home
- Thẻ KPI tổng quan.
- Bảng cảnh báo mới nhất.

### 4.4 Global Dashboard
- Biểu đồ realtime theo thiết bị.
- Trạng thái trực tuyến (live).

### 4.5 Devices
- Danh sách thiết bị.
- Tìm kiếm theo tên/ID/vị trí.
- Admin có nút thêm thiết bị.

### 4.6 Device Detail
- Tab `Account`, `History`, `Dashboard`.
- Hiển thị thông tin định danh, lịch sử, biểu đồ chi tiết.

### 4.7 User Management (Admin)
- Danh sách user.
- Tạo mới user.
- Đổi trạng thái active/inactive.
- Xóa user có xác nhận.
- Gán thiết bị cho user.

## 5. Quy tắc tương tác

- Mọi thao tác submit phải disable nút khi đang xử lý.
- Mọi lỗi API hiển thị rõ ở khu vực form.
- Điều hướng quyền hạn:
  - User thường vào route admin -> hiển thị trang 403.

## 6. Quy chuẩn component

- Domain components:
  - `Layout`, `ProtectedRoute`, `AdminRoute`
  - `AddDeviceModal`, `AssignDeviceModal`, `ChangePasswordModal`
- Primitive UI components:
  - đặt trong `app_service/src/components/ui/`
- Quy ước callback:
  - dùng tiền tố `on*` (`onClose`, `onSuccess`, `onAdd`).

## 7. Tiêu chí nghiệm thu UI/UX

- Không có màn hình trắng khi API lỗi.
- Có thông điệp phản hồi rõ ràng cho mọi thao tác CRUD.
- Dòng chảy login -> dashboard hoàn chỉnh.
- Dòng chảy admin quản trị user và phân quyền hoạt động thông suốt.
