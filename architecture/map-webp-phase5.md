# Map WebP theo nhóm — Phase 5: Regression và hardening

- **Trạng thái**: Hoàn thành
- **Ngày hoàn thành**: 2026-07-23
- **Phạm vi**: Task 10

## 1. Mục tiêu

Đóng feature Map WebP bằng regression test toàn diện, kiểm tra thực tế nhiều vai trò,
loại credential khỏi WebSocket URL, xử lý dependency advisory và xác nhận cấu hình
production.

## 2. Thay đổi hardening

- WebSocket frontend dùng `Sec-WebSocket-Protocol: iot-jwt, <jwt>`; backend chỉ echo
  tên protocol an toàn và không còn nhận JWT qua query string.
- WebSocket thiết bị dùng protocol `iot-device` hoặc header `x-device-password`; query
  password bị từ chối.
- GPS tải `/api/devices/my` cho user thường và `/api/devices` cho admin, tránh request
  `403` không cần thiết.
- Compose frontend healthcheck dùng `127.0.0.1` để không phụ thuộc cách `localhost`
  resolve trong container.
- Loại dependency trực tiếp `webflow-api` không được sử dụng; production dependency
  audit trở về 0 advisory.
- Lặp security header tại các Nginx location có `add_header Cache-Control`, vì quy tắc
  kế thừa của Nginx sẽ bỏ header cấp server khi location tự khai báo `add_header`.

## 3. Phạm vi kiểm thử

- Backend integration bao phủ API contract, quyền admin/owner/member, duplicate,
  archive/cascade/seed và upload độc hại (file giả định dạng, WebP động).
- Frontend bao phủ upload dialog, lỗi backend, group dropdown, invitation và endpoint
  danh sách thiết bị theo vai trò.
- Kiểm tra trình duyệt với hai user, một admin và ba nhóm độc lập:
  member chỉ thấy nhóm được mời và nhóm của mình; owner không thấy nhóm riêng của
  member; admin thấy toàn bộ.
- Gửi payload realtime có `location=PHASE5_GATEWAY` qua WebSocket thiết bị và xác nhận
  đúng một thiết bị xuất hiện trên map được chia sẻ.
- Xác nhận console sạch, WebSocket handshake chọn `iot-jwt`, access log không chứa
  query credential và dữ liệu kiểm thử được dọn sau khi hoàn tất.

## 4. Kết quả verification

- Backend: **61 passed**, còn một `PytestCollectionWarning` có sẵn về class exception
  `TestPayloadDecodeError`.
- Frontend: **32 passed** trên 11 test files.
- Vite production build: pass.
- `npm audit --omit=dev --audit-level=low`: **0 vulnerabilities**.
- Docker Compose: backend, frontend và Mosquitto đều healthy; HTTP frontend/backend
  trả `200`.
- Nginx production HTTP config pass `nginx -t`; response document và static asset đều
  có CSP, `X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy` và
  `Permissions-Policy`.

## 5. Quyết định liên quan

Xem [ADR-0005](../adr/ADR-0005-xac-thuc-websocket-khong-dung-query.md) cho hợp đồng xác
thực WebSocket và các phương án đã cân nhắc.
