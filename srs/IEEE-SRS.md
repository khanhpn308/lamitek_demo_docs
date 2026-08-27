# IEEE SRS - Đặc Tả Yêu Cầu Phần Mềm

- **Mã tài liệu**: SRS-IOT-001
- **Phiên bản**: 1.3.0
- **Ngày cập nhật**: 2026-08-07
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
- Quản lý nhóm, upload ảnh bản đồ và theo dõi thiết bị realtime trên GPS map.
- Quản lý Anchor trên bản đồ và đồng bộ cấu hình theo location xuống Gateway.
- Khôi phục mật khẩu, đổi mật khẩu.
- Giám sát health của API, DB và MQTT subscriber.

### 1.3 Định nghĩa và viết tắt
- **SRS**: Software Requirements Specification.
- **JWT**: JSON Web Token.
- **RBAC**: Role-Based Access Control.
- **API**: Application Programming Interface.
- **ADR**: Architecture Decision Record.
- **Anchor**: Thiết bị tính toán trực tiếp với node ESP32 để suy ra vị trí rồi chuyển kết quả qua Gateway.
- **ACK**: Acknowledgement, phản hồi của Gateway xác nhận đã áp dụng hoặc từ chối cấu hình.
- **Transactional outbox**: Bản ghi sự kiện được lưu cùng transaction nghiệp vụ để publish MQTT tin cậy sau commit.

### 1.4 Tài liệu tham chiếu
- `docs/api/openapi-like.yaml`
- `docs/architecture/system-architecture.md`
- `docs/srs/anchor-configuration-spec.md`
- `docs/api/anchor-configuration-api.md`
- `docs/adr/ADR-0007-phan-quyen-va-dong-bo-cau-hinh-anchor.md`
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
- WebSocket user bắt buộc JWT qua subprotocol/header, không dùng query credential.

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
- WebSocket over WS/WSS cho dữ liệu realtime.
- JSON request/response.
- JWT truyền trong header `Authorization: Bearer <token>`.
- Trình duyệt truyền JWT WebSocket bằng subprotocol `iot-jwt`; thiết bị dùng
  `iot-device` hoặc `x-device-password`. Credential trong query string bị từ chối.

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

### 4.9 SF-09: Quản lý nhóm bản đồ và lời mời
- **Mô tả**:
  - User active tạo và quản lý nhiều nhóm bản đồ của chính mình.
  - Admin xem mọi nhóm và có thể tạo nhóm cho owner khác bằng username chính xác.
  - Owner/admin đổi tên nhóm, gửi/hủy lời mời và gỡ thành viên.
  - User được mời phải accept trước khi nhóm xuất hiện trong phạm vi truy cập.
- **Ràng buộc**:
  - Tên nhóm duy nhất không phân biệt hoa/thường theo từng owner.
  - Chặn self-invite, duplicate invitation và tài khoản inactive/hết hạn.
  - Member chỉ xem; không được quản lý nhóm hoặc tự rời nhóm.
  - User thường không thấy nhóm nếu owner inactive/hết hạn.

### 4.10 SF-10: Upload, hiển thị và archive ảnh bản đồ
- **Mô tả**:
  - Owner nhóm/admin upload ảnh WebP, PNG hoặc JPG/JPEG cùng location trùng với payload gateway.
  - User chọn theo `Nhóm bản đồ → Khu vực (Map)`; hệ thống chỉ tải BLOB đang chọn.
  - Marker thiết bị hiển thị khi payload location khớp sau trim và so sánh không phân biệt hoa/thường.
  - Danh sách thiết bị hiển thị `devicename(device_id)`; marker luôn hiện `devicename`
    phía trên vị trí và ô tìm kiếm hỗ trợ cả tên lẫn ID.
  - Dashboard dùng một đồng hồ giờ địa phương chung theo định dạng `HH:mm:ss`; không
    hiển thị tọa độ hoặc timestamp riêng của từng thiết bị cho người dùng.
  - Owner/admin archive map; admin xem lịch sử metadata đã xóa.
- **Ràng buộc**:
  - Ảnh phải tĩnh; đuôi file, MIME và nội dung thực tế phải là WebP, PNG hoặc JPEG và khớp nhau.
  - Không giới hạn width/height; dung lượng phải từ 1 byte và nhỏ hơn 10 MiB.
  - Location active duy nhất không phân biệt hoa/thường.
  - Accepted member chỉ xem, không upload hoặc xóa.
  - Lịch sử không hỗ trợ tải BLOB, preview, restore hoặc purge.
  - `devicename` null/rỗng fallback thành ID; tọa độ X/Y vẫn được giữ nội bộ để định vị marker.

### 4.11 SF-11: Lifecycle group/owner và map hệ thống

- **Mô tả**:
  - Xóa group phải archive toàn bộ map active trước khi dọn membership và xóa group.
  - Xóa tài khoản owner phải archive map của các group thuộc sở hữu; membership của
    user trong group owner khác chỉ bị dọn.
  - Hệ thống seed `Floor_1`…`Floor_4` vào group `System Debug Maps` của `AD00000`.
- **Ràng buộc**:
  - Cascade chạy trong một transaction và rollback toàn bộ nếu có lỗi.
  - Non-admin không thấy group/map khi owner inactive hoặc hết hạn; admin vẫn quản trị được.
  - Seed dùng cùng validator WebP với upload và không seed lại location đã có trong
    bảng active hoặc archive.

### 4.12 SF-12: Realtime và security hardening

- **Mô tả**:
  - Frontend nhận dữ liệu realtime qua WebSocket user đã xác thực.
  - Thiết bị gửi payload qua WebSocket device đã xác thực.
  - Payload có location khớp map đang chọn làm marker xuất hiện cho user có quyền xem group.
- **Ràng buộc**:
  - JWT/mật khẩu thiết bị không được đưa vào WebSocket URL hoặc access log.
  - Backend chỉ negotiate tên subprotocol công khai, không echo credential.
  - Frontend production phải trả security header cho cả document và static asset.

### 4.13 SF-13: Cấu hình Anchor và đồng bộ Gateway

- **Trạng thái triển khai**: Phase 0–6 đã hoàn thành nền dữ liệu, API/UI, dispatcher,
  liveness, ACK/status, lifecycle/reconciliation và rollout readiness.

- **Mô tả**:
  - Admin luôn được cấu hình Anchor; user thường chỉ được cấu hình khi
    `can_config_anchor = 'yes'` và là owner của group chứa map.
  - Accepted member chỉ được xem Anchor trong group đã tham gia, không được tạo, sửa,
    kéo hoặc xóa Anchor dù có cờ `can_config_anchor`.
  - Anchor có ID nội bộ, `mac_address` chuẩn sáu octet hexadecimal và bất biến sau khi gán,
    tên, tọa độ phần trăm `x/y` và giá trị
    `z`; tạo mới mặc định tại `(50, 50, 0)`.
  - Owner/admin có thể mở cùng một giao diện chỉnh sửa từ marker trên map hoặc từ danh
    sách Anchor có tìm kiếm và phân trang phía server.
  - Sau mutation có thay đổi, backend lưu Anchor và delta cấu hình trong cùng transaction;
    dispatcher compose payload versioned riêng theo revision đã ACK của từng Gateway.
  - Gateway phản hồi ACK theo revision; backend theo dõi trạng thái tổng hợp, từng Gateway
    và `last_seen_at` để UI polling hiển thị tiến độ đồng bộ.
- **Ràng buộc**:
  - `x/y` thuộc `[0, 100]`, gốc tọa độ ở góc trái dưới; `z` được lưu nhưng chưa render trên map 2D.
  - Xóa là soft delete; `mac_address` không được tái sử dụng trong phase này.
  - Replace rỗng có nghĩa xóa toàn bộ cấu hình Anchor tại location; delta rỗng không được tạo.
  - MQTT dùng QoS 1, retained message và retry theo chính sách outbox; firmware Gateway
    nằm ngoài phạm vi nhưng phải tuân thủ contract payload/ACK đã công bố.

---

## 5. Yêu cầu phi chức năng

### 5.1 Bảo mật
- Mật khẩu lưu bcrypt hash.
- JWT secret phải cấu hình qua môi trường và không hard-code production.
- API admin-only bắt buộc kiểm tra role ở backend.
- WebSocket phải xác thực ở backend và từ chối credential trong query string.
- Production phải dùng HTTPS/WSS và trả CSP, chống MIME sniffing/clickjacking.

### 5.2 Hiệu năng
- Endpoint CRUD cơ bản phản hồi trong ngưỡng chấp nhận được ở môi trường nội bộ.
- Kết nối DB dùng pool cấu hình sẵn.
- Đồng hồ GPS phải được cô lập trong component riêng để tick mỗi giây không làm map và
  danh sách thiết bị re-render theo.

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
- RealtimeHub hỗ trợ kênh global, theo device và kết nối hai chiều cho thiết bị.

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

### 6.4 Thực thể map
- `locations_using` chứa map active, BLOB, checksum, kích thước, group, owner,
  người upload và thời điểm tạo.
- `locations_deleted` chứa bản archive cùng snapshot group/owner/người thao tác,
  lý do và thời điểm xóa; không có foreign key làm mất lịch sử.

### 6.5 Thực thể Anchor và outbox

> Schema MySQL, migration idempotent và ORM cho các thực thể trong mục này đã được triển khai ở Phase 0.

- `anchor` lưu ID nội bộ, `mac_address` duy nhất và bất biến, `hardware_id` legacy,
  tên, `x/y/z`, map/location,
  trạng thái active/inactive, audit timestamps và người tạo/cập nhật/xóa.
- Tên Anchor duy nhất không phân biệt hoa/thường trong các Anchor active của cùng map.
- Bảng revision/outbox lưu event delta/replace, target Gateway tùy chọn và trạng thái publish/retry.
- Bảng delivery/ACK lưu payload wire bất biến và kết quả áp dụng riêng cho từng Gateway của một revision.
- `user.can_config_anchor` nhận `yes/no`, mặc định `no`; admin không phụ thuộc cờ này.

---

## 7. Quy tắc nghiệp vụ

- BR-01: User `deactive` không được đăng nhập.
- BR-02: User hết hạn (`expired_at`) phải bị chuyển `deactive` theo job xử lý hiện tại.
- BR-03: User thường không được truy cập endpoint admin.
- BR-04: User thường chỉ xem thiết bị đã được gán và chưa hết hạn phân quyền.
- BR-05: Không xóa được tài khoản admin đang đăng nhập.
- BR-06: Backend phải giải mã và kiểm tra lại định dạng ảnh thực tế, không tin
  validation frontend, đuôi file hoặc MIME do client khai báo.
- BR-07: Trạng thái map được xác định bằng bảng active/archive, không dùng cột status.
- BR-08: Chỉ tải ảnh khi request hiện tại có quyền xem group của map.
- BR-09: Xóa group/owner phải archive map trước và hoàn tất atomically với việc dọn quan hệ.
- BR-10: Seed không được làm sống lại location đã tồn tại trong lịch sử archive.
- BR-11: JWT hoặc mật khẩu thiết bị không được truyền bằng WebSocket query string.
- BR-12: User thường chỉ mutation Anchor khi vừa có `can_config_anchor = 'yes'` vừa là
  owner group; accepted member luôn chỉ xem.
- BR-13: Quyền phải được kiểm tra lại ở backend cho từng request; thu hồi cờ có hiệu lực
  ngay và không xóa dữ liệu Anchor đã tạo.
- BR-14: Mỗi thay đổi thực sự phải tăng revision và ghi delta cùng transaction; PATCH no-op
  không tăng revision; publish MQTT không được làm transaction HTTP chờ broker.
- BR-15: Mỗi Gateway active có `device_type = 'gateway'` và location chuẩn hóa khớp map
  nhận payload coalesce riêng; Gateway mới nhận retained full replace để thiết lập baseline.
- BR-16: Revision mới thay thế delivery cũ chưa hoàn tất theo từng Gateway; retry cùng payload theo lịch
  `5, 15, 30, 60, 300` giây rồi mỗi 300 giây cho tới khi superseded hoặc thành công.
- BR-17: Xóa map/group/owner phải soft-delete Anchor liên quan và phát replace rỗng
  trong cùng lifecycle transaction.

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
- AC-09: User/admin tạo, xem và đổi tên group đúng phạm vi quyền.
- AC-10: Invitation chuyển đúng `pending → accepted/rejected`, hỗ trợ hủy, gỡ và re-invite.
- AC-11: Upload hợp lệ thành công; sai type/size/dimension/animation và location trùng bị chặn.
- AC-12: GPS chỉ tải BLOB đang chọn và vẫn lọc marker đúng theo payload location.
- AC-13: Archive giữ BLOB cũ, ngừng phục vụ ID cũ và cho phép upload lại location bằng ID mới.
- AC-14: Xóa group/owner archive đủ map, dọn đúng quan hệ và rollback khi có lỗi.
- AC-15: Owner inactive tạm ẩn group/map với non-admin; active lại khôi phục khả năng xem.
- AC-16: Seed lần đầu tạo đủ bốn map và restart không tạo trùng hoặc tái tạo map đã archive.
- AC-17: WebSocket user/device xác thực qua subprotocol/header, query credential bị từ chối
  và access log không chứa secret.
- AC-18: Regression backend/frontend, production build, dependency audit, Docker health
  và kiểm thử trình duyệt nhiều vai trò đều pass.
- AC-19: GPS hiển thị tên/ID và nhãn marker đúng fallback, tìm theo tên/ID, dùng một
  đồng hồ chung và không lộ tọa độ/timestamp từng thiết bị trên UI.
- AC-20: Admin luôn CRUD Anchor; owner có cờ `yes` CRUD được; owner cờ `no` và accepted
  member đều không mutation được ở cả UI và API.
- AC-21: Tạo Anchor mặc định `(50, 50, 0)`; nhập tọa độ hoặc kéo marker cập nhật cùng
  draft, Save mới gửi API và Cancel khôi phục trạng thái trước chỉnh sửa.
- AC-22: Viewer thấy marker nhưng click không mở editor; editor cho phép đổi tên, vị trí
  và xóa đối với người có quyền.
- AC-23: Danh sách cấu hình chỉ chứa Anchor caller được sửa, hỗ trợ tìm theo tên hoặc
  ID/hardware ID, phân trang server-side và chuyển đúng group/map khi chọn kết quả.
- AC-24: Mutation có thay đổi tạo revision/outbox; mỗi Gateway nhận delta coalesce chỉ chứa
  Anchor cần xử lý. Bootstrap/resync/clear map dùng full replace QoS 1 retained.
- AC-25: ACK đúng Gateway/location/revision cập nhật trạng thái delivery; UI polling 5 giây
  hiển thị tổng hợp và chi tiết từng Gateway, online theo ngưỡng mặc định 30 giây.
- AC-26: Broker/Gateway offline không làm mất cấu hình; retry đúng payload, revision mới
  coalesce/supersede theo Gateway và resync có đích tạo revision mới.
- AC-27: Xóa map/group/owner soft-delete đủ Anchor và phát replace rỗng atomically.

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
| SF-09 | `/api/map-groups`, `/api/map-group-invitations`, `MapGroupManagerDialog` |
| SF-10 | `/api/map-groups/{group_id}/maps`, `/api/maps/*`, `GPSDashboard`, `UploadMapDialog`, `DeletedMapsPanel` |
| SF-11 | `map_lifecycle.py`, `seed.py`, `DELETE /api/map-groups/{group_id}`, `DELETE /api/users/{user_id}` |
| SF-12 | `deps.py`, `websocket_routes.py`, `realtime_hub.py`, `wsUrl.js`, `GPSPage`, `nginx/prod*.conf` |
| SF-13 | `/api/anchors*`, `/api/anchor-sync*`, Anchor editor/list, transactional outbox, MQTT `anchor_config.v1`/ACK |

---

## 10. Rủi ro và tồn tại kỹ thuật

- Frontend đang có fallback mock data ở một số màn hình.
- Modal đổi password thiết bị hiện thiên về mock UI, chưa có API đổi mật khẩu thiết bị riêng.
- Firmware Gateway áp dụng `anchor_config.v1` và phát ACK chưa thuộc repository này; cần
  kiểm thử contract tích hợp trước khi rollout production.
- Outbox có thể tích lũy khi broker hoặc Gateway offline dài ngày; cần metric, cảnh báo và
  chính sách lưu trữ/vận hành trước khi go-live.
