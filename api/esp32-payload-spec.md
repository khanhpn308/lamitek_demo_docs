# Đặc Tả Layout Dữ Liệu ESP32 → Server (MQTT Telemetry)

- **Mã tài liệu**: API-PAYLOAD-001
- **Phiên bản**: 1.0.0
- **Ngày cập nhật**: 2026-06-25
- **Nguồn chân lý (source of truth)**:
  - `app_service/backend/app/core/payload_decoder.py` (`decode_sensor_payload`)
  - `app_service/backend/app/core/test_payload_codec.py` (decoder binary/protobuf)

> Tài liệu này mô tả **contract** dữ liệu thiết bị (ESP32/gateway) gửi lên backend qua MQTT.
> Backend nhận tại topic subscribe (mặc định `devices/+/telemetry`), giải mã về **schema thống
> nhất**, rồi ghi InfluxDB + broadcast WebSocket. Khi firmware đổi định dạng, phải cập nhật
> decoder VÀ tài liệu này cùng lúc.

---

## 1. Kênh truyền & topic

- **Broker**: Mosquitto (TCP `1883`, có auth). WebSocket `9001` chỉ dùng nội bộ.
- **Topic uplink (device → server)**: dạng `devices/{device_id}/telemetry`
  (backend subscribe wildcard `devices/+/telemetry`; có thể gán topic riêng mỗi thiết bị ở trang
  Quản lý topic).
- **device_id**: backend trích từ payload trước, nếu thiếu thì lấy từ topic theo regex
  `(?:devices?/)?(\d{1,12})` (vd `devices/101/telemetry` → `101`).
- **Topic downlink (server → device)**: `devices/{device_id}/downlink` (cấu hình ở `publish_topic`).

### 1.1 Endpoint gửi dữ liệu cụ thể (host : port + URL)

| Kênh                        | Giao thức                        | Host : Port                                       | Đường dẫn / Topic                        | Dùng cho                              |
| --------------------------- | -------------------------------- | ------------------------------------------------- | ---------------------------------------- | ------------------------------------- |
| **MQTT TCP**                | `mqtt://`                        | `<server-ip>:1883`                                | publish topic `devices/{id}/telemetry`   | **ESP32/thiết bị thật** (khuyến nghị) |
| **MQTT WebSocket**          | `ws://`                          | `<server-ip>:9001`                                | cùng topic MQTT                          | Client trong trình duyệt (nội bộ)     |
| **WebSocket ESP32**         | `ws://` (prod), `wss://` (HTTPS) | prod `<server>:80` · dev backend `localhost:8001` | `/api/ws/esp32/{device_id}`              | Thiết bị nói WebSocket thay vì MQTT   |
| **WS broadcast (chỉ nhận)** | `ws://` / `wss://`               | như trên                                          | `/api/ws/global`, `/api/ws/devices/{id}` | Frontend nhận realtime                |

Ghi chú port:

- Production: tất cả HTTP/WS đi qua **nginx cổng `80`** (proxy `/api` và `/ws` → backend container `:8000`).
  Khi bật HTTPS dùng `wss://<domain>` cổng `443`.
- Dev (chạy backend trực tiếp): backend bind `127.0.0.1:8001` → `ws://localhost:8001/api/ws/...`.
- MQTT `1883` (TCP) lộ ra ngoài cho ESP32 thật nhưng **bắt buộc auth** (username/password);
  `9001` (MQTT over WS) chỉ bind localhost cho frontend.
- Route WS được expose ở **cả hai** path (backend include router 2 lần, nginx proxy cả hai):
  `/api/ws/esp32/{id}` và `/ws/esp32/{id}` — dùng path nào cũng được.

#### Ví dụ A — MQTT bằng `mosquitto_pub` (test nhanh từ máy tính)

```bash
# Gửi 1 bản tin telemetry nhiệt độ cho thiết bị id=101
mosquitto_pub -h <server-ip> -p 1883 -u <mqtt_user> -P <mqtt_pass> \
  -t "devices/101/telemetry" \
  -m '{"device_id":"101","sensor_type":"temperature","temperature":28.5,"ts":1714000000}'
```

#### Ví dụ B — MQTT trên ESP32 (Arduino, thư viện PubSubClient)

```cpp
#include <WiFi.h>
#include <PubSubClient.h>

WiFiClient espClient;
PubSubClient mqtt(espClient);

const char* MQTT_HOST = "192.168.1.50";   // IP server (broker)
const int   MQTT_PORT = 1883;
const char* MQTT_USER = "esp32-node";
const char* MQTT_PASS = "******";
const char* DEVICE_ID = "101";

void connectMqtt() {
  mqtt.setServer(MQTT_HOST, MQTT_PORT);
  while (!mqtt.connected()) {
    mqtt.connect(DEVICE_ID, MQTT_USER, MQTT_PASS);  // clientId nên là device_id
    delay(1000);
  }
}

void loop() {
  if (!mqtt.connected()) connectMqtt();
  mqtt.loop();

  char topic[64];
  snprintf(topic, sizeof(topic), "devices/%s/telemetry", DEVICE_ID);

  char payload[128];
  snprintf(payload, sizeof(payload),
    "{\"device_id\":\"%s\",\"sensor_type\":\"power\",\"voltage\":220.4,\"current\":4.2}",
    DEVICE_ID);

  mqtt.publish(topic, payload);
  delay(5000);
}
```

#### Ví dụ C — WebSocket trên ESP32 (Arduino, thư viện WebSockets của links2004)

```cpp
#include <WebSocketsClient.h>
WebSocketsClient webSocket;

void setup() {
  // Production qua nginx cổng 80; nếu HTTPS dùng beginSSL(host, 443, path)
  webSocket.begin("192.168.1.50", 80, "/api/ws/esp32/101");
}

void loop() {
  webSocket.loop();
  // Gửi JSON telemetry; server echo lại {"ok":true,"received":{...}}
  webSocket.sendTXT("{\"sensor_type\":\"temperature\",\"temperature\":28.5}");
  delay(5000);
}
```

#### Ví dụ D — WebSocket từ trình duyệt (JS, để test uplink)

```js
// Dev: ws://localhost:8001/...  | Prod cùng origin: dùng location.host + wss nếu HTTPS
const ws = new WebSocket("ws://localhost:8001/api/ws/esp32/101");
ws.onopen = () =>
  ws.send(
    JSON.stringify({
      device_id: "101",
      sensor_type: "vibration",
      vibration: 1.8,
    }),
  );
ws.onmessage = (e) => console.log("server echo:", e.data);
```

> **Chọn kênh nào?** Thiết bị thật nên dùng **MQTT (Ví dụ A/B)** — nhẹ, có buffer/QoS, hợp pub/sub.
> WebSocket (`/api/ws/esp32/{id}`) phù hợp khi thiết bị/gateway đã nói HTTP/WS sẵn hoặc cần
> kênh 2 chiều tức thời. Cả hai đều đi qua cùng `decode_sensor_payload` nên payload giống nhau.

## 2. Bộ mã loại cảm biến (THỐNG NHẤT — server là nguồn sự thật)

Firmware phải gửi đúng mã số này (`sensor_type` dạng số) hoặc tên tương ứng.

| Mã  | `sensor_type` | Trường dữ liệu kèm theo         |
| --- | ------------- | ------------------------------- |
| 0   | `unknown`     | —                               |
| 1   | `temperature` | `temperature` (°C)              |
| 2   | `vibration`   | `vibration` (mm/s)              |
| 3   | `power`       | `voltage` (V), `current` (A)    |
| 4   | `gps`         | `x`, `y`, `location` (tuỳ chọn) |

Nếu thiếu `sensor_type`, backend **tự suy luận** theo trường có mặt: có voltage/current → `power`;
có temperature → `temperature`; có vibration → `vibration`; có x & y → `gps`.

## 3. Định dạng payload được hỗ trợ

Backend thử giải mã theo thứ tự sau (dừng ở định dạng đầu tiên khớp). Trường `decode_format`
trong kết quả cho biết định dạng đã dùng.

1. **JSON UTF-8** (khuyến nghị cho ESP32 đơn giản) — mục 4.
2. **Protobuf `coordinates_data`** (GPS) — mục 5.1. → `decode_format = coordinates-data-proto`
3. **Test uplink binary** (đo độ trễ Node→Gateway→Server) — mục 5.2. → `test-uplink-binary`
4. **Protobuf `SimpleSensor`** (test) — mục 5.3. → `protobuf-simple-sensor`
5. **NanoPB template binary** — mục 5.4. → `nanopb-template`
6. **Fallback**: không khớp → `decode_format = raw-bytes` + `raw_hex` + `decode_error` (để debug).

## 4. JSON payload (khuyến nghị)

### 4.1 Trường lõi

| Trường        | Bắt buộc | Mô tả                                                                                       | Alias chấp nhận                       |
| ------------- | -------- | ------------------------------------------------------------------------------------------- | ------------------------------------- |
| `device_id`   | Nên có   | ID thiết bị (số/chuỗi). Thiếu → lấy từ topic                                                | `deviceId`, `id`, `node_id`, `nodeId` |
| `sensor_type` | Nên có   | Mã số (1-4) hoặc tên                                                                        | `sensorType`, `type`                  |
| `ts`          | Không    | Thời điểm đo. Giây/mili-giây epoch hoặc ISO-8601. Thiếu/không hợp lệ → server dùng giờ nhận | `timestamp_ms`, `timestamp`, `time`   |

> **Lưu ý `ts`**: giá trị > `1e12` được coi là milliseconds (tự chia 1000). ESP32 gửi uptime
> (giây từ lúc boot) sẽ KHÔNG hợp lệ làm epoch → server thay bằng thời gian nhận.

### 4.2 Trường giá trị theo loại (kèm alias)

| Chỉ số    | Trường chuẩn  | Alias chấp nhận                        |
| --------- | ------------- | -------------------------------------- |
| Nhiệt độ  | `temperature` | `temp`, `temp_c`, `temperature_c`      |
| Độ rung   | `vibration`   | `vibration_mms`, `vibrationMmS`, `vib` |
| Điện áp   | `voltage`     | `volt`, `v`                            |
| Dòng điện | `current`     | `ampere`, `amps`, `a`                  |
| Toạ độ X  | `x`           | `longitude`, `lon`                     |
| Toạ độ Y  | `y`           | `latitude`, `lat`                      |
| Vị trí    | `location`    | `loc`                                  |

Ngoài ra hỗ trợ payload chung `{ "sensor_type": ..., "value": ... }` (alias `reading`,
`measurement`): backend gán `value` vào đúng trường theo `sensor_type`.

### 4.3 Ví dụ JSON

Nhiệt độ:

```json
{
  "device_id": "101",
  "sensor_type": "temperature",
  "temperature": 28.5,
  "ts": 1714000000
}
```

Công suất (power):

```json
{
  "device_id": 102,
  "sensor_type": 3,
  "voltage": 220.4,
  "current": 4.2,
  "ts": 1714000000000
}
```

Độ rung:

```json
{ "device_id": "103", "sensor_type": "vibration", "vibration": 1.8 }
```

GPS:

```json
{
  "device_id": "104",
  "sensor_type": "gps",
  "x": 12.5,
  "y": 80.2,
  "location": "Tang1"
}
```

## 5. Định dạng nhị phân / protobuf

### 5.1 Protobuf `coordinates_data` (GPS)

```proto
syntax = "proto3";
message coordinates_data {
  uint32 device_id   = 1;
  uint32 type        = 2;   // sub-type toạ độ (giữ cho mở rộng)
  float  x           = 3;
  float  y           = 4;
  string location    = 5;
  uint64 timestamp_ms = 6;
}
```

Decoder luôn đặt `sensor_type = "gps"`; `ts = timestamp_ms / 1000`.

### 5.2 Test uplink binary (đo độ trễ) — `version = 0x02`

Layout little-endian, parse nghiêm ngặt (thừa byte → lỗi):

| Offset | Kích thước | Trường                             |
| ------ | ---------- | ---------------------------------- |
| 0      | 1          | `version` (= `0x02`)               |
| 1      | 1 + N      | `message` (len-prefixed ASCII)     |
| …      | 1 + N      | `node_id` (len-prefixed ASCII)     |
| …      | 8          | `event_timestamp_ms` (uint64 LE)   |
| …      | 8          | `gateway_timestamp_ms` (uint64 LE) |
| …      | 1          | `rssi` (int8, bù 256 nếu > 127)    |
| …      | 6          | `src_mac` (6 byte → `AA:BB:...`)   |
| …      | 1 + N      | `gateway_id` (len-prefixed ASCII)  |

Dùng cho pipeline đo trễ Node→Gateway→Server (xem `delaytest`). Server cũng chấp nhận
`t_gateway_appended_ms` thay cho `gateway_timestamp_ms` ở JSON.

### 5.3 Protobuf `SimpleSensor` (test)

```proto
syntax = "proto3";
message SimpleSensor {
  string device_id   = 1;
  float  temperature = 2;
  bool   is_active   = 3;
  uint32 sequence    = 4;
  uint64 timestamp_ms = 5;
}
```

Decoder mặc định `sensor_type = "temperature"`.

### 5.4 NanoPB template binary (khung mẫu — chỉnh theo .proto thực tế)

Little-endian:

| Offset | Kích thước | Trường                                                                                         |
| ------ | ---------- | ---------------------------------------------------------------------------------------------- |
| 0      | 1          | `sensor_type_code` (1=temp, 2=vib, 3=power, 4=gps)                                             |
| 1–4    | 4          | `timestamp_s` (uint32, epoch giây)                                                             |
| 5–8    | 4          | `device_id` (uint32)                                                                           |
| 9…     | 4/8        | float32 theo loại: temp=`temperature`; vib=`vibration`; power=`voltage`+`current`; gps=`x`+`y` |

> GPS dạng đầy đủ (kèm `location` string) nên gửi bằng protobuf `coordinates_data` (5.1) thay vì
> khung byte cố định này.

## 6. Schema thống nhất sau giải mã (đầu ra `decode_sensor_payload`)

Mọi định dạng đều được chuẩn hoá về dict sau (dùng để ghi Influx + broadcast WS). Trường không
áp dụng sẽ là `null`:

```
topic, device_id, sensor_type, ts, ts_iso,
temperature, vibration, voltage, current, x, y, location,
value/is_active/sequence/timestamp_ms,
version, message, node_id, gateway_id, event_timestamp_ms, gateway_timestamp_ms,
rssi, src_mac, decode_format, raw_hex, raw
```

Frontend (GlobalDashboard/DeviceDetail) tiêu thụ các trường: `device_id`, `sensor_type`,
`temperature`, `voltage`, `current`, `vibration`, `x`, `y`, `ts`.

## 7. WebSocket liên quan (tham chiếu)

- `/ws/global`, `/ws/devices/{device_id}`: server **broadcast** dữ liệu đã giải mã (client chỉ nhận).
- `/ws/esp32/{device_id}`: kênh **2 chiều** cho thiết bị (uplink JSON/binary; server echo). Chi tiết
  ở `docs/api/api-documentation.md`.
