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

- Admin xóa user khác; không cho xóa chính tài khoản đang đăng nhập.
- Nếu user là owner nhóm, backend archive toàn bộ map active với lý do `owner_deleted`,
  dọn membership/invitation, xóa nhóm rồi mới xóa user trong cùng transaction.
- Membership của user trong nhóm do người khác sở hữu chỉ bị dọn; group và map của owner khác được giữ nguyên.
- Trả `409` nếu lifecycle không thể hoàn tất an toàn; transaction được rollback.

## 4. WebSocket (Realtime Data Streaming)

### WS `/ws/global`

**URL**: `ws://localhost:8000/api/ws/global`

**Mục đích**: Realtime broadcast sensor data tới tất cả frontend clients (GlobalDashboard).

**Cách kết nối**:

```javascript
const token = localStorage.getItem("iot_token");
const ws = new WebSocket(
  "ws://localhost:8000/api/ws/global",
  ["iot-jwt", token],
);
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
- `/ws/global` và `/ws/devices/{device_id}` bắt buộc xác thực user. Trình duyệt gửi
  `Sec-WebSocket-Protocol: iot-jwt, <jwt>`; backend chỉ echo `iot-jwt`.
- Client không phải trình duyệt có thể dùng `Authorization: Bearer <jwt>`.
- Không truyền JWT bằng `?access_token=...`; backend từ chối query credential để
  tránh secret xuất hiện trong URL/access log.

### WS `/ws/devices/{device_id}`

**URL**: `ws://localhost:8000/api/ws/devices/101`

**Mục đích**: Realtime data của **một thiết bị cụ thể** (Device Dashboard).

**Payload nhận**: Tương tự `/ws/global` nhưng chỉ có data của `device_id` khi request.

**Cách dùng**: Khi hiển thị dashboard chi tiết của 1 device (chart, timeline), dùng endpoint này để nhận data riêng thay vì filter từ global stream.

### WS `/ws/esp32/{device_id}`

**URL**: `ws://localhost:8000/api/ws/esp32/101`

**Mục đích**: **Bi-directional** kết nối từ thiết bị ESP32/IoT device.

**Xác thực**:

- Dùng `Sec-WebSocket-Protocol: iot-device, <device-password>` hoặc header
  `x-device-password: <device-password>`.
- Không truyền mật khẩu bằng `?device_password=...`; backend từ chối query credential.

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
  // Cấu hình header x-device-password hoặc subprotocol iot-device theo thư viện
  // WebSocket đang dùng trước khi gọi begin().
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
- Có thể truyền thêm `topic` để lưu topic MQTT nhận mặc định cho thiết bị và `publish_topic` để lưu topic MQTT gửi/echo về thiết bị.

### GET `/devices/my`

- User thường lấy danh sách thiết bị đã được cấp quyền còn hiệu lực.

### GET `/devices/{device_id}`

- Admin xem tất cả.
- User thường chỉ xem thiết bị có quyền.
- Response bổ sung:
  - `authorized_users`: danh sách user được phân quyền RBAC (`user_id`, `username`, `fullname`, `expired_at`).
  - `user_device_asignment_id`: chỉ có giá trị thật khi caller là **admin**; user thường nhận `null` (không lộ trường legacy).
  - `topic`: topic MQTT nhận đang lưu trên bản ghi thiết bị.
  - `publish_topic`: topic MQTT gửi đang lưu trên bản ghi thiết bị.

### PATCH `/devices/{device_id}` (admin)

- Cập nhật một phần thiết bị, gồm `user_device_asignment_id` (gán tài khoản legacy trên bản ghi thiết bị).
- Hỗ trợ cập nhật `topic` và `publish_topic`; backend sẽ tự đồng bộ subscribe/unsubscribe runtime theo `topic` mới.

### GET `/devices/topics` (admin)

- Danh sách topic MQTT đã lưu theo từng thiết bị, gồm `topic` (nhận) và `publish_topic` (gửi).
- Dùng cho UI admin trang quản lý topic.

### PUT `/devices/{device_id}/topic` (admin)

- Cập nhật riêng `topic` và `publish_topic` cho một thiết bị.
- Body ví dụ: `{ "topic": "devices/101/telemetry", "publish_topic": "devices/101/downlink" }`.
- Có thể gửi `null` cho từng field để xóa riêng giá trị đó.
- Sau khi cập nhật DB, backend tự đồng bộ subscribe/unsubscribe runtime theo `topic`.

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

## 6. Nhóm Locations & GPS

### GET `/locations`

- Trả về danh sách location active mà user hiện tại có quyền xem, đọc từ MySQL.
- Định dạng response: `{ "data": ["zone-1", "warehouse-a", ...] }`.

### GET `/floorplans/{location}.webp`

- Compatibility API legacy trả BLOB ảnh active theo location sau khi kiểm tra quyền nhóm.
- Dù path giữ hậu tố `.webp` để tương thích, `Content-Type` phản ánh định dạng thật
  đã lưu: `image/webp`, `image/png` hoặc `image/jpeg`.
- Trả `404` khi location không tồn tại hoặc user không có quyền.
- Header bảo mật: `X-Content-Type-Options: nosniff`,
  `Cache-Control: private, no-store`.

## 7. Nhóm Map Groups và lời mời

Tất cả endpoint trong nhóm này yêu cầu JWT Bearer.

### GET `/map-groups`

- Admin nhận mọi group.
- User active chỉ nhận group mình sở hữu hoặc có membership `accepted`.
- Group có owner inactive/hết hạn bị ẩn với non-admin.
- Response có `access_role` (`admin/owner/member`) và `can_manage`.

### POST `/map-groups`

- User tạo group cho chính mình với body `{ "name": "Factory A" }`.
- Admin có thể thêm `owner_username`; username phải khớp chính xác.
- Tên được trim, dài tối đa 100 và duy nhất không phân biệt hoa/thường theo owner.
- Trả `409` khi trùng tên, `404` khi không tìm thấy owner.

### PATCH `/map-groups/{group_id}`

- Owner active hoặc admin đổi tên bằng `{ "name": "Factory B" }`.
- Group ngoài quyền trả `404` để hạn chế IDOR.

### DELETE `/map-groups/{group_id}`

- Owner active hoặc admin xóa group sau bước xác nhận.
- Backend khóa group và các map active, archive từng map với lý do `group_deleted`,
  dọn membership/invitation rồi hard-delete group trong cùng transaction.
- Trả `409` nếu archive/cascade không thể hoàn tất; toàn bộ transaction được rollback.

### GET `/map-groups/{group_id}/members`

- Owner/admin xem toàn bộ membership với trạng thái `pending/accepted/rejected`.

### POST `/map-groups/{group_id}/invitations`

- Owner/admin mời bằng `{ "username": "MemberExact" }`.
- Username phải khớp chính xác; chặn self-invite, duplicate và tài khoản inactive.
- Lời mời đã reject có thể được mời lại và trở về `pending`.
- Giới hạn 100 request/giờ theo JWT user.

### DELETE `/map-groups/{group_id}/members/{user_id}`

- Owner/admin hủy lời mời pending hoặc gỡ membership accepted/rejected.
- Member không có endpoint tự rời nhóm.

### GET `/map-group-invitations`

- Trả lời mời `pending` của chính user đang đăng nhập.

### PATCH `/map-group-invitations/{group_id}`

- Body `{ "status": "accepted" }` hoặc `{ "status": "rejected" }`.
- Chỉ user được mời mới phản hồi được; phản hồi lặp lại trả `409`.
- Accept bị chặn nếu owner của group không còn active.

## 8. Nhóm Maps và archive

Tất cả endpoint yêu cầu JWT Bearer.

### GET `/map-groups/{group_id}/maps`

- Admin, owner và accepted member xem metadata map active của nhóm.
- Response không chứa `image_data`; dùng `image_url` để tải đúng BLOB đang chọn.
- Group không tồn tại hoặc ngoài quyền trả `404`.

### POST `/map-groups/{group_id}/maps`

- Owner/admin gửi multipart gồm `location` và `file`.
- File phải là ảnh tĩnh WebP, PNG hoặc JPG/JPEG; đuôi file, MIME và nội dung giải
  mã thực tế phải khớp nhau.
- Không giới hạn width/height theo nghiệp vụ; file phải có dung lượng từ 1 byte
  và **nhỏ hơn 10 MiB**.
- Location được trim, tối đa 255 ký tự và duy nhất không phân biệt hoa/thường.
- Lỗi: `409` trùng location, `413` quá dung lượng, `415` sai loại file,
  `422` sai nội dung/định dạng thực tế/ảnh động, `429` vượt 30 upload/giờ/user.

### GET `/maps/{map_id}/image`

- Kiểm tra quyền nhóm cho từng request rồi trả BLOB với `Content-Type` đã được
  backend xác thực (`image/webp`, `image/png` hoặc `image/jpeg`).
- Trả `404` khi map không tồn tại, đã archive hoặc user không có quyền.
- Dùng `nosniff` và cache `private, no-store`.

### DELETE `/maps/{map_id}`

- Owner nhóm/admin archive map bằng transaction copy-before-delete.
- Accepted member hoặc lookup ngoài quyền nhận `404`.
- Sau archive, location có thể upload lại và nhận ID mới.

### GET `/admin/deleted-maps?limit=50&offset=0`

- Chỉ admin; trả metadata archive có phân trang, tuyệt đối không trả BLOB.
- `limit` từ 1–100, `offset >= 0`; không có restore, preview hoặc purge.

## 9. Nhóm Health/MQTT

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

### POST `/test/send` (admin)

- Dùng để gửi gói test MQTT dạng text giống WebSocket, không còn dùng payload protobuf kèm timestamp delay như trước.
- Body: `{ "protocol": "mqtt", "gateway_id": "tempt-01", "node_id": "node_01", "message": "PING|hello" }`
- Backend publish payload UTF-8 trực tiếp lên topic downlink cấu hình cho test.
- Nếu payload bắt đầu bằng `PING|`, subscriber MQTT sẽ echo lại payload tương tự về topic gửi của thiết bị nếu có, hoặc topic nhận nếu không có `publish_topic`.

### WebSocket realtime

- `ws://<host>/ws/global`: luồng realtime cho Global Dashboard.
- `ws://<host>/ws/devices/{device_id}`: luồng realtime cho trang Device Detail.
- Hai endpoint frontend dùng subprotocol `["iot-jwt", token]`; URL không chứa token.
- Payload realtime chuẩn hóa gồm: `device_id`, `sensor_type`, `temperature`, `vibration`, `voltage`, `current`, `ts`, `ts_iso`.

## 9. Ma trận mã lỗi

- `200`: Thành công.
- `201`: Tạo mới thành công.
- `204`: Xóa thành công, không trả body.
- `400`: Dữ liệu không hợp lệ hoặc vi phạm rule nghiệp vụ.
- `401`: Chưa xác thực hoặc token sai/hết hạn.
- `403`: Không có quyền.
- `404`: Không tìm thấy.
- `409`: Dữ liệu trùng/đã tồn tại.
- `429`: Vượt giới hạn request.
- `503`: Dịch vụ phụ thuộc chưa sẵn sàng.

## 10. UI admin quản lý topic

- Đường dẫn frontend: `/topic-management` (admin only).
- Chức năng:
  - xem topic runtime đang subscribe,
  - cập nhật `device.topic` và `device.publish_topic` theo từng thiết bị,
  - đồng bộ subscribe runtime ngay sau khi lưu.
