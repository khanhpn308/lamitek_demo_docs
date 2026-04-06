# Tài Liệu Thiết Kế Giao Diện (UI/UX)

- **Mã tài liệu**: UIUX-IOT-001
- **Phiên bản**: 1.0.0
- **Ngày cập nhật**: 2026-04-06

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
- Biểu đồ realtime tổng quan theo thiết bị (trục X: thiết bị, trục Y: giá trị).
- Có đủ 4 biểu đồ: `Current (A)`, `Voltage (V)`, `Temperature (°C)`, `Vibration (mm/s)`.
- Mapping biểu đồ theo loại thiết bị:
  - Thiết bị `Temperature` chỉ xuất hiện ở biểu đồ `Temperature`.
  - Thiết bị `Power` xuất hiện ở 2 biểu đồ `Voltage` và `Current`.
  - Thiết bị `Vibration` chỉ xuất hiện ở biểu đồ `Vibration`.
- Dữ liệu hiển thị theo quyền:
  - Admin: tất cả thiết bị.
  - User: chỉ thiết bị được phân quyền/quản lý.
- Tự scale trục Y theo dữ liệu hiện tại của từng biểu đồ.
- Tự giãn cột và nhãn trục X theo số lượng thiết bị; khi thêm thiết bị mới thì tự xuất hiện trên biểu đồ.
- Ẩn nhãn tên thiết bị trên trục X (để giao diện gọn), tên thiết bị hiển thị trong tooltip khi hover vào cột.
- Mỗi biểu đồ có nút phóng to/thu nhỏ toàn màn hình để quan sát chi tiết.

### 4.5 Devices
- Danh sách thiết bị.
- Tìm kiếm theo tên/ID/vị trí.
- Admin có nút thêm thiết bị.

### 4.6 Device Detail
- Tab `Account`, `History`, `Dashboard`.
- Hiển thị thông tin định danh, lịch sử, biểu đồ chi tiết.
- `Dashboard` theo loại cảm biến:
  - `Temperature`: 1 biểu đồ miền thời gian cho nhiệt độ `°C`.
  - `Power`: 2 biểu đồ miền thời gian cho `Voltage (V)` và `Current (A)`.
  - `Vibration`: 1 biểu đồ miền thời gian cho `mm/s`.
- Mỗi biểu đồ trong tab `Dashboard` có nút phóng to/thu nhỏ toàn màn hình.

### 4.8 Add Device Modal
- Danh sách `Device Type` gồm 3 loại:
  - `Nhiệt độ (Temperature)`
  - `Công suất (Power)`
  - `Độ rung (Vibration)`

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
