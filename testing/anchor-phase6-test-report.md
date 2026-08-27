# Báo cáo kiểm thử cấu hình Anchor — Phase 6

- **Ngày chạy**: 2026-08-09.
- **Backend**: `126 passed, 1 skipped` mặc định; MQTT integration opt-in chạy riêng `1 passed`.
- **Frontend**: `79 passed` trên 23 test files.
- **Build**: Vite production và Docker backend/frontend thành công.
- **MySQL 8.4**: migration `012` chạy hai lần trên database disposable; không còn FK
  Anchor/location; database test đã được xóa.
- **Mosquitto thật**: retained QoS 1 được subscriber mới nhận; simulator trả ACK `applied`
  và `rejected`; retained test topic được clear.
- **Chrome**: badge, chi tiết Gateway và resync pass ở 375×800; panel nằm trọn viewport,
  console/page error sạch.
- **Security**: đã vá `nanoid` high và PostCSS moderate; còn 2 moderate React Router cần
  migration major riêng; không còn high/critical. Mosquitto bắt buộc password authentication.

Bao phủ permission/IDOR, Anchor CRUD/validation/rollback, outbox/supersede/retry/lease,
ACK/liveness/status, lifecycle, Gateway reconciliation, invitation, editor/drag/navigation,
poll cleanup và permission revoke. Gateway vật lý cùng quan sát pilot 24 giờ vẫn là bước
thực hiện tại môi trường triển khai, không thể giả lập trong repository.

Đợt regression 2026-08-09 bổ sung delta create/update/delete, PATCH no-op, coalesce theo
`applied_revision` riêng từng Gateway, last-action-wins, retry payload bất biến, targeted
bootstrap/resync, legacy resync 410 và bảo vệ duplicate downlink topic. Frontend xác nhận
resync đúng Gateway và không còn thao tác resync toàn location.

Migration 014 đã chạy lặp hai lần trên volume MySQL hiện tại sau khi backup hai bảng vào
`D:\iot\mysql\anchor_delta_pre014_20260809.sql`; cột delivery payload không còn giá trị null.
Docker backend/frontend healthy. MQTT smoke nhận retained targeted replace revision 47 cho
Gateway 136024, không có `hardware_id`, rồi ACK thực tế đưa delivery/outbox về
`applied/completed`. Regression riêng xác nhận aggregate đúng với Session `autoflush=False`.

Vòng review cuối còn xác nhận ACK terminal không bị ACK mâu thuẫn ghi đè, stale ACK được
lưu audit, Gateway rời location được supersede khỏi hàng đợi và ACK MQTT không đi vào
pipeline telemetry.
