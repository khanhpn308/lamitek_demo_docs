# IEEE SRS - Đặc Tả Yêu Cầu Phần Mềm

- **Mã tài liệu**: SRS-IOT-001
- **Phiên bản**: 1.0.0
- **Ngày cập nhật**: 2026-04-04
- **Hệ thống**: IoT Management System (React + FastAPI)
- **Trạng thái**: Bản chính thức nội bộ

---

## 1. Giới thiệu

### 1.1 Mục đích
Tài liệu này mô tả đầy đủ yêu cầu nghiệp vụ và kỹ thuật của hệ thống quản lý IoT, làm chuẩn tham chiếu chung cho các nhóm Product, Frontend, Backend, QA, DevOps.

### 1.2 Phạm vi
Hệ thống hỗ trợ:
- Quản lý xác thực người dùng và phiên đăng nhập.
- Phân quyền theo vai trò `admin` và `user`.
- Quản lý người dùng, thiết bị, cấp quyền thiết bị.
- Theo dõi dashboard tổng quan và dashboard chi tiết thiết bị.
- Khôi phục mật khẩu, đổi mật khẩu.
- Giám sát health của API, DB và MQTT subscriber.

### 1.3 Định nghĩa và viết tắt
- **SRS**: Software Requirements Specification.
- **JWT**: JSON Web Token.
- **RBAC**: Role-Based Access Control.
- **API**: Application Programming Interface.
- **ADR**: Architecture Decision Record.

### 1.4 Tài liệu tham chiếu
- `docs/api/openapi-like.yaml`
- `docs/architecture/system-architecture.md`
- `docs/adr/*`

---

## 2. Mô tả tổng quan

### 2.1 Bối cảnh sản phẩm
Hệ thống gồm frontend React (Vite) gọi backend FastAPI qua REST, backend kết nối MySQL và tiêu thụ dữ liệu MQTT từ broker để phục vụ giám sát.

### 2.2 Nhóm người dùng
- **Admin**
  - Quản lý user.
  - Tạo thiết bị.
  - Gán quyền user-thiết bị.
  - Xem toàn bộ thiết bị.
- **User**
  - Đăng nhập, đổi mật khẩu.
  - Xem danh sách thiết bị đã được cấp quyền.
  - Xem chi tiết thiết bị thuộc phạm vi được cấp.

### 2.3 Ràng buộc tổng quát
- Frontend chạy trên Node.js, build qua Vite.
- Backend chạy trên Python 3.12.
- CSDL MySQL bắt buộc sẵn sàng trước khi backend khởi động hoàn chỉnh.
- Cơ chế xác thực bắt buộc JWT Bearer.

### 2.4 Giả định và phụ thuộc
- MQTT broker có thể không sẵn sàng, nhưng API core vẫn cần hoạt động.
- Một số dữ liệu UI hiện có khả năng fallback mock khi API lỗi.

---

## 3. Yêu cầu giao diện bên ngoài

### 3.1 Giao diện người dùng
- Trang đăng nhập (`/login`), quên mật khẩu (`/forgot-password`).
- Trang nghiệp vụ sau đăng nhập: `home`, `dashboard`, `devices`, `device-detail`, `change-password`.
- Trang chỉ admin: `user-management`.
- Giao diện phải có trạng thái loading/error/empty cho dữ liệu từ API.

### 3.2 Giao diện phần mềm
- Frontend gọi backend qua các endpoint `/api/*`.
- Backend kết nối MySQL bằng SQLAlchemy.
- Backend kết nối MQTT broker qua `paho-mqtt`.

### 3.3 Giao diện truyền thông
- REST over HTTP/HTTPS.
- JSON request/response.
- JWT truyền trong header `Authorization: Bearer <token>`.

---

## 4. Tính năng hệ thống (System Features)

### 4.1 SF-01: Đăng nhập
- **Mô tả**: Người dùng đăng nhập bằng username/password.
- **Tác nhân chính**: Admin, User.
- **Tiền điều kiện**: Tài khoản tồn tại, chưa bị vô hiệu hóa.
- **Hậu điều kiện**: Trả JWT token và thông tin user.
- **Luồng chính**:
  1. Nhập username/password.
  2. Gửi request login.
  3. Backend xác thực mật khẩu hash.
  4. Trả token và user profile.
- **Ngoại lệ**:
  - Sai thông tin đăng nhập -> 401.
  - Tài khoản deactive -> 403.

### 4.2 SF-02: Quản lý phiên người dùng
- **Mô tả**: Frontend tải session từ token đã lưu, gọi `/auth/me`.
- **Kỳ vọng**: Token không hợp lệ phải bị xóa local, người dùng quay về login.

### 4.3 SF-03: Quản lý người dùng (Admin)
- **Mô tả**:
  - Liệt kê user.
  - Tạo user mới.
  - Cập nhật trạng thái active/deactive.
  - Xóa user (không cho xóa chính mình).
- **Ràng buộc**:
  - `username` và `cccd` không trùng.
  - `expired_at` không trước ngày hiện tại khi tạo.

### 4.4 SF-04: Quản lý thiết bị
- **Mô tả**:
  - Admin xem toàn bộ và tạo mới thiết bị.
  - User xem thiết bị theo phân quyền còn hiệu lực.
  - Xem chi tiết thiết bị theo quyền.

### 4.5 SF-05: Cấp quyền thiết bị (Admin)
- **Mô tả**:
  - Gán thiết bị cho user.
  - Thiết lập `granted_at`, `expired_at`, `granted_by`.
- **Ràng buộc**:
  - Một cặp `device_id + user_id` là duy nhất.

### 4.6 SF-06: Quên mật khẩu
- **Mô tả**: Xác minh username + CCCD, hệ thống cấp mật khẩu tạm mới.
- **Lưu ý**: Không thể trả mật khẩu cũ do lưu hash bcrypt.

### 4.7 SF-07: Đổi mật khẩu
- **Mô tả**:
  - So khớp mật khẩu hiện tại.
  - Mật khẩu mới khác mật khẩu cũ.
  - Mật khẩu mới và confirm phải khớp.

### 4.8 SF-08: Giám sát hệ thống
- **Mô tả**:
  - API health tổng quát.
  - DB health.
  - MQTT status và danh sách message gần nhất.

---

## 5. Yêu cầu phi chức năng

### 5.1 Bảo mật
- Mật khẩu lưu bcrypt hash.
- JWT secret phải cấu hình qua môi trường và không hard-code production.
- API admin-only bắt buộc kiểm tra role ở backend.

### 5.2 Hiệu năng
- Endpoint CRUD cơ bản phản hồi trong ngưỡng chấp nhận được ở môi trường nội bộ.
- Kết nối DB dùng pool cấu hình sẵn.

### 5.3 Độ tin cậy
- Backend có cơ chế đợi DB khởi động.
- Có cơ chế patch schema nhẹ cho DB cũ.
- MQTT subscriber tự reconnect khi ngắt.

### 5.4 Khả năng bảo trì
- Backend phân lớp rõ `api/core/models/schemas`.
- Frontend tách `pages/components/contexts/lib`.
- Tài liệu hóa theo SRS + OpenAPI-like + ADR.

### 5.5 Khả năng mở rộng
- Có thể mở rộng thêm service layer/repository layer mà không phá API contract hiện tại.
- Có thể bổ sung WebSocket backend cho dashboard realtime.

---

## 6. Yêu cầu dữ liệu

### 6.1 Thực thể `user`
- Khóa chính: `user_id`.
- Thuộc tính chính: `username`, `password`, `fullname`, `cccd`, `expired_at`, `status`, `role`.
- Quy tắc:
  - `cccd` đúng 12 chữ số.
  - `status` thuộc tập `active/deactive`.
  - `role` thuộc tập `admin/user`.

### 6.2 Thực thể `device`
- Khóa chính: `device_id`.
- Thuộc tính chính: `devicename`, `password`, `status`, `user_device_asignment_id`.

### 6.3 Thực thể `device_authorization`
- Khóa chính kép: (`device_id`, `user_id`).
- Thuộc tính: `granted_at`, `granted_by`, `expired_at`.

---

## 7. Quy tắc nghiệp vụ

- BR-01: User `deactive` không được đăng nhập.
- BR-02: User hết hạn (`expired_at`) phải bị chuyển `deactive` theo job xử lý hiện tại.
- BR-03: User thường không được truy cập endpoint admin.
- BR-04: User thường chỉ xem thiết bị đã được gán và chưa hết hạn phân quyền.
- BR-05: Không xóa được tài khoản admin đang đăng nhập.

---

## 8. Tiêu chí chấp nhận

- AC-01: Login thành công trả token và user profile.
- AC-02: Login sai trả lỗi đúng mã.
- AC-03: Admin tạo user hợp lệ thành công.
- AC-04: Admin đổi trạng thái user thành công.
- AC-05: Admin xóa user khác thành công.
- AC-06: User thường không truy cập được trang quản lý người dùng.
- AC-07: Endpoint health, db health hoạt động.
- AC-08: Cấp quyền thiết bị và lọc thiết bị theo quyền hoạt động đúng.

---

## 9. Truy vết yêu cầu

| Mã yêu cầu | Endpoint/Module liên quan |
|---|---|
| SF-01 | `/api/auth/login`, `AuthContext` |
| SF-03 | `/api/users`, `/api/auth/register`, `UserManagement` |
| SF-04 | `/api/devices`, `/api/devices/my`, `Devices`, `DeviceDetail` |
| SF-05 | `/api/authorizations`, `AssignDeviceModal` |
| SF-06 | `/api/auth/recover-password`, `ForgotPassword` |
| SF-07 | `/api/auth/change-password`, `ChangePassword` |
| SF-08 | `/api/health`, `/api/health/db`, `/api/mqtt/*` |

---

## 10. Rủi ro và tồn tại kỹ thuật

- Frontend đang có fallback mock data ở một số màn hình.
- Frontend có luồng WebSocket, backend hiện chưa có endpoint WS tương ứng.
- Modal đổi password thiết bị hiện thiên về mock UI, chưa có API đổi mật khẩu thiết bị riêng.
