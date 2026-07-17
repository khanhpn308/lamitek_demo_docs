# Tài Liệu Thiết Kế Giao Diện (UI/UX)

- **Mã tài liệu**: UIUX-IOT-001
- **Phiên bản**: 1.2.0
- **Ngày cập nhật**: 2026-07-17

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
  - `/dashboard` (Telemetry)
  - `/dashboard/gps` (GPS Tracking)
  - `/devices`
  - `/devices/:deviceId`
  - `/change-password`
- Admin:
  - `/user-management`
  - `/topic-management` (Quản lý topic MQTT)

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
- **Empty state:** khi một chỉ số chưa có thiết bị nào có giá trị > 0 (chưa nhận telemetry),
  biểu đồ hiển thị thông báo "Chưa có dữ liệu" + icon thay vì trục trống. Khi đó không render
  `BarChart` (nên cũng không có cursor tooltip).
- **Cursor tooltip:** dùng nền mờ nhẹ (`rgba(148,163,184,0.12)`) thay cho ô xám đặc mặc định
  của recharts (vốn phủ kín category band gây che biểu đồ khi ít thiết bị).

### 4.4.1 GPS Tracking (`/dashboard/gps`)
- Bản đồ floorplan WebP theo khu vực (`MapViewer`), ánh xạ tọa độ `x`/`y` trong miền `0..100`.
- Dropdown chọn khu vực đồng bộ từ `/api/locations`; bộ lọc location không phân biệt hoa/thường; hỗ trợ tìm kiếm và danh sách thiết bị realtime.
- Metadata thiết bị tải một lần từ `/api/devices`; vị trí nhận trực tiếp từ WebSocket `/ws/global`, không polling InfluxDB.
- Marker cập nhật ngay khi backend broadcast GPS; WebSocket tự kết nối lại khi gián đoạn.
- Floorplan rộng tối đa 800px, giữ đúng tỷ lệ ảnh thật và tự thu nhỏ để vừa toàn bộ vùng map theo cả hai trục; không crop và không có thanh cuộn nội bộ.
- Ảnh và overlay marker dùng chung kích thước sau scale để tọa độ phần trăm không bị lệch.
- Hệ tọa độ dùng gốc dưới-trái:
  - `(0,0)`: góc dưới bên trái.
  - `(100,0)`: góc dưới bên phải.
  - `(0,100)`: góc trên bên trái.
  - `(100,100)`: góc trên bên phải.
  - X tăng từ trái sang phải; Y tăng từ dưới lên trên; phép chiếu CSS: `left = x%`, `top = (100 - y)%`.
- Khi vừa mở trang, marker xuất hiện sau gói GPS kế tiếp vì màn hình này chủ ý không preload snapshot từ InfluxDB.
- Chuyển nhanh giữa Telemetry và GPS qua menu điều hướng ở `Layout`.

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

### 4.9 Topic Management (Admin)
- Bảng gán topic nhận (subscribe) / topic gửi (publish) theo từng thiết bị.
- Hiển thị danh sách topic runtime đang subscribe.
- Lưu từng dòng → backend tự subscribe/unsubscribe; bỏ trống để xoá giá trị.

## 5. Quy tắc tương tác

- Mọi thao tác submit phải disable nút khi đang xử lý.
- Mọi lỗi API hiển thị rõ ở khu vực form.
- Điều hướng quyền hạn:
  - User thường vào route admin -> hiển thị trang 403.

## 6. Quy chuẩn component

- Domain components:
  - `Layout`, `ProtectedRoute`, `AdminRoute`
  - `AddDeviceModal`, `AssignDeviceModal`, `ChangePasswordModal` (modal dựng trên `<Dialog>`/`<AlertDialog>` shadcn)
- Component dùng chung (theo token), trong `app_service/src/components/common/`:
  - `PageHeader`, `Panel`, `StatCard`, `StatusBadge`
- Primitive UI components:
  - đặt trong `app_service/src/components/ui/`
- Quy ước callback:
  - dùng tiền tố `on*` (`onClose`, `onSuccess`, `onAdd`).

## 7. Tiêu chí nghiệm thu UI/UX

- Không có màn hình trắng khi API lỗi.
- Có thông điệp phản hồi rõ ràng cho mọi thao tác CRUD.
- Dòng chảy login -> dashboard hoàn chỉnh.
- Dòng chảy admin quản trị user và phân quyền hoạt động thông suốt.
