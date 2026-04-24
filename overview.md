# Tổng Quan `app_service`

## 1. Mục tiêu

`app_service` là một monorepo full-stack cho hệ thống quản lý thiết bị IoT, gồm:

- FE (Frontend): React + Vite + Tailwind/shadcn.
- BE (Backend): FastAPI + SQLAlchemy + JWT.
- RT (Realtime): MQTT subscriber + WebSocket broadcast.
- TSDB (Time-Series DB): InfluxDB cho telemetry.
- Deploy: Docker Compose + Nginx + Mosquitto.

## 2. Kiến trúc tổng thể

Luồng dữ liệu chính:

1. Thiết bị IoT publish dữ liệu lên MQTT broker (Mosquitto).
2. Backend subscriber nhận payload, decode về schema chuẩn.
3. Backend ghi dữ liệu cảm biến vào InfluxDB.
4. Backend phát realtime qua WebSocket cho frontend.
5. Frontend gọi REST API để quản trị user/device và hiển thị dashboard.

Thành phần chính:

- FE app: React SPA ở `app_service/src`.
- API app: FastAPI ở `app_service/backend/app`.
- MQTT broker: Mosquitto cấu hình tại `app_service/mosquitto.conf`.
- Reverse proxy: Nginx cấu hình tại `app_service/nginx`.

## 3. Cây thư mục chính

```text
app_service/
├── backend/                  # FastAPI app
│   ├── app/
│   │   ├── api/              # REST routes
│   │   ├── core/             # config, db, security, mqtt, influx, realtime
│   │   ├── models/           # SQLAlchemy ORM models
│   │   ├── schemas/          # Pydantic schemas
│   │   └── main.py           # FastAPI entrypoint + lifespan
│   ├── requirements.txt      # Python dependencies
│   └── README.md             # Backend setup/run guide
├── src/                      # React source
│   ├── components/           # layout, guards, modals, app router
│   ├── components/ui/        # shadcn/radix UI primitives
│   ├── contexts/             # Auth context
│   ├── pages/                # route pages
│   ├── lib/                  # API client + utils
│   ├── hooks/                # custom hooks
│   ├── data/                 # mock/fallback data
│   ├── styles/               # global styles
│   ├── main.jsx              # FE entrypoint
│   └── App.jsx               # FE root component
├── nginx/                    # Nginx conf (HTTP/HTTPS)
├── generated/                # Webflow generated assets
├── public/                   # Public static assets
├── ssl/                      # TLS cert mount point
├── docker-compose.yml        # App stack compose
├── Dockerfile                # FE build + Nginx image
├── package.json              # FE scripts/dependencies
├── vite.config.js            # Vite config + /api proxy
└── mosquitto.conf            # MQTT broker config
```

## 4. Chức năng theo thư mục

### `backend/`

- Chứa toàn bộ logic API và nghiệp vụ server-side.
- Quản lý xác thực JWT, RBAC theo role, quyền user-device.
- Nhận dữ liệu MQTT, decode payload, ghi Influx, phát WebSocket.

### `backend/app/api/`

- Định nghĩa endpoint HTTP:
  - auth,
  - users,
  - devices,
  - authorizations,
  - mqtt,
  - health.

### `backend/app/core/`

- Hạ tầng lõi backend:
  - `config.py`: đọc biến môi trường.
  - `db.py`: kết nối MySQL.
  - `security.py`: bcrypt + JWT.
  - `mqtt_subscriber.py`: subscriber runtime.
  - `payload_decoder.py`: chuẩn hóa payload.
  - `influx_service.py`: ghi/đọc InfluxDB.
  - `realtime_hub.py`: quản lý websocket clients.

### `backend/app/models/`

- ORM model cho bảng:
  - `user`,
  - `device`,
  - `device_authorization`.

### `backend/app/schemas/`

- Schema request/response API bằng Pydantic.

### `src/`

- Toàn bộ frontend React:
  - routing,
  - auth state,
  - page UI,
  - API integration,
  - dashboard realtime.

### `src/components/ui/`

- Bộ component tái sử dụng dựa trên shadcn/radix.

### `nginx/`

- Reverse proxy và serve static FE.
- Proxy `/api` và `/ws` sang backend.

### `generated/`

- CSS/JS generated từ Webflow, phục vụ token/style nền.

## 5. Chức năng theo file trọng yếu

### Root config/deploy

- `docker-compose.yml`: chạy service `mosquitto`, `backend`, `frontend`.
- `Dockerfile`: build FE bằng Node, serve qua Nginx.
- `vite.config.js`: chạy FE dev ở `3000`, proxy `/api` sang backend.
- `package.json`: scripts `dev/build/preview` và dependencies frontend.
- `mosquitto.conf`: listener MQTT TCP `1883` + WebSocket `9001`.

### Backend entry/router

- `backend/app/main.py`: tạo FastAPI app, startup/shutdown lifecycle, mount router `/api`, mở websocket `/ws/*`.
- `backend/app/api/router.py`: gom tất cả router con.
- `backend/app/api/health.py`: health check app và DB.

### Backend auth/user/device

- `backend/app/api/auth_routes.py`: login/register/bootstrap/me/recover/change-password.
- `backend/app/api/users_routes.py`: list user, đổi trạng thái, xóa user.
- `backend/app/api/devices_routes.py`: CRUD device, list theo quyền, cập nhật topic.
- `backend/app/api/authorizations_routes.py`: gán quyền user-device.

### Backend realtime/MQTT/Influx

- `backend/app/api/mqtt_routes.py`: status/messages/topics/history/influx status.
- `backend/app/core/mqtt_subscriber.py`: subscribe topic, buffer message, callback xử lý payload.
- `backend/app/core/payload_decoder.py`: decode JSON/protobuf/binary template về schema thống nhất.
- `backend/app/core/influx_service.py`: ghi telemetry vào InfluxDB, query history.
- `backend/app/core/realtime_hub.py`: broadcast realtime tới websocket global/device.

### Backend security/db

- `backend/app/core/config.py`: lớp `Settings` cho env.
- `backend/app/core/db.py`: engine + session SQLAlchemy.
- `backend/app/core/deps.py`: `get_db`, `get_current_user`, `require_admin`.
- `backend/app/core/security.py`: hash/verify password và tạo/giải mã JWT.
- `backend/app/core/db_wait.py`: chờ DB ready khi startup.
- `backend/app/core/db_migrate.py`: migration nhẹ khi volume DB cũ.
- `backend/app/core/seed.py`: seed admin + devices mặc định.
- `backend/app/core/user_expiry.py`: tự vô hiệu user hết hạn.

### Backend model/schema

- `backend/app/models/base.py`: base declarative.
- `backend/app/models/user.py`: bảng `user`.
- `backend/app/models/device.py`: bảng `device`.
- `backend/app/models/device_authorization.py`: bảng quyền user-device.
- `backend/app/schemas/auth.py`: schema auth/user.
- `backend/app/schemas/devices.py`: schema device/detail/topic.
- `backend/app/schemas/authorizations.py`: schema authorization.

### Frontend app shell

- `src/main.jsx`: FE entrypoint.
- `src/App.jsx`: app root.
- `src/components/IoTApp.jsx`: định tuyến route public/protected/admin.
- `src/components/Layout.jsx`: layout khung sau đăng nhập.
- `src/components/ProtectedRoute.jsx`: chặn route chưa auth.
- `src/components/AdminRoute.jsx`: chặn route non-admin.
- `src/contexts/AuthContext.jsx`: quản lý token/user/login/logout.

### Frontend API + utils

- `src/lib/api.js`: wrapper fetch + Bearer token + error mapping.
- `src/lib/base-url.ts`: đọc base URL từ env.
- `src/lib/utils.ts`: helper hợp nhất class CSS.

### Frontend pages

- `src/pages/Login.jsx`: đăng nhập.
- `src/pages/ForgotPassword.jsx`: khôi phục mật khẩu.
- `src/pages/Home.jsx`: overview dashboard (mock).
- `src/pages/GlobalDashboard.jsx`: dashboard realtime toàn thiết bị.
- `src/pages/Devices.jsx`: danh sách thiết bị, add/delete.
- `src/pages/DeviceDetail.jsx`: chi tiết thiết bị + chart realtime/history.
- `src/pages/UserManagement.jsx`: quản lý người dùng và phân quyền.
- `src/pages/TopicManagement.jsx`: quản lý topic MQTT theo thiết bị.
- `src/pages/ChangePassword.jsx`: đổi mật khẩu tài khoản.
- `src/pages/Forbidden.jsx`: trang 403.

### Frontend components bổ trợ

- `src/components/AddDeviceModal.jsx`: form tạo device.
- `src/components/AssignDeviceModal.jsx`: form cấp quyền device cho user.
- `src/components/ChangePasswordModal.jsx`: modal đổi password theo ngữ cảnh thiết bị (UI phụ trợ).

### Frontend style/assets

- `src/styles/global.css`: Tailwind + token + global style.
- `src/data/mockData.js`: dữ liệu giả cho fallback/demo.
- `src/hooks/use-mobile.ts`: hook nhận diện mobile viewport.
- `src/vite-env.d.ts`: type cho Vite env.

### Deploy/network

- `nginx/prod.conf`: cấu hình HTTP production.
- `nginx/prod.https.conf`: cấu hình HTTPS production.

## 6. Ghi chú vận hành nhanh

1. FE dev chạy cổng `3000`, backend cổng `8000`.
2. Backend mount prefix API là `/api`.
3. WebSocket chính:
   - `/ws/global`
   - `/ws/devices/{device_id}`
4. MQTT broker:
   - TCP: `1883`
   - WS: `9001`

## 7. Tóm tắt

`app_service` đã có đủ khối cho một nền tảng IoT quản trị tập trung:

- xác thực/phan quyền user-device,
- thu thập telemetry realtime,
- dashboard FE theo quyền,
- cấu hình topic động theo thiết bị,
- sẵn sàng đóng gói Docker cho môi trường production.
