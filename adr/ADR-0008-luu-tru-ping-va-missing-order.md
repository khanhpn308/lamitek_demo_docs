# ADR-0008: Lưu trữ Ping và missing order theo device cycle

- **Trạng thái**: Accepted
- **Ngày**: 2026-08-26
- **Liên quan**: ADR-0003, `alltasks.md` PING-3, PING-4 và PING-6 đến PING-8

## Bối cảnh

ESP32 Node gửi application-level ping qua Gateway. Backend cần lưu từng ping hợp lệ và các
order bị thiếu để các phase sau tính tổng payload, payload mới nhất và tổng missing theo
Node. `order` quay lại 1 sau khi Node reset, vì vậy chỉ dùng `(device_id, order)` không phân
biệt được các lần chạy khác nhau. Timestamp trong payload là uptime của Node, không phải Unix
time.

PING-01 dựng nền tảng dữ liệu; Phase P2 bổ sung strict validation và sequence transaction.
Phase P3 đã nối text/binary JSON ping vào WebSocket. Phase P4 đã hoàn thành REST summary,
atomic delete và dedicated admin realtime invalidation. Frontend vẫn thuộc phase sau.

## Quyết định

1. Tạo `ping_payload` với khóa chính `BIGINT AUTO_INCREMENT`; lưu `device_id`, `cycle_id`,
   `order` và `node_timestamp_ms`.
2. Tạo `missing_ping_payload` với khóa chính riêng; `payload_id` mang nghĩa missing order,
   tuyệt đối không phải foreign key tới `ping_payload.id`.
3. Cả hai bảng tham chiếu `device.device_id` với `ON DELETE CASCADE ON UPDATE CASCADE`.
4. Cho phép duplicate/out-of-order trong `ping_payload`; không đặt unique cho order.
5. Chống ghi trùng missing bằng unique `(device_id, cycle_id, payload_id)`.
6. Dùng index `(device_id, id)` cho count/latest và
   `(device_id, cycle_id, order)` cho sequence state.
7. `cycle_id`, `order`, `payload_id` và `node_timestamp_ms` dùng CHECK constraint theo
   contract trong `alltasks.md`.
8. Cung cấp đồng thời ORM, clean-install schema, migration additive `015` và rollback chỉ
   drop hai bảng mới theo thứ tự `missing_ping_payload` rồi `ping_payload`.
9. Xóa dữ liệu một device ở phase sau không reset global auto-increment; trạng thái sequence
   được suy ra lại từ dữ liệu còn tồn tại.
10. Boundary model `PingMessage` dùng Pydantic v2 strict mode, cấm extra field và kiểm tra
    kích thước theo UTF-8 bytes. Formatter lỗi chỉ lấy field/reason, không đưa input vào lỗi.
11. `persist_ping` tạo session riêng từ factory, khóa row `device` bằng `FOR UPDATE`, suy ra
    cycle/predicted order từ DB và commit ping/missing atomically.
12. Forward gap tối đa 10.000 được bulk insert; gap 10.001 trở lên rollback trước khi ghi.
    Duplicate/late order vẫn được lưu, late order xóa missing tương ứng trong current cycle.
13. Lỗi catalog và gap có message nghiệp vụ ổn định; lỗi DB được rollback và chỉ lộ
    `ping persistence failed` cho lớp gọi.
14. `/api/ws/esp32/{device_id}` chỉ nhận dạng application-level ping khi JSON có
    `sensor_type` chính xác bằng `"ping"`; URL ID vẫn là danh tính xác thực Gateway còn JSON
    `device_id` là Node nguồn và được kiểm tra trong catalog.
15. Text và binary JSON ping dùng chung validation/persistence path. Backend chỉ echo raw
    text/bytes ban đầu sau khi transaction commit, giữ nguyên frame type và không reserialize.
16. Recognized ping lỗi trả text `ping_error` đã redacted, giữ connection sống và không đi
    ACK, presence, Influx hay telemetry pipeline. Ping hợp lệ cũng không cập nhật
    `device.last_seen_at`.
17. WebSocket không còn special-case `PING|`; MQTT legacy `PING|` vẫn giữ nguyên. Binary
    non-UTF-8, malformed hoặc non-ping tiếp tục đi binary/protobuf pipeline cũ.
18. Summary/delete REST chỉ dành cho admin. Summary dùng aggregate/latest-by-id; delete khóa
    cùng Device row với ingest và xóa missing/payload trong một transaction.
19. Ping admin realtime dùng group riêng và JWT subprotocol `iot-jwt`; event chỉ gồm
    `type/device_id/reason`, không đi telemetry groups. Sync delete schedule event thread-safe
    về asyncio loop sau commit.

## Các phương án đã loại

### Dùng `payload_id` làm foreign key tới ping đã nhận

Missing order là gói chưa nhận nên không có row cha tương ứng. Foreign key sẽ biểu diễn sai
nghiệp vụ và ngăn lưu khoảng trống.

### Unique `(device_id, order)`

Order được phép lặp do Node reset, duplicate hoặc gói đến muộn. Unique này sẽ làm mất dữ liệu
đo thực tế.

### Lưu timestamp thành `DATETIME`

Timestamp là uptime millisecond của ESP32; chuyển thành datetime sẽ tạo dữ liệu thời gian sai
nghĩa. Giá trị được giữ nguyên trong `node_timestamp_ms BIGINT`.

### Tạo bảng sequence state thứ ba

Contract đã khóa việc suy ra cycle/predicted order từ lịch sử và dùng row lock của `device`
làm mutex liên worker. PING-01 không thêm state dư thừa.

## Hệ quả

### Tích cực

- Schema biểu diễn rõ reset cycle, duplicate, out-of-order và missing recovery.
- Query count/latest/sequence có index phục vụ trực tiếp.
- Clean install và existing volume có cùng tên cột, constraint và index.
- SQLite test vẫn tạo được khóa chính autoincrement nhờ type variant, trong khi MySQL dùng
  `BIGINT`.

### Tiêu cực

- Dữ liệu ping có thể tăng nhanh vì mỗi ping hợp lệ, kể cả duplicate, là một row.
- Khóa row Device serialize ping cùng Node giữa nhiều worker và làm RTT phụ thuộc thời gian
  chờ transaction; đây là trade-off để state nhất quán.
- `ON DELETE CASCADE` làm lịch sử ping mất khi xóa device; đây là lifecycle đã chốt.
- Rollback migration `015` làm mất toàn bộ dữ liệu trong hai bảng và không được chạy tự động.

## Kiểm chứng bắt buộc

- Contract test đối chiếu ORM, `schema.sql`, migration và rollback.
- `Base.metadata.create_all()` tạo được hai bảng trên SQLite memory database.
- Schema tests bao phủ strict types, ranges, UTF-8 byte length và redacted error.
- Service tests bao phủ gap, late recovery, duplicate, reset, clear state và rollback.
- WebSocket tests bao phủ exact text/binary echo, full validation/error matrix, connection
  survival, no-side-effect boundary và non-ping regression.
- REST/realtime tests bao phủ RBAC, zero state, atomic rollback, event order, group isolation,
  stale cleanup và Hub shutdown.
- Toàn bộ backend regression suite phải pass.
- Migration cần được dry-run trên MySQL disposable khi môi trường MySQL sẵn có.
