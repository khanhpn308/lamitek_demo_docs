# ADR-0003: Chiến lược dữ liệu realtime cho giám sát thiết bị

- **Trạng thái**: Accepted
- **Ngày**: 2026-04-04
- **Người quyết định**: 

## Bối cảnh

Dashboard yêu cầu dữ liệu gần realtime.
Hiện tại backend đã có MQTT subscriber để nhận dữ liệu từ broker.
Frontend có phần chuẩn bị luồng WebSocket tiêu thụ dữ liệu realtime.

## Quyết định ban đầu (2026-04-04)

- Tiếp tục dùng MQTT subscriber ở backend để lấy dữ liệu đầu vào.
- Giai đoạn hiện tại:
  - Cung cấp API giám sát MQTT (`/api/mqtt/status`, `/api/mqtt/messages`).
  - Cho phép frontend fallback mock data khi chưa có luồng WS backend hoàn chỉnh.
- Giai đoạn tiếp theo:
  - Bổ sung endpoint WS/SSE chính thức từ backend để push dữ liệu realtime cho dashboard.

## Lý do

- Đảm bảo hệ thống có thể vận hành từng bước, không chặn release.
- Tách vấn đề ingest MQTT và phân phối realtime đến frontend thành hai lớp rõ ràng.

## Hệ quả

### Tích cực
- Có khả năng quan sát trạng thái ingest dữ liệu sớm.
- Giảm rủi ro khi triển khai dần các phần realtime.

### Tiêu cực
- Tạm thời có chênh lệch giữa kỳ vọng realtime của frontend và năng lực backend.
- Cần roadmap cụ thể để đóng gap WS/SSE.

## Cập nhật 2026-07-04: điểm hợp lưu ingest (MQTT + WebSocket)

Backend hiện có **hai đường vào** dữ liệu thiết bị:

1. **MQTT**: `MqttSubscriber` → `decode_sensor_payload` → callback `_handle_sensor_payload` (trong `main.lifespan`).
2. **WebSocket uplink**: `/api/ws/esp32/{id}` và `/api/ws/devices/{id}` nhận frame JSON/binary từ thiết bị.

Trước đây chỉ đường MQTT ghi InfluxDB; đường WebSocket chỉ broadcast realtime → dữ liệu gửi qua WS không có lịch sử (xem `docs/bugs/2026-07-04-websocket-uplink-khong-ghi-influxdb.md`).

Quyết định: cả hai đường đi qua **một điểm hợp lưu duy nhất** — `app/core/ingest.py::ingest_sensor_payload(app, payload)` — thực hiện đồng thời:

- **Persist**: `influx.write_sensor_point()` → InfluxDB (bucket `iot_telemetry`, measurement `sensor_readings`).
- **Broadcast**: `realtime_hub.publish_from_thread()` → WS `/ws/global` + `/ws/devices/{id}`.

Sơ đồ:

```
ESP32 ──MQTT──> Mosquitto ──> MqttSubscriber ──> decode_sensor_payload ─┐
                                                                        ├─> ingest_sensor_payload ─┬─> InfluxDB (lịch sử)
ESP32 ──WebSocket──> ws_esp32 / ws_device ──────────────────────────────┘                         └─> RealtimeHub ──> WS ──> chart
```

Các chart telemetry có thể ghép 2 nguồn: **lịch sử** qua REST `GET /api/mqtt/history` (đọc Influx) + **live** qua WebSocket. Riêng GPS Dashboard áp dụng quyết định live-only bên dưới.

## Cập nhật 2026-07-17: GPS Dashboard live-only qua WebSocket

### Bối cảnh

GPS Dashboard trước đây polling `GET /api/mqtt/history?minutes=60` mỗi 15 giây. Gateway phát tọa độ khoảng mỗi 0,5 giây nên polling tạo độ trễ nhìn thấy được, thực hiện nhiều truy vấn InfluxDB không cần thiết và không phản ánh đúng nhãn “Live Tracking”.

### Quyết định

- `GPSPage` chỉ dùng REST `/api/devices` để tải danh mục và metadata thiết bị.
- Vị trí GPS mới đến trực tiếp từ backend qua WebSocket `/ws/global`; frontend chỉ nhận payload có kiểu `gps` hoặc payload có đủ `x`/`y` khi không có type.
- Không gọi `/api/mqtt/history` trong luồng GPS Tracking. InfluxDB vẫn là kho lưu lịch sử và vẫn phục vụ các màn hình/chart cần dữ liệu quá khứ.
- Client WebSocket dùng JWT theo cơ chế chung trong `wsUrl.js`, cập nhật state ngay khi nhận frame và tự kết nối lại sau khi mất kết nối.
- Dữ liệu live và REST catalog được merge theo `device_id`; tọa độ đã nhận không bị ghi đè khi catalog tải xong muộn hơn.

Luồng GPS hiện tại:

```text
Gateway ──MQTT hoặc WS uplink──> ingest_sensor_payload
                                      ├──> InfluxDB (lưu lịch sử)
                                      └──> RealtimeHub ──/ws/global──> GPSPage ──> MapViewer

MySQL ──GET /api/devices────────────────────────────────> GPSPage (metadata thiết bị)
```

### Quy ước bản đồ

- Floorplan và overlay marker dùng chung một frame, rộng tối đa 800px và scale theo tỷ lệ ảnh thật để vừa cả chiều rộng lẫn chiều cao dashboard.
- Không dùng scrollbar nội bộ để xem phần còn lại của floorplan; toàn bộ ảnh phải hiện trong vùng map.
- Tọa độ chuẩn nằm trong miền `0..100`.
- Gốc `(0,0)` ở góc dưới bên trái; X tăng sang phải và Y tăng lên trên.
- DOM/CSS có trục dọc tăng xuống dưới, vì vậy phép chiếu là `left = x%`, `top = (100 - y)%`.

### Lý do

- Độ trễ hiển thị bám theo nhịp dữ liệu backend thay vì chu kỳ polling.
- Tách rõ mục đích: WebSocket cho trạng thái hiện tại, InfluxDB cho lịch sử.
- Một frame duy nhất cho ảnh và overlay loại bỏ sai lệch tọa độ khi floorplan bị scale theo viewport.
- Gốc dưới-trái phù hợp với hệ tọa độ Descartes mà gateway sử dụng.

### Hệ quả

#### Tích cực

- Với gateway gửi mỗi 0,5 giây, marker có thể cập nhật theo từng frame sau độ trễ mạng/backend.
- Giảm truy vấn lịch sử lặp lại từ GPS Dashboard.
- Tọa độ không thay đổi khi màn hình hoặc tỷ lệ ảnh thay đổi.

#### Đánh đổi

- Khi vừa mở trang, chưa có snapshot GPS cho đến khi backend phát gói mới; đây là chủ ý của chế độ live-only.
- `/ws/global` phải hoạt động và token phải hợp lệ; khi kết nối bị gián đoạn, giao diện giữ vị trí cuối cùng trong memory cho đến khi nhận frame tiếp theo.
