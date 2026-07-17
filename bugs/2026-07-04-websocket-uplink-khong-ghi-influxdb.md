# Bug Log - Dữ liệu uplink qua WebSocket không được ghi vào InfluxDB

## Thông tin chung

- Thời gian hoàn tất: 2026-07-04
- Khu vực bị ảnh hưởng: app_service (backend WebSocket routes, pipeline ingest)
- Mức độ: High (mất toàn bộ lịch sử time-series cho dữ liệu gửi qua WebSocket — GPS, và mọi sensor đi đường WS)

## Mô tả lỗi chính xác

Thiết bị/gateway gửi dữ liệu GPS lên server và:

- Gateway báo gửi thành công.
- Trang chi tiết thiết bị (`/devices/{id}`) hiển thị được toạ độ realtime (ví dụ `X: 13.00, Y: 20.25`, `Points: 2`).
- **Nhưng InfluxDB rỗng hoàn toàn**: query trong Data Explorer (`localhost:8086`) và bằng API `/api/v2/query` đều trả về 0 dòng cho `sensor_type == "gps"` (thậm chí không có bất kỳ điểm nào trong bucket `iot_telemetry`).
- Bản đồ GPS (`GPSDashboard`) báo "Không có thiết bị trong khu vực này".

Triệu chứng gây hiểu nhầm:

- Vì device page vẫn "sáng đèn", dễ tưởng dữ liệu đã vào Influx và lỗi nằm ở tầng query/hiển thị.
- `GET /api/mqtt/influx/status` trả về `enabled: true, started: true, last_error: null` → Influx service khoẻ mạnh, không có lỗi ghi.

## Nguyên nhân gốc

Hệ thống có **hai đường vào** dữ liệu, nhưng chỉ **một** đường ghi InfluxDB:

1. **Đường MQTT**: `MqttSubscriber` → callback `_handle_sensor_payload` (trong `main.lifespan`) → ghi Influx + broadcast. **Có** ghi Influx.
2. **Đường WebSocket** (`/api/ws/esp32/{id}`, `/api/ws/devices/{id}`): các handler chỉ gọi `hub.publish_from_thread(payload)` → **chỉ broadcast realtime, KHÔNG gọi `influx.write_sensor_point()`**.

Dữ liệu GPS thực tế đi **đường WebSocket**, nên:

- `hub.publish_from_thread()` broadcast qua WS → device page hiển thị realtime (giải thích "Points: 2").
- Không có lệnh ghi Influx nào được phát ra → bucket rỗng, `last_error` vẫn `null` (vì Influx service chưa từng được yêu cầu ghi từ đường này).

Docstring cũ của `ws_esp32` đã tự thú: *"Hiện tại endpoint chỉ nhận + echo; chưa integrate uplink data vào InfluxDB/DB"*.

## Dấu hiệu nhận biết

- Realtime hiển thị được nhưng lịch sử (chart 30 phút, bản đồ GPS, Data Explorer) đều rỗng.
- `/api/mqtt/influx/status`: `started: true`, `last_error: null` nhưng query trả 0 dòng.
- Bucket/org/token đúng (kiểm tra bằng `/api/v2/buckets?org=iot` thấy có `iot_telemetry`), query trả `status 200` nhưng CSV rỗng.

## Các bước chẩn đoán (step by step)

1. Xác nhận dữ liệu KHÔNG nằm trong Influx (loại trừ lỗi query/UI):
   - Chọn đúng bucket `iot_telemetry`, org `iot`, range đủ rộng (`-30d`).
   - Query trực tiếp bằng token backend qua `POST /api/v2/query?org=iot` → CSV rỗng ⇒ thực sự không có data.
2. Xác nhận Influx service khoẻ: `GET /api/mqtt/influx/status` → `started: true`, `last_error: null`.
3. Đối chiếu với device page vẫn hiển thị realtime ⇒ dữ liệu tới server nhưng đi đường **không** ghi Influx.
4. Trace code: chỉ `_handle_sensor_payload` (nối vào MQTT) gọi `write_sensor_point`; các handler WebSocket chỉ `publish_from_thread`.

## Cách khắc phục (đã áp dụng)

Tập trung hoá pipeline để **mọi nguồn** (MQTT + WebSocket) đi cùng một đường: ghi Influx + broadcast.

1. Thêm `app/core/ingest.py` với hàm dùng chung `ingest_sensor_payload(app, payload)`:
   - Ghi InfluxDB qua `app.state.influx.write_sensor_point()`.
   - Broadcast realtime qua `app.state.realtime_hub.publish_from_thread()`.
   - Mỗi nhánh bọc try/except riêng (lỗi ghi Influx không làm hỏng broadcast và ngược lại), có log rõ ràng khi thất bại.
   - Lấy `influx`/`realtime_hub` từ `app.state` để tránh vòng import giữa `main` và `websocket_routes`.
2. `main.py`: `_handle_sensor_payload` chuyển sang gọi `ingest_sensor_payload(app, payload)`.
3. `api/websocket_routes.py`: cả 4 điểm uplink (`ws_esp32` và `ws_device`, cả frame bytes lẫn JSON) đổi từ `hub.publish_from_thread(...)` → `ingest_sensor_payload(websocket.app, ...)`.
   - Endpoint `/ws/global` không nhận uplink (chỉ broadcast) nên không thay đổi.

## Xác thực sau fix

1. Rebuild + restart backend container (code trong image cũ vẫn là code cũ):
   `docker compose -f app_service/docker-compose.yml up -d --build backend`
2. Gửi lại payload GPS qua đường cũ (WebSocket).
3. Query lại Influx (range `-1h`, `sensor_type == "gps"`) → phải có dòng x/y.
4. Kiểm tra bản đồ GPS hiển thị marker và chart lịch sử có điểm.

## Lưu ý còn lại

- Payload JSON gửi thẳng qua WebSocket **không** đi qua `decode_sensor_payload`, nên:
  - Không có `ts` chuẩn hoá → Influx dùng thời điểm server nhận (chấp nhận được).
  - `sensor_type` phải đúng chữ (`write_sensor_point` tự `.lower()`).
  - Nếu muốn WS JSON được chuẩn hoá đầy đủ như MQTT, cho nhánh JSON đi qua `decode_sensor_payload` trước khi ingest (chưa làm ở lần sửa này).
- Frame binary WS đã đi qua `decode_coordinates_data_proto` nên có schema chuẩn.

## Bài học rút ra

- Khi có nhiều nguồn dữ liệu cùng đích, **tập trung hoá** thành một pipeline chung ngay từ đầu; tránh mỗi nguồn tự lặp logic rồi lệch nhau.
- Realtime hiển thị được KHÔNG chứng minh dữ liệu đã được lưu trữ — hai nhánh (broadcast vs. persist) độc lập.
- Lỗi bị nuốt trong `try/except` che mất triệu chứng: `last_error: null` không có nghĩa "ghi thành công", có thể là "chưa từng ghi".
