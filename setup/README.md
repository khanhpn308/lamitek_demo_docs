# README Thiết Lập Môi Trường

- **Mã tài liệu**: SETUP-IOT-001
- **Phiên bản**: 1.0.0
- **Ngày cập nhật**: 2026-04-04

## 1. Yêu cầu hệ thống

- Node.js >= 20
- Python 3.12
- MySQL 8
- PowerShell (Windows) hoặc shell tương đương

## 2. Cấu trúc triển khai

Dự án đã tách thành 2 service root độc lập:

- `app_service` (frontend + backend)
- `database_service` (MySQL)

## 3. Cấu hình biến môi trường

### 3.1 database_service

Tạo file `database_service/.env` từ `database_service/.env.example`:

- `MYSQL_ROOT_PASSWORD`
- `MYSQL_DATABASE`
- `MYSQL_USER`
- `MYSQL_PASSWORD`
- `MYSQL_PORT`

### 3.2 app_service

Tạo file `app_service/.env` từ `app_service/.env.example`:

- DB:
  - `DB_HOST`
  - `DB_PORT`
  - `DB_USER`
  - `DB_PASSWORD`
  - `DB_NAME`
- CORS:
  - `CORS_ORIGINS` (ví dụ `http://localhost`)
- Frontend/API:
  - `API_URL`
  - `FRONTEND_HTTP_PORT`
- JWT:
  - `JWT_SECRET`
  - `JWT_EXPIRE_MINUTES`

## 4. Khởi chạy Database Service

```powershell
cd database_service
copy .env.example .env
docker compose up -d
```

## 5. Chạy Backend (chế độ phát triển)

```powershell
cd ..\app_service\backend
py -V:3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

Kiểm tra:

- `http://localhost:8000/api/health`
- `http://localhost:8000/api/health/db`

## 6. Chạy Frontend (chế độ phát triển)

```powershell
cd ..\
npm install
npm run dev
```

Truy cập: `http://localhost:3000`

## 7. Chạy `app_service` bằng Docker

```powershell
cd ..\
copy .env.example .env
docker compose up -d --build
```

## 8. Tài khoản mặc định

Backend tự đảm bảo admin mặc định (nếu chưa tồn tại):

- Username: `AD00000`
- Password: `khanhxx007`

Khuyến nghị đổi mật khẩu ngay sau khi đăng nhập lần đầu.

## 9. Lưu ý vận hành

- Không commit file `.env`.
- Nếu schema DB cũ, backend có cơ chế vá migration nhẹ khi startup.
- MQTT không sẵn sàng không nên làm sập luồng API cốt lõi.

## 10. Triển khai Docker trên Linux (production)

Để triển khai production trên máy chủ Linux cho cả `app_service` và `database_service`,
tham khảo tài liệu chi tiết: `docs/deployment/docker-linux-deployment.md`.

Các lệnh Docker/Compose thường dùng khi vận hành (build, up/down, log): `docs/deployment/docker-linux-tutorial.md`.

Thao tác trực tiếp với MySQL trên máy chủ (root/user, lọc dữ liệu): `docs/deployment/mysql-linux-operations.md`.
