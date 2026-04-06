# Hướng dẫn về mã nguồn 

- **Mã tài liệu**: ARCH-IOT-002  
- **Phiên bản**: 1.0.0  
- **Ngày cập nhật**: 2026-04-06  

Tài liệu này bổ sung [system-architecture.md](./system-architecture.md): giải thích **tên gọi**, **viết tắt**, **vai trò từng thư mục/file** và **thứ tự đọc code** trong dự án.

---

## 1. Thuật ngữ và viết tắt thường gặp

| Ký hiệu / từ | Ý nghĩa | Dùng ở đâu |
|--------------|---------|------------|
| **API** | Application Programming Interface — giao diện HTTP REST giữa React và FastAPI | `app_service/backend/app/api/` |
| **JWT** | JSON Web Token — chuỗi ký điện tử chứa `sub` (username), `uid`, `role`, `exp` | Header `Authorization: Bearer …`, tạo/ghi trong `core/security.py` |
| **RBAC** | Role-Based Access Control — phân quyền theo `user.role` (`admin` / `user`) và bảng `device_authorization` | Route admin (`require_admin`), user chỉ xem thiết bị được cấp |
| **ORM** | Object-Relational Mapping — map bảng MySQL ↔ class Python | SQLAlchemy trong `app/models/` |
| **Pydantic** | Thư viện validate/request-response schema | `app/schemas/` |
| **CORS** | Cross-Origin Resource Sharing — cho phép trình duyệt gọi API khác origin | `main.py` + `Settings.cors_origins` |
| **MQTT** | Message Queuing Telemetry Transport — broker pub/sub cho IoT | `core/mqtt_subscriber.py`, env `mqtt_*` |
| **QoS** | Quality of Service (0–2) — mức đảm bảo giao tin MQTT | `mqtt_qos` |
| **HS256** | HMAC SHA-256 — thuật toán ký JWT (đối xứng, cần `jwt_secret`) | `Settings.jwt_algorithm` |
| **CCCD** | Căn cước công dân (12 số) — định danh người dùng trong schema | Cột `user.cccd`, validate trong `schemas/auth.py` |
| **`sub` (JWT)** | Subject — trong chuỗi JWT là **username** (không phải user_id) | `decode_token` → `get_current_user` |
| **`device_authorization`** | Bảng quan hệ nhiều-nhiều: user nào được quyền thiết bị nào, có `expired_at` | Phân quyền thiết bị thực sự (khác với cột legacy `user_device_asignment_id` trên `device`) |
| **`user_device_asignment_id`** | Tên cột trong DB (chữ *asignment* là lỗi chính tả cũ) — giá trị số legacy, không thay thế RBAC | Chỉ admin thấy trong chi tiết thiết bị; thường `0` |

---

## 2. Bản đồ thư mục `app_service/`

```
app_service/
├── backend/app/          # FastAPI: API, DB, MQTT
│   ├── main.py           # Điểm vào ASGI, lifespan (DB + MQTT)
│   ├── api/              # Router theo domain (auth, devices, users, …)
│   ├── core/             # Cấu hình, DB session, JWT, phụ thuộc (deps), MQTT
│   ├── models/           # SQLAlchemy ORM — bảng MySQL
│   └── schemas/          # Pydantic — body/query/response JSON
├── src/                  # React (Vite)
│   ├── main.jsx          # mount React
│   ├── App.jsx           # bọc IoTApp
│   ├── components/       # Layout, route guards, modal; ui/ = shadcn/ui
│   ├── contexts/         # AuthContext — JWT trong localStorage
│   ├── lib/              # apiFetch, base URL, utils
│   └── pages/            # Màn hình theo route
└── nginx/                # Cấu hình reverse proxy (production)
```

**Frontend — `src/components/ui/`**  
Đây là các **primitive UI** theo pattern [shadcn/ui](https://ui.shadcn.com/) (Radix + Tailwind). Không chứa nghiệp vụ IoT; tái sử dụng trên toàn app. Khi sửa, ưu tiên giữ API component (props) ổn định.

---

## 3. Thứ tự đọc code khuyến nghị

### 3.1 Luồng request HTTP (backend)

1. `main.py` — `create_app()`, middleware CORS, `lifespan` (chờ DB → migrate nhẹ → seed → MQTT).  
2. `api/router.py` — gom các router con dưới prefix `/api`.  
3. Từng file `api/*_routes.py` — endpoint cụ thể; phụ thuộc `Depends(get_db)`, `Depends(get_current_user)`, `Depends(require_admin)`.  
4. `core/deps.py` — cách lấy `User` từ JWT.  
5. `core/security.py` — bcrypt + JWT encode/decode.  
6. `models/` + `schemas/` — dữ liệu lưu vs dữ liệu trả về JSON.

### 3.2 Luồng đăng nhập (end-to-end)

1. Frontend `Login` → `apiFetch('/api/auth/login', { skipAuth: true })` — `lib/api.js`.  
2. Token lưu `localStorage` key `iot_token`; profile lưu `iot_user`.  
3. `AuthContext` gọi `GET /api/auth/me` khi load để đồng bộ server.  
4. Mọi request sau gửi `Authorization: Bearer` (trừ khi `skipAuth`).

### 3.3 Phân quyền thiết bị

- **Admin**: `GET /api/devices` — toàn bộ thiết bị; CRUD + xóa authorization liên quan khi xóa device.  
- **User**: `GET /api/devices/my` — join `device_authorization` và lọc `expired_at`.  
- Gán quyền: `POST /api/authorizations` (admin); tra cứu theo `user_id` hoặc `device_id` — `GET /api/authorizations?...`.

---

## 4. Biến môi trường quan trọng (backend)

| Biến | Vai trò |
|------|--------|
| `DB_*` / `database_url` (qua `Settings`) | Kết nối MySQL |
| `JWT_SECRET`, `JWT_ALGORITHM`, `JWT_EXPIRE_MINUTES` | Ký và thời hạn token |
| `CORS_ORIGINS` | Danh sách origin được phép (phân tách dấu phẩy) |
| `MQTT_*` | Bật/tắt subscriber, broker, topic, QoS, buffer |

Thứ tự ưu tiên cấu hình được tinh chỉnh trong `Settings.settings_customise_sources`: **biến môi trường runtime** (Docker) phải thắng file `.env` khi deploy.

---

## 5. File “mỏ neo” nên mở đầu tiên khi debug

| Vấn đề | Mở file |
|--------|---------|
| 401 / mất đăng nhập | `core/deps.py`, `core/security.py`, `AuthContext.jsx`, `api.js` |
| 403 admin | `require_admin` trong `deps.py`, `AdminRoute.jsx` |
| Thiết bị không hiện với user | `devices_routes.py` (`/my`), bảng `device_authorization` |
| MQTT không có tin | `mqtt_subscriber.py`, env topic, `GET /api/mqtt/status` |
| DB lỗi khi khởi động | `db_wait.py`, `db_migrate.py`, log migration |

---

## 6. Quy ước đặt tên

- **Python**: `snake_case` hàm/biến; `PascalCase` class (model/schema); route path kiểu REST (`/devices/{device_id}`).  
- **React**: component `PascalCase`; file trang/modal khớp tên route hoặc chức năng.  
- **API path**: tiền tố `/api` gắn trong `main.py` khi `include_router`.

Chi tiết từng module có **docstring** trong mã nguồn (Python) và **JSDoc** ở các file entry/HTTP client (JavaScript).
