# Changelog (docs)

## 2026-05-07

- **Backend - Refactoring**
  - Cấu trúc WebSocket routes: chuyển 2 endpoint `/ws/global` và `/ws/devices/{device_id}` từ inline `@app.websocket` trong `app/main.py` sang `app/api/websocket_routes.py` để nhất quán với chuẩn architecture (tất cả routes trong `app/api/**_routes.py`).
  - Cập nhật `app/api/websocket_routes.py`:
    - Thêm endpoint `/ws/global` (broadcast realtime data tới tất cả GlobalDashboard clients).
    - Thêm endpoint `/ws/devices/{device_id}` (broadcast realtime data tới dashboard per-device).
    - Cập nhật endpoint `/ws/esp32/{device_id}` (bi-directional device uplink):
      - Hỗ trợ text JSON, binary frames, ping/pong.
      - Thêm extensive docstring giải thích flow, payload schema, error handling.
      - TODO comment: future downlink support qua `hub.send_to_esp32()` từ REST API.
      - TODO comment: thêm device authentication (API key, JWT token, device_secret).
    - Cập nhật docstring module với data flow diagram: MQTT → decoder → InfluxDB/RealtimeHub/TestService.
    - Thêm hàm `_get_realtime_hub()` helper lấy shared RealtimeHub từ app state.
  - Cập nhật `app/main.py`:
    - Xóa 2 endpoint `/ws/global` và `/ws/devices/{device_id}` (chuyển sang websocket_routes.py).
    - Thêm inline comment rõ ràng giải thích WebSocket routes được define trong websocket_routes.py.
  - Cập nhật `docs/api/api-documentation.md`:
    - Thêm section mới "## 4. WebSocket (Realtime Data Streaming)" với chi tiết:
      - WS `/ws/global`: mô tả, cách kết nối, payload example, ghi chú.
      - WS `/ws/devices/{device_id}`: mô tả, khi nào dùng vs `/ws/global`.
      - WS `/ws/esp32/{device_id}`: uplink/downlink mô tả, binary support, Arduino/ESP32 code example.

- **Backend - Test mode (WebSocket + đo độ trễ)**
  - Bổ sung luồng ghi log uplink qua WebSocket để phục vụ đo độ trễ end-to-end (tính `delay_ms` dựa trên `timestamp_ms`/`ts` và `server_receive_ms`).
  - Cập nhật API test:
    - `GET /api/test/config` (admin): xem cấu hình test mode.
    - `PUT /api/test/config` (admin): cập nhật cấu hình, hỗ trợ `protocol: mqtt | websocket` và `device_id`.
    - `GET /api/test/logs` (admin): bổ sung query params `protocol` và `device_id` để auto-filter log theo cấu hình đang test.

- **Backend - WebSocket compatibility & robustness**
  - Mở đồng thời 2 base path cho WebSocket để tương thích nhiều client:
    - Có prefix `/api`: `/api/ws/global`, `/api/ws/devices/{device_id}`, `/api/ws/esp32/{device_id}`
    - Không prefix `/api`: `/ws/global`, `/ws/devices/{device_id}`, `/ws/esp32/{device_id}`
  - Tăng độ bền handler khi client disconnect (handle `websocket.disconnect` frame) và log rõ connect attempt để debug mạng.
  - Hỗ trợ uplink binary protobuf (GPS `coordinates_data`) trên `/ws/devices/{device_id}` và `/ws/esp32/{device_id}`.

- **Nginx - WebSocket proxy hardening (prod)**
  - Cập nhật cấu hình proxy cho `/ws/` và `/api/ws/` để hỗ trợ Upgrade ổn định: `Connection upgrade`, timeout dài, tắt buffering cho WS.

- **Docs - Troubleshooting**
  - Thêm tài liệu xử lý lỗi “kẹt cổng” (localhost OK nhưng IP LAN fail) và các câu lệnh kiểm tra/xoá/tạo `portproxy` + firewall:
    - `docs/troubleshooting/bugs_ket_port_local.md`

## 2026-04-24

- **Docs**
  - Cập nhật `docs/testing/testcases.md`:
    - Mở rộng chi tiết **Test Case 02 (Latency Profiling)** từ đo đơn điểm thành đo theo từng chặng `Node -> Gateway -> Server -> Frontend`.
    - Bổ sung điều kiện tiên quyết (NTP sync, payload timestamp bắt buộc), công thức đo chi tiết, và bộ chỉ số thống kê (`mean`, `median`, `p95`, `p99`, `max`, `std`).
    - Bổ sung quy trình thu mẫu chuẩn (>= 1000 mẫu, bỏ warm-up), đo theo 3 bối cảnh tải (Baseline/Normal/Stress), và tiêu chí chấp nhận cụ thể cho `delay_node_to_server_ms`.
    - Bổ sung yêu cầu đầu ra kiểm thử (artifact CSV/JSON + báo cáo thống kê).
  - Cập nhật `docs/adr/SRS.md`:
    - Làm rõ NFR độ trễ bằng ngưỡng đo định lượng (`mean` và `p95`) và liên kết phương pháp đo với `docs/testing/testcases.md` (Test Case 02).
  - Cập nhật `docs/guidelines/backend-guidelines.md`:
    - Bổ sung guideline kiểm thử backend cho luồng latency profiling: bắt buộc lưu timestamp/delay theo từng chặng và chuẩn hóa artifact phục vụ phân tích hồi quy hiệu năng.

## 2026-04-13

- **Docs**
  - Thêm `docs/overvew.md`: tài liệu tổng quan chuẩn Markdown cho phạm vi `app_service`, gồm:
    - kiến trúc tổng thể FE/BE/MQTT/Influx/Nginx,
    - cây thư mục chính,
    - chức năng từng thư mục,
    - chức năng từng file trọng yếu (root config, backend modules, frontend pages/components/lib, deployment).

## 2026-04-10

- **Backend**
  - Cập nhật `app_service/backend/app/core/payload_decoder.py`:
    - Bổ sung nhánh decode Protobuf cho schema `SimpleSensor` mới của node ESP32:
      - `string device_id = 1`
      - `float temperature = 2`
      - `bool is_active = 3`
      - `uint32 sequence = 4`
      - `uint64 timestamp_ms = 5`
    - Cập nhật thứ tự fallback decode: JSON UTF-8 -> Protobuf `SimpleSensor` -> template binary NanoPB -> `raw_hex` khi không parse được.
    - Bổ sung map trường chuẩn đầu ra gồm `is_active`, `sequence`, `timestamp_ms`.
    - Chuẩn hóa `ts` ưu tiên từ `timestamp_ms` (nếu có) để đồng bộ timeline dữ liệu giữa publisher và backend.
  - Cập nhật `app_service/backend/app/core/payload_decoder.py`:
    - Bổ sung chuẩn hóa timestamp epoch với guard hợp lệ (tránh ghi điểm về năm `1970` khi node gửi uptime),
    - Parse chuỗi ISO không timezone theo chuẩn UTC (không cộng/bù giờ theo timezone máy chủ, ví dụ `+7`),
    - Chuẩn hóa alias payload cho template mới (`device_id`/`deviceId`/`id`, `sensor_type`/`sensorType`/`type`, `temperature`/`temp`/`temp_c`, ...),
    - Hỗ trợ map payload dạng generic `value`/`reading`/`measurement` theo `sensor_type`.
  - Cập nhật `app_service/backend/app/core/influx_service.py`:
    - Chuẩn hóa ghi field metric theo alias (`temperature`, `vibration`, `voltage`, `current`),
    - Bổ sung guard timestamp khi ghi point vào InfluxDB để fallback về thời gian hiện tại nếu timestamp không hợp lệ.
  - Cập nhật `app_service/backend/app/api/mqtt_routes.py`:
    - Thêm endpoint debug `GET /api/mqtt/influx/status` để kiểm tra trạng thái kết nối Influx (`enabled`, `started`, `bucket`, `measurement`, `last_error`).
- **Frontend**
  - Cập nhật `app_service/src/pages/GlobalDashboard.jsx`:
    - Chuẩn hóa `device_type` có dấu tiếng Việt bằng hàm normalize bỏ dấu,
    - Cho phép suy luận type từ payload realtime (`sensor_type`) và từ history,
    - Bổ sung hiển thị thiết bị phát hiện từ dữ liệu history (trường hợp `device_id` là chuỗi từ node ESP32).
  - Cập nhật `app_service/src/pages/DeviceDetail.jsx`:
    - Fallback truy vấn history theo `topic` khi `device_id` DB không khớp `device_id` payload,
    - Fallback realtime qua `ws/global` + filter theo `topic`/`device_id` để không mất dữ liệu chart khi định danh thiết bị lệch giữa DB và node,
    - Chuẩn hóa timestamp event realtime về milliseconds trước khi render biểu đồ.
- **Docs**
  - Thêm `.gitattributes` trong repo `docs` để chuẩn hóa line ending (DLE = Docs Line Endings), tránh cảnh báo `LF will be replaced by CRLF` khi `git add` trên Windows.
  - Cập nhật `docs/changelogs.md` để phản ánh luồng fix Influx + dashboard mapping cho payload template mới từ ESP32.
  - Thêm `.gitattributes` tại root repo để chuẩn hóa line ending (GLE = Git Line Endings), giảm cảnh báo `LF will be replaced by CRLF` và tránh diff không cần thiết giữa Windows/Linux.
  - Cập nhật `docs/deployment/docker-linux-deployment.md`: bổ sung mục `9.8 Restart container khi có cập nhật` (RCU), kèm quy trình restart/recreate cho `app_service`, `database_service`, `influxdb_service`.
  - Cập nhật `deloy.md`: bổ sung mục `7) Restart container khi có cập nhật` (RCU) cho cả ba stack dịch vụ.
  - Thêm `docs/bugs/2026-04-10-mosquitto-esp32-intermittent-disconnect-clientid.md`: ghi nhận sự cố MQTT kết nối dao động do trùng Client ID ESP32 và quy trình xử lý từng bước.
  - Thêm `docs/bugs/2026-04-09-mysql-auth-docker-env-override.md`: ghi nhận sự cố xác thực MySQL trong Docker do biến môi trường CMD ghi đè `.env`, lỗi `1045` cho `root` và `iot_user`, cùng quy trình xử lý step by step (chuẩn hóa `--env-file`, đồng bộ password/quyền user ứng dụng, và kiểm tra lệch schema `iot`/`demo_iot`).
  - Cập nhật nội dung `docs/bugs/2026-04-10-mosquitto-esp32-intermittent-disconnect-clientid.md`: làm rõ triệu chứng ESP32 vẫn publish trong lúc broker đóng phiên cũ và reconnect.

## 2026-04-08

- **Docker**
  - Tách InfluxDB thành stack riêng `influxdb_service` với `docker-compose.yml` độc lập, volume `influxdb-data`, healthcheck và kết nối external network `iot-net`.
  - Cập nhật `app_service/docker-compose.yml`: gỡ service `influxdb` nội bộ khỏi app stack để `backend` kết nối sang InfluxDB qua tên dịch vụ `influxdb` trên mạng chung `iot-net`.
  - Cập nhật environment cho `backend` trong compose để truyền đầy đủ cấu hình MQTT + Influx (`MQTT_*`, `INFLUX_*`).
- **Backend**
  - Thêm `app_service/backend/app/core/influx_service.py`: service ghi dữ liệu cảm biến vào InfluxDB và truy vấn lịch sử theo cửa sổ thời gian (mặc định 30 phút).
  - Thêm `app_service/backend/app/core/payload_decoder.py`: khung mẫu giải mã payload nhị phân NanoPB (template) + chuẩn hóa dữ liệu cho các loại cảm biến `temperature`, `vibration`, `power`.
  - Thêm `app_service/backend/app/core/realtime_hub.py`: broadcast realtime qua WebSocket cho global dashboard và dashboard theo từng thiết bị.
  - Cập nhật `app_service/backend/app/core/mqtt_subscriber.py`:
    - decode payload nhận từ MQTT bằng decoder mới,
    - callback để ghi Influx + phát realtime,
    - hỗ trợ subscribe/unsubscribe topic động khi runtime (`list_topics`, `subscribe_topic`, `unsubscribe_topic`).
  - Cập nhật `app_service/backend/app/main.py`: khởi tạo `InfluxService`, `RealtimeHub`, wiring callback MQTT, và mở WebSocket endpoint `/ws/global`, `/ws/devices/{device_id}`.
  - Cập nhật `app_service/backend/app/api/mqtt_routes.py`: thêm API quản lý topic động cho admin (`/mqtt/topics`, `/mqtt/topics/subscribe`, `/mqtt/topics/unsubscribe`) và API lịch sử InfluxDB (`/mqtt/history?minutes=30&device_id=...`).
  - Cập nhật mô hình thiết bị: thêm trường `topic` trên bảng `device` (ORM + schema + API), hỗ trợ lưu topic theo từng thiết bị và tự đồng bộ subscribe/unsubscribe runtime khi admin cập nhật topic.
  - Bổ sung API quản lý topic theo thiết bị:
    - `GET /api/devices/topics` (admin): danh sách topic đã lưu theo thiết bị,
    - `PUT /api/devices/{device_id}/topic` (admin): cập nhật riêng topic và đồng bộ runtime.
  - Cập nhật startup `main.lifespan`: khôi phục danh sách subscribe MQTT từ dữ liệu `device.topic` đã lưu trong DB.
  - Cập nhật `app_service/backend/requirements.txt`: thêm dependency `influxdb-client`.
  - Cập nhật file môi trường `app_service/backend/.env.example` và `app_service/.env.example` để bổ sung cấu hình MQTT/Influx/WebSocket.
- **Frontend**
  - Cập nhật `app_service/src/pages/DeviceDetail.jsx`:
    - lấy dữ liệu lịch sử 30 phút từ API `/api/mqtt/history` cho bảng History và chart Dashboard,
    - giữ luồng realtime qua WebSocket nối thêm vào chart,
    - thêm UI cho admin subscribe/unsubscribe topic trực tiếp từ web.
  - Cập nhật `app_service/src/pages/GlobalDashboard.jsx`: preload số liệu gần nhất từ lịch sử 30 phút trước khi nhận stream realtime WebSocket.
  - Thêm trang admin `app_service/src/pages/TopicManagement.jsx` và route `/topic-management`: giao diện quản lý topic MQTT theo từng thiết bị (lưu vào DB + đồng bộ runtime).
- **Docs**
  - Cập nhật `docs/deployment/docker-linux-deployment.md`: bổ sung luồng triển khai `influxdb_service` riêng và thứ tự vận hành nhiều stack.
  - Cập nhật `docs/api/api-documentation.md`, `docs/api/openapi-like.yaml`: bổ sung trường `topic` trong hợp đồng thiết bị và mô tả UI admin quản lý topic.
  - Cập nhật `app_service/backend/README.md`: bổ sung hướng dẫn InfluxDB, payload decode template NanoPB, endpoint topic động, endpoint history và websocket realtime.

## 2026-04-06

- **Docs**
  - Thêm `docs/architecture/codebase-walkthrough.md`: thuật ngữ/viết tắt (JWT, RBAC, MQTT, CCCD, …), bản đồ thư mục `app_service/`, thứ tự đọc code, bảng biến môi trường, file “mỏ neo” khi debug.
  - Cập nhật `docs/architecture/system-architecture.md` (mục 7): liên kết walkthrough và mô tả docstring/JSDoc trong repo.
  - Thêm `docs/app_service-functions.md`: liệt kê toàn bộ function trong phạm vi source `app_service/backend/app` và `app_service/src` (kèm line và loại function).
- **Docker**
  - Chuẩn hóa kết nối `app_service` ↔ `database_service` khi chạy bằng 2 compose riêng: dùng external network chung `iot-net` và `DB_HOST=db` (service name) thay vì `127.0.0.1`.
- **Mã nguồn (comment / docstring)**
  - Backend (`app_service/backend/app/`): module docstring + docstring hàm/lớp cho `main`, `core` (config, db, deps, security, db*wait, mqtt_subscriber, user_expiry), `api/`*, `models/_`, `schemas/\*` (bổ sung đầu file / class nơi cần).
  - Frontend: JSDoc/ghi chú đầu file cho `main.jsx`, `App.jsx`, `IoTApp.jsx`, `AuthContext.jsx`, `lib/api.js`, `lib/base-url.ts`, `ProtectedRoute.jsx`, `AdminRoute.jsx`, `Layout.jsx`.
  - Thêm `app_service/src/components/ui/README.md` (giải thích thư mục shadcn/ui, không doc từng file primitive).
  - Ghi chú đầu file `app_service/vite.config.js` (proxy API khi dev).
- **Frontend (Device types & dashboard)**
  - Cập nhật `AddDeviceModal`: `deviceTypes` còn 3 loại chuẩn: `Nhiệt độ (Temperature)`, `Công suất (Power)`, `Độ rung (Vibration)`.
  - Refactor `DeviceDetail` tab `Dashboard` thành biểu đồ theo loại cảm biến:
    - `Temperature`: chỉ biểu đồ nhiệt độ `°C` theo thời gian.
    - `Power`: biểu đồ `Voltage (V)` và `Current (A)` theo thời gian.
    - `Vibration`: biểu đồ `mm/s` theo thời gian (line chart).
  - Chuẩn hóa hiển thị `device_type` trên trang `Devices` theo 3 loại trên.
  - Cập nhật `GlobalDashboard`:
    - Hiển thị đủ 4 biểu đồ tổng quan `Current`, `Voltage`, `Temperature`, `Vibration` (trục X theo thiết bị, trục Y theo giá trị).
    - Thiết bị chỉ đi vào đúng biểu đồ theo `device_type`: `Temperature` -> biểu đồ nhiệt độ; `Power` -> biểu đồ voltage/current; `Vibration` -> biểu đồ rung.
    - Phạm vi dữ liệu theo quyền: admin xem toàn bộ, user xem thiết bị được phân quyền.
    - Thêm cơ chế auto scale trục Y và tự giãn cột/nhãn theo số lượng thiết bị; thiết bị mới tự xuất hiện khi có dữ liệu realtime.
    - Tối giản giao diện chart: ẩn nhãn thiết bị trên trục X, chỉ hiển thị tên thiết bị khi hover cột (tooltip).
    - Bổ sung nút phóng to/thu nhỏ biểu đồ cho cả `GlobalDashboard` và tab `Dashboard` trong `DeviceDetail` (hỗ trợ thoát bằng `Esc` hoặc click nền).

## 2026-04-05

- **Backend**
  - `DELETE /api/devices/{device_id}` (admin): xóa thiết bị và bản ghi `device_authorization` liên quan.
  - `GET /api/devices/{device_id}`: thêm `authorized_users` (RBAC); `user_device_asignment_id` chỉ trả cho admin.
  - `PATCH /api/devices/{device_id}`: hỗ trợ cập nhật `user_device_asignment_id`.
  - `GET /api/users`: mỗi user có `authorized_devices` (thiết bị đã phân quyền).
  - `GET /api/authorizations`: tham số `user_id` hoặc `device_id` (đúng một trong hai).
- **Frontend**
  - Trang Devices: admin có nút xóa thiết bị (xác nhận bằng `OK`).
  - Quản lý người dùng: mỗi thẻ user hiển thị danh sách thiết bị được phân quyền.
  - Chi tiết thiết bị: danh sách user được phân quyền; khối chỉnh sửa `user_device_asignment_id` chỉ admin.
  - Hotfix tương thích backend cũ: nếu `GET /users` chưa có `authorized_devices`, frontend tự backfill từ `GET /authorizations?user_id=...`.
  - Hiển thị lỗi rõ ràng khi backend chưa deploy endpoint `DELETE /api/devices/{id}` (405 Method Not Allowed).
- **Docs**
  - Thêm `docs/guidelines/git-github-teamwork.md`: hướng dẫn branch, fetch/pull, làm việc nhóm trên GitHub, đồng bộ với remote không mất code local, conflict, rebase/merge, stash, force-with-lease.
  - Bổ sung checklist deploy cực nhanh EC2 (pull, build, restart backend, verify endpoint) tại `docs/deployment/docker-linux-deployment.md` mục `8.7`.
  - Hotfix runtime backend sau deploy: thêm import `date` trong `app_service/backend/app/schemas/devices.py` để tránh lỗi `NameError: name 'date' is not defined` khi khởi động.
- **Frontend hotfix**
  - Trang `Devices`: mỗi thẻ thiết bị hiển thị user đang được phân quyền quản lý; ưu tiên dữ liệu từ `GET /users` và có fallback sang `GET /authorizations?device_id=...` cho backend cũ.

Tài liệu API: `docs/api/openapi-like.yaml`, `docs/api/api-documentation.md`.
