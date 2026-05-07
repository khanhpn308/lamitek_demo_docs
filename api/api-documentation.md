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

- Trả danh sách user (admin).
- Mỗi phần tử có thêm `authorized_devices`: danh sách `{ device_id, devicename }` — các thiết bị đã phân quyền RBAC cho user đó.

### PATCH `/users/{user_id}`

- Cập nhật `status` (`active`/`deactive`).

### DELETE `/users/{user_id}`

- Xóa user.

## 4. WebSocket (Realtime Data Streaming)

### WS `/ws/global`

**URL**: `ws://localhost:8000/api/ws/global`

**Mục đích**: Realtime broadcast sensor data tới tất cả frontend clients (GlobalDashboard).

**Cách kết nối**:

```javascript
const ws = new WebSocket("ws://localhost:8000/api/ws/global");
ws.onmessage = (e) => {
  const data = JSON.parse(e.data);
  console.log("Device:", data.device_id, "Temp:", data.temperature);
};
```

**Payload nhận** (ví dụ):

```json
{
  "device_id": "101",
  "sensor_type": "temperature",
  "temperature": 28.5,
  "ts": 1714000000,
  "server_receive_ms": 1714000001234
}
```

**Ghi chú**:

- Server chỉ broadcast dữ liệu; client không cần gửi gì.
- Kết nối sẽ maintain mở cho tới khi client close hoặc connection break.
- Dữ liệu từ MQTT Subscriber qua RealtimeHub broadcast.

### WS `/ws/devices/{device_id}`

**URL**: `ws://localhost:8000/api/ws/devices/101`

**Mục đích**: Realtime data của **một thiết bị cụ thể** (Device Dashboard).

**Payload nhận**: Tương tự `/ws/global` nhưng chỉ có data của `device_id` khi request.

**Cách dùng**: Khi hiển thị dashboard chi tiết của 1 device (chart, timeline), dùng endpoint này để nhận data riêng thay vì filter từ global stream.

### WS `/ws/esp32/{device_id}`

**URL**: `ws://localhost:8000/api/ws/esp32/101`

**Mục đích**: **Bi-directional** kết nối từ thiết bị ESP32/IoT device.

**Uplink (device → server)**:

```json
{
  "device_id": "101",
  "temperature": 28.5,
  "humidity": 65,
  "timestamp_ms": 1714000012345
}
```

Server sẽ echo lại: `{"ok": true, "received": {...}}`.

**Downlink (server → device)** [Future]:
Server có thể gửi command qua REST API; framework support sẵn via `hub.send_to_esp32(device_id, msg)`.

**Binary support**:

- Device có thể gửi binary frame (ví dụ protobuf); server echo bytes count.
- Empty string → server phản hồi pong.

**Ví dụ Arduino/ESP32 code**:

```cpp
void setup() {
  webSocket.begin("server", 8000, "/api/ws/esp32/101");
}

void loop() {
  webSocket.loop();

  // Send JSON telemetry
  String payload = "{\"temperature\": 28.5, \"humidity\": 65}";
  webSocket.sendTXT(payload);
  delay(5000);
}
```

- Không được xóa chính mình.

## 4. Nhóm Devices

### GET `/devices` (admin)

- Lấy toàn bộ thiết bị.

### POST `/devices` (admin)

- Tạo mới thiết bị.
- Có thể truyền thêm `topic` để lưu topic MQTT mặc định cho thiết bị.

### GET `/devices/my`

- User thường lấy danh sách thiết bị đã được cấp quyền còn hiệu lực.

### GET `/devices/{device_id}`

- Admin xem tất cả.
- User thường chỉ xem thiết bị có quyền.
- Response bổ sung:
  - `authorized_users`: danh sách user được phân quyền RBAC (`user_id`, `username`, `fullname`, `expired_at`).
  - `user_device_asignment_id`: chỉ có giá trị thật khi caller là **admin**; user thường nhận `null` (không lộ trường legacy).
  - `topic`: topic MQTT đang lưu trên bản ghi thiết bị.

### PATCH `/devices/{device_id}` (admin)

- Cập nhật một phần thiết bị, gồm `user_device_asignment_id` (gán tài khoản legacy trên bản ghi thiết bị).
- Hỗ trợ cập nhật `topic`; backend sẽ tự đồng bộ subscribe/unsubscribe runtime theo topic mới.

### GET `/devices/topics` (admin)

- Danh sách topic MQTT đã lưu theo từng thiết bị.
- Dùng cho UI admin trang quản lý topic.

### PUT `/devices/{device_id}/topic` (admin)

- Cập nhật riêng `topic` cho một thiết bị.
- Body: `{ "topic": "devices/101/telemetry" }` hoặc `{ "topic": null }` để xóa.
- Sau khi cập nhật DB, backend tự đồng bộ subscribe/unsubscribe runtime.

### DELETE `/devices/{device_id}` (admin)

- Xóa thiết bị; xóa trước các dòng `device_authorization` liên quan, sau đó xóa `device`.

## 5. Nhóm Authorizations (admin)

### GET `/authorizations?user_id={id}` hoặc `?device_id={id}`

- Cần **đúng một** tham số `user_id` **hoặc** `device_id`.
- `user_id`: các phân quyền của user đó.
- `device_id`: các phân quyền gắn với thiết bị đó.

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

### GET `/mqtt/topics` (admin)

- Danh sách topic MQTT đang theo dõi runtime.
- Topic runtime được khôi phục từ cột `device.topic` khi backend khởi động.

### POST `/mqtt/topics/subscribe` (admin)

- Body: `{ "topic": "devices/101/telemetry", "qos": 0 }`
- Subscribe topic động trong lúc hệ thống đang chạy (không restart backend).

### POST `/mqtt/topics/unsubscribe` (admin)

- Body: `{ "topic": "devices/101/telemetry" }`
- Unsubscribe topic động trong runtime.

### GET `/mqtt/history?minutes=30&device_id=101`

- Truy vấn dữ liệu từ InfluxDB trong `minutes` phút gần nhất (mặc định 30, max 180).
- Nếu truyền `device_id`, chỉ lấy dữ liệu của thiết bị đó.

### WebSocket realtime

- `ws://<host>/ws/global`: luồng realtime cho Global Dashboard.
- `ws://<host>/ws/devices/{device_id}`: luồng realtime cho trang Device Detail.
- Payload realtime chuẩn hóa gồm: `device_id`, `sensor_type`, `temperature`, `vibration`, `voltage`, `current`, `ts`, `ts_iso`.

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

## 8. UI admin quản lý topic

- Đường dẫn frontend: `/topic-management` (admin only).
- Chức năng:
  - xem topic runtime đang subscribe,
  - cập nhật `device.topic` theo từng thiết bị,
  - đồng bộ subscribe runtime ngay sau khi lưu.
