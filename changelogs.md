# Changelog (docs)

## 2026-04-06

- **Docs**
  - Thêm `docs/architecture/codebase-walkthrough.md`: thuật ngữ/viết tắt (JWT, RBAC, MQTT, CCCD, …), bản đồ thư mục `app_service/`, thứ tự đọc code, bảng biến môi trường, file “mỏ neo” khi debug.
  - Cập nhật `docs/architecture/system-architecture.md` (mục 7): liên kết walkthrough và mô tả docstring/JSDoc trong repo.
- **Mã nguồn (comment / docstring)**
  - Backend (`app_service/backend/app/`): module docstring + docstring hàm/lớp cho `main`, `core` (config, db, deps, security, db_wait, mqtt_subscriber, user_expiry), `api/*`, `models/*`, `schemas/*` (bổ sung đầu file / class nơi cần).
  - Frontend: JSDoc/ghi chú đầu file cho `main.jsx`, `App.jsx`, `IoTApp.jsx`, `AuthContext.jsx`, `lib/api.js`, `lib/base-url.ts`, `ProtectedRoute.jsx`, `AdminRoute.jsx`, `Layout.jsx`.
  - Thêm `app_service/src/components/ui/README.md` (giải thích thư mục shadcn/ui, không doc từng file primitive).
  - Ghi chú đầu file `app_service/vite.config.js` (proxy API khi dev).

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
