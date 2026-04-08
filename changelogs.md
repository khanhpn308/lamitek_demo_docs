# Changelog (docs)

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
