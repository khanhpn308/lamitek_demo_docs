# Runbook vận hành cấu hình Anchor

## Điều kiện trước rollout

- Backup MySQL và `D:\iot\mysql\erd.mwb`.
- Chạy lần lượt migration `011_anchor_configuration.sql`, `012_anchor_location_snapshot.sql`,
  `013_anchor_mac_address.sql` và `014_anchor_delta_delivery.sql` trước khi khởi động image backend mới.
- Gateway cần `device_type=gateway`, `status=active`, location canonical, uplink `topic`
  và downlink `publish_topic`.
- Admin có thể tạo Gateway tại **Devices → Add New Device → Gateway**. Nếu không sửa topic,
  hệ thống dùng `gateway/{device_id}/backend_receive` cho uplink và
  `gateway/{device_id}/backend_send` cho downlink.
- Mosquitto dùng `allow_anonymous false`, password file và credential trong backend/Gateway.
- `ANCHOR_DISPATCHER_ENABLED` là kill switch; mặc định bật.
- Mỗi Gateway active phải có `publish_topic` riêng. Topic downlink trùng bị API từ chối 409
  hoặc được dispatcher đánh dấu `misconfigured` nếu là dữ liệu legacy.

## Trình tự rollout

1. Deploy backend và kiểm tra `/api/health`.
2. Deploy backend. Khi Gateway mới hoặc thay location/topic, backend tạo full `replace` có đích.
3. Chạy Gateway simulator với topic/location của pilot và ACK `applied` cho baseline replace.
4. Sửa một Anchor; xác nhận payload kế tiếp có `operation=delta` và chỉ chứa Anchor thay đổi.
5. Deploy frontend; cấp `can_config_anchor=yes` cho owner pilot.
6. Theo dõi badge, outbox pending/rejected và `last_seen_at` ít nhất 24 giờ trước khi mở rộng.

## Chẩn đoán

- `no_gateway`: kiểm tra Gateway active, `device_type` và location.
- `misconfigured`: bổ sung topic còn thiếu hoặc đổi topic bị trùng; backend tự bootstrap,
  không cần chỉnh giả Anchor.
- `pending/partial`: kiểm tra broker, online/last-seen và target/applied revision; sau khi
  khắc phục có thể dùng “Gửi lại cấu hình” trên đúng card Gateway.
- `rejected`: đọc `last_error`, sửa dữ liệu/firmware rồi resync; revision bị từ chối không retry vô hạn.
- Broker offline: không xóa outbox/delivery. Retry `5,15,30,60,300` giây rồi mỗi 300 giây.

## Gateway simulator

Từ `app_service/backend`:

```powershell
.venv\Scripts\python.exe -m tools.anchor_gateway_simulator `
  --host localhost --port 1883 --gateway-id 101 `
  --location-id 12 --location Floor_1 `
  --down-topic gateways/101/config --up-topic gateways/101/telemetry `
  --username <mqtt-user> --password <mqtt-password>
```

Thêm `--reject` để kiểm tra luồng Gateway từ chối cấu hình.

## Rollback

1. Đặt `ANCHOR_DISPATCHER_ENABLED=false` và restart backend để dừng publish nhưng giữ job DB.
2. Rollback image frontend/backend; không drop bảng/cột Anchor.
3. Giữ retained payload cuối làm last-known-good; chỉ clear đúng topic bằng thao tác có chủ đích.
4. Không phục hồi FK `anchor.location_id`; FK đó xung đột với audit sau map archive.

## Giám sát

- Outbox/delivery theo status và tuổi job cũ nhất.
- Publish failure/retry rate, ACK latency và Gateway offline count.
- Error rate/P95 status-resync API và JavaScript error phía frontend.

## Quy tắc xử lý payload phía Gateway

- `operation=replace`: thay toàn bộ danh sách Anchor; áp dụng atomically rồi ACK revision.
- `operation=delta`: chỉ xử lý phần tử trong `anchors`; `action=upsert` ghi đè một Anchor,
  `action=delete` xóa một Anchor. Không reset các Anchor không có trong payload.
- Revision có thể nhảy số vì backend compose riêng theo Gateway; chỉ bỏ qua revision thấp hơn
  revision đã áp dụng. Nhận lại cùng revision phải idempotent và ACK lại được.
- Nếu mất state cục bộ hoặc cần phục hồi, gọi resync có `gateway_id`; không dùng endpoint toàn location cũ.
