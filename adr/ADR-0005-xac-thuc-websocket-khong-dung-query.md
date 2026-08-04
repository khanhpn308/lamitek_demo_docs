# ADR-0005: Xác thực WebSocket không truyền credential trong URL

- **Trạng thái**: Accepted
- **Ngày**: 2026-07-23
- **Phạm vi**: Frontend React, FastAPI WebSocket và thiết bị IoT

## Bối cảnh

Trình duyệt không cho JavaScript tự đặt header `Authorization` khi mở WebSocket. Cách
truyền JWT hoặc mật khẩu thiết bị bằng query string làm credential xuất hiện trong URL,
access log của backend/reverse proxy, lịch sử chẩn đoán và các hệ thống quan sát trung gian.

## Quyết định

- Frontend xác thực `/ws/global` và `/ws/devices/{device_id}` bằng danh sách
  `Sec-WebSocket-Protocol: iot-jwt, <jwt>`.
- Backend đọc JWT từ protocol thứ hai nhưng chỉ negotiate/echo protocol công khai
  `iot-jwt`; JWT không xuất hiện trong response handshake.
- Thiết bị xác thực `/ws/esp32/{device_id}` bằng
  `Sec-WebSocket-Protocol: iot-device, <device-password>` hoặc header
  `x-device-password`.
- Client không phải trình duyệt vẫn có thể dùng `Authorization: Bearer <jwt>` cho luồng
  user.
- Backend từ chối credential trong `access_token` hoặc `device_password` query string.
- Production vẫn bắt buộc TLS (`wss://`), vì subprotocol/header bảo vệ log chứ không tự
  mã hóa credential trên đường truyền.

## Phương án đã cân nhắc

- **Giữ query string và redaction log**: loại bỏ vì credential vẫn có thể bị ghi bởi
  proxy hoặc hạ tầng đứng trước ứng dụng.
- **Cấp WebSocket ticket dùng một lần**: an toàn nhưng cần thêm endpoint, state, thời
  hạn và cơ chế chống replay; chưa cần thiết cho phạm vi hiện tại.
- **Cookie phiên HttpOnly**: phù hợp cho một đợt chuyển đổi auth toàn hệ thống, nhưng
  vượt phạm vi thay đổi WebSocket và cần thiết kế lại CSRF/session.

## Hệ quả

- Mọi WebSocket client phải dùng cơ chế mới; client query cũ nhận lỗi xác thực.
- URL và access log không còn chứa JWT/mật khẩu thiết bị.
- Tên subprotocol công khai vẫn xuất hiện trong log/handshake nhưng không phải secret.
- Test contract phải bao phủ protocol hợp lệ, header fallback và query bị từ chối.
