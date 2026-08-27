# Báo cáo kiểm thử và bàn giao JSON Ping

**Ngày xác minh:** 2026-08-27  
**Phạm vi:** PING-00 đến PING-17, Checkpoint P1 đến P6.

## 1. Kết quả

Tính năng JSON Ping đã hoàn thành theo contract: Gateway xác thực WebSocket bằng danh tính của
Gateway, backend nhận dạng Node từ `device_id` trong JSON, validate schema nghiêm ngặt, commit dữ
liệu MySQL rồi echo nguyên frame. Nhánh Ping không đi qua telemetry, InfluxDB, presence hoặc MQTT.

Frontend admin `/ping` hỗ trợ chọn thiết bị, xem ba chỉ số, cập nhật qua WebSocket riêng và xóa
toàn bộ dữ liệu Ping của một thiết bị với bước xác nhận.

## 2. Thành phần thay đổi

### Backend và database

- `app_service/backend/app/api/websocket_routes.py`: nhận dạng text/binary JSON Ping, exact raw echo
  sau commit và giữ nguyên pipeline non-Ping.
- `app_service/backend/app/api/ping_routes.py`: summary, delete và admin realtime WebSocket.
- `app_service/backend/app/schemas/pings.py`: schema/response contract.
- `app_service/backend/app/models/ping.py`: ORM cho payload và missing order.
- `app_service/backend/app/services/ping_service.py`: transaction, cycle, gap, duplicate và late
  recovery.
- `app_service/backend/app/core/realtime_hub.py`: group riêng cho admin Ping.
- `app_service/backend/app/api/router.py`, `app_service/backend/app/models/__init__.py`: đăng ký route
  và model.
- `database_service/sql/015_ping_payload_tracking.sql`: migration forward.
- `database_service/sql/015_ping_payload_tracking.rollback.sql`: rollback có cảnh báo mất dữ liệu.
- `app_service/backend/tests/test_ping_*.py` và `app_service/backend/tests/integration/`: schema,
  WebSocket, persistence, RBAC, concurrency và regression.

### Frontend

- `app_service/src/pages/Ping.jsx`: catalog filter, ba card, realtime coalescing, stale-response guard,
  reconnect và delete confirmation.
- `app_service/src/pages/Ping.test.jsx`: loading/error/empty, event race/coalescing và delete flows.
- `app_service/src/components/IoTApp.jsx`, `app_service/src/components/Layout.jsx`: admin route/nav.
- `app_service/src/components/AdminRoute.test.jsx`, `app_service/src/components/Layout.test.jsx`:
  route và navigation regression.
- `app_service/index.html`, `app_service/src/styles/global.css`: bỏ Google Fonts ngoài để tuân thủ
  CSP production; dùng system font stack.
- `app_service/scripts/docker-build-context.test.js`: regression test khóa việc không tải font ngoài.

### Tài liệu

- `docs/api/api-documentation.md`, `docs/api/openapi-like.yaml`.
- `docs/architecture/system-architecture.md`, `docs/adr/ADR-0008-luu-tru-ping-va-missing-order.md`.
- `docs/deployment/docker-linux-deployment.md`, `docs/changelogs.md`.
- `alltasks.md`, `alltasks-done.md`.

## 3. Verification đã chạy

| Kiểm tra | Kết quả |
|---|---|
| Backend full Pytest | **PASS** — 200 passed, 1 skipped, 17.24s |
| Frontend full Vitest (`maxWorkers=4`) | **PASS** — 25 files, 92 tests, 23.17s |
| Vite production build | **PASS** — 2394 modules, 16.17s |
| Docker frontend build sau CSP fix | **PASS** — 2394 modules, 10.51s |
| OpenAPI-like YAML parse | **PASS** |
| `git diff --check` | **PASS** |

Một lần chạy Vitest với 8 workers có một timeout ngẫu nhiên tại test User Management không thuộc
Ping (91/92 pass). Chạy lại toàn suite với 4 workers đạt 92/92. Trước đó một test Anchor khác cũng
từng timeout khi chạy song song cao và pass khi chạy riêng. Đây là timing flake của test suite khi
quá tải worker, không phải failure chức năng Ping.

Suite hiện còn warning có sẵn về React key trong GPS dashboard và React Router v7 future flags;
không có test failure.

## 4. MySQL migration verification

- Rebuild backend trên database hiện có đã tạo đúng `ping_payload` và `missing_ping_payload`, gồm
  primary key, foreign key, check constraint và index theo contract.
- Migration `015_ping_payload_tracking.sql` được áp dụng độc lập trên database tạm sạch
  `iot_ping_verify_20260826` có bảng `device` tối thiểu; schema/index được kiểm tra thành công.
- Database tạm trên đã được drop sau verification; không chạm dữ liệu ứng dụng.

## 5. Full-stack browser/WebSocket smoke

Smoke chạy trên frontend/backend Docker hiện tại và MySQL thật, dùng Gateway/Node tạm
`1999999901`/`1999999902`, sau đó xóa toàn bộ dữ liệu thử nghiệm.

```json
{
  "textExactEcho": true,
  "binaryExactEcho": true,
  "forwardGapMissing": 1,
  "lateRecoveryMissing": 0,
  "liveUiTotal": 4,
  "deleteZeroState": true,
  "consoleIssues": 0
}
```

Kiểm tra cleanup MySQL trả `0/0/0` cho device, `ping_payload` và `missing_ping_payload` của hai ID
tạm. Smoke cũng phát hiện Google Fonts bị CSP chặn; external font links đã được bỏ, thêm regression
test và lần chạy cuối không còn console issue.

## 6. Review và giả định còn lại

- Review correctness/readability/architecture/security/performance không phát hiện lỗi chặn bàn giao.
- MQTT legacy giữ nguyên; chỉ WebSocket legacy `PING|...` bị bỏ special-case theo quyết định đã khóa.
- `timestamp` của Node luôn là uptime millisecond, không chuyển thành datetime và không thay thế thời
  gian backend.
- Docker stack local đã được rebuild để chạy full-stack smoke và đang hoạt động sau kiểm thử.
