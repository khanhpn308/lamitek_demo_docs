# Hợp đồng API và MQTT cấu hình Anchor

- **Phiên bản contract**: 1.2.0
- **Ngày**: 2026-08-09
- **Trạng thái**: Contract đã triển khai và kiểm thử qua Phase 6
- **REST base URL**: `/api`

## 1. Quy ước chung

- REST dùng JWT Bearer hiện có.
- Tọa độ x/y là phần trăm `[0,100]`, gốc dưới-trái; z là số chưa render.
- X/Y/Z được làm tròn half-up còn tối đa 2 chữ số thập phân trước khi lưu và tạo snapshot.
- Datetime trả ISO-8601 UTC.
- Resource ngoài phạm vi xem trả 404; thiếu năng lực config trả 403.
- User permission dùng literal `yes|no` để khớp schema MySQL.

## 2. User permission

> User permission đã triển khai ở Phase 1; Anchor CRUD ở Phase 2; management UI và
> bulk invitations ở Phase 3; status/resync HTTP và MQTT delivery/ACK đã triển khai ở Phase 4.

### POST `/auth/register`

Thêm field tùy chọn:

```json
{
  "can_config_anchor": "no"
}
```

### UserPublic

Mọi response user/login/me bổ sung:

```json
{
  "can_config_anchor": "yes"
}
```

### PATCH `/users/{user_id}/anchor-permission`

Admin-only.

```json
{
  "can_config_anchor": "yes"
}
```

Response: `UserPublic` mới nhất.

## 3. Anchor schemas

### AnchorCreate

```json
{
  "mac_address": "12:21:AA:43:1A:9F",
  "name": "Anchor cửa chính",
  "x": 50.0,
  "y": 50.0,
  "z": 0.0
}
```

- x/y mặc định 50, z mặc định 0 nếu bỏ qua.
- Giá trị có hơn 2 chữ số thập phân được làm tròn, ví dụ `8.325 → 8.33`.
- `mac_address` bắt buộc với client mới, theo đúng sáu octet hexadecimal
  `XX:XX:XX:XX:XX:XX`; backend chuẩn hóa uppercase.
- Ví dụ `12:21:aa:43:jh` không hợp lệ vì `J/H` không thuộc hexadecimal và thiếu octet.
- `hardware_id` là field tương thích ngược tạm thời cho client cũ; không dùng cho UI mới.

### AnchorPatch

```json
{
  "name": "Anchor sảnh chính",
  "x": 42.25,
  "y": 67.5,
  "z": 1.2
}
```

Request phải có ít nhất một field. `hardware_id`, `location_id` và `status` bị từ chối.
Anchor legacy có `mac_address=null` được phép PATCH `mac_address` đúng chuẩn đúng một lần;
sau đó MAC Address bất biến.

### AnchorPublic

```json
{
  "anchor_id": 31,
  "mac_address": "12:21:AA:43:1A:9F",
  "hardware_id": "12:21:AA:43:1A:9F",
  "name": "Anchor cửa chính",
  "x": 50.0,
  "y": 50.0,
  "z": 0.0,
  "location_id": 12,
  "location": "Floor_1",
  "group_id": 4,
  "status": "active",
  "created_by_user_id": 8,
  "created_at": "2026-08-07T10:00:00Z",
  "updated_at": "2026-08-07T10:00:00Z"
}
```

### Mutation response

Create/PATCH:

```json
{
  "data": {
    "anchor_id": 31,
    "mac_address": "12:21:AA:43:1A:9F",
    "hardware_id": "12:21:AA:43:1A:9F",
    "name": "Anchor cửa chính",
    "x": 50.0,
    "y": 50.0,
    "z": 0.0,
    "location_id": 12,
    "location": "Floor_1",
    "group_id": 4,
    "status": "active",
    "created_by_user_id": 8,
    "created_at": "2026-08-07T10:00:00Z",
    "updated_at": "2026-08-07T10:00:00Z"
  },
  "config_revision": 7,
  "sync_status": "pending"
}
```

DELETE:

```json
{
  "deleted_anchor_id": 31,
  "config_revision": 8,
  "sync_status": "pending"
}
```

## 4. Anchor endpoints

### GET `/locations/{location_id}/anchors`

- Trả anchor active theo `anchor_id ASC`.
- Admin, owner hoặc accepted member được xem.

### POST `/locations/{location_id}/anchors`

- Admin hoặc owner active có cờ `yes`.
- Trả 201 và mutation response.

### GET `/anchors/{anchor_id}`

- Trả chi tiết nếu caller được xem group.

### PATCH `/anchors/{anchor_id}`

- Admin hoặc owner active có cờ `yes`.
- Trả mutation response. Nếu dữ liệu sau chuẩn hóa không đổi, trả `config_revision=null`,
  `sync_status="unchanged"` và không ghi outbox.

### DELETE `/anchors/{anchor_id}`

- Soft-delete, trả 200 với revision mới.
- Không có restore endpoint trong version 1.

### GET `/anchors/manage`

Query:

- `q`: tối đa 100 ký tự.
- `group_id`, `location_id`: optional.
- `limit`: mặc định 25, `1–100`.
- `offset`: mặc định 0.

Response:

```json
{
  "data": [],
  "total": 0,
  "limit": 25,
  "offset": 0
}
```

User thường chỉ nhận anchor trong group sở hữu; admin nhận tất cả.
Search khớp name, Anchor DB ID, `mac_address` và `hardware_id` legacy.

## 5. Sync status và resync

### GET `/locations/{location_id}/anchor-config-status`

Chỉ admin hoặc owner có cờ `yes`.

```json
{
  "location_id": 12,
  "location": "Floor_1",
  "revision": 7,
  "aggregate": "partial",
  "anchor_count": 4,
  "gateways": [
    {
      "gateway_id": 101,
      "devicename": "Gateway A",
      "online": true,
      "last_seen_at": "2026-08-07T10:15:29Z",
      "target_revision": 7,
      "applied_revision": 7,
      "delivery_status": "applied",
      "error": null
    },
    {
      "gateway_id": 102,
      "devicename": "Gateway B",
      "online": false,
      "last_seen_at": "2026-08-07T10:12:00Z",
      "target_revision": 7,
      "applied_revision": 6,
      "delivery_status": "published",
      "error": null
    }
  ]
}
```

Aggregate enum: `synced|partial|pending|error|no_gateway`.

`delivery_status` enum:

- `pending`: chưa publish thành công tới broker.
- `published`: broker đã xác nhận QoS 1, chưa có ACK Gateway.
- `applied`: Gateway ACK revision thành công.
- `rejected`: Gateway từ chối và trả lỗi.
- `misconfigured`: Gateway thiếu `publish_topic` hợp lệ.
- `superseded`: revision mới hơn đã thay thế delivery này.

### POST `/locations/{location_id}/gateways/{gateway_id}/anchor-config-resync`

- Tạo full `replace` mới chỉ cho Gateway được chọn mà không sửa anchor.
- Trả 202:

```json
{
  "config_revision": 9,
  "sync_status": "pending",
  "gateway_id": 101
}
```

Endpoint cũ `POST /locations/{location_id}/anchor-config-resync` trả `410 Gone` và không tạo revision.

## 6. Bulk invitations

### POST `/map-groups/{group_id}/invitations/bulk`

> Đã triển khai và kiểm thử trong Phase 3; dùng cùng policy/rate-limit identity với
> endpoint mời đơn.

- Tối đa 50 username; trim và dedupe input nhưng lookup giữ exact-case như endpoint đơn.
- Batch là partial success: lỗi của một username không rollback lời mời hợp lệ khác.

```json
{
  "usernames": ["user01", "user02", "missing"]
}
```

Response 200:

```json
{
  "invited_count": 2,
  "error_count": 1,
  "results": [
    {"username": "user01", "status": "invited", "code": null, "message": null},
    {"username": "user02", "status": "invited", "code": null, "message": null},
    {"username": "missing", "status": "error", "code": "user_not_found", "message": "Không tìm thấy người dùng"}
  ]
}
```

## 7. MQTT downlink delta/replace

Publish payload riêng tới từng `device.publish_topic` hợp lệ, QoS 1, retained:

- Gateway tạo từ web mặc định nhận cấu hình tại
  `gateway/{device_id}/backend_send` (`device.publish_topic`).
- Gateway gửi ACK/telemetry tới `gateway/{device_id}/backend_receive`
  (`device.topic`); backend subscribe topic này ngay sau khi tạo Gateway.

```json
{
  "schema": "anchor_config.v1",
  "operation": "delta",
  "gateway_id": 101,
  "location_id": 12,
  "location": "Floor_1",
  "revision": 7,
  "generated_at": "2026-08-07T10:15:30Z",
  "anchors": [
    {
      "action": "upsert",
      "id": 31,
      "mac_address": "12:21:AA:43:1A:9F",
      "name": "Anchor cửa chính",
      "x": 50.0,
      "y": 50.0,
      "z": 0.0
    },
    {
      "action": "delete",
      "id": 32,
      "mac_address": "12:21:AA:43:1A:A0"
    }
  ]
}
```

Quy tắc firmware:

- `operation=delta`: chỉ xử lý phần tử trong `anchors`; `upsert` ghi đè một Anchor,
  `delete` xóa một Anchor theo `id`/`mac_address`.
- `operation=replace`: thay toàn bộ cấu hình anchor tại location; chỉ dùng khi Gateway mới/đổi
  location-topic, resync đúng Gateway hoặc clear map.
- Firmware nhận diện Anchor bằng `mac_address`; MQTT contract `anchor_config.v1` không gửi
  `hardware_id` legacy. REST và database vẫn giữ field này để tương thích dữ liệu cũ.
- `anchors=[]` chỉ xóa toàn bộ khi `operation=replace`; backend không tạo delta rỗng.
- Bỏ qua revision thấp hơn revision đã áp dụng cho cùng location.
- Revision không bắt buộc liên tiếp. Gateway phải ACK chính revision nhận được sau khi áp dụng toàn bộ payload.
- Nhận lại cùng revision phải idempotent và có thể gửi lại ACK.
- Mỗi Gateway có payload coalesce riêng dựa trên revision cuối đã ACK. Retry luôn phát lại đúng payload đã lưu.
- Hai Gateway active không được dùng chung downlink topic; API create/update trả 409 nếu trùng.
- Backend giữ trạng thái terminal đầu tiên (`applied` hoặc `rejected`); ACK mâu thuẫn
  đến sau không ghi đè, còn ACK của revision `superseded` chỉ được lưu audit.

## 8. MQTT/WebSocket ACK uplink

```json
{
  "type": "anchor_config_ack",
  "schema": "anchor_config_ack.v1",
  "gateway_id": 101,
  "location_id": 12,
  "location": "Floor_1",
  "revision": 7,
  "status": "applied",
  "error": null
}
```

- `status`: `applied|rejected`.
- `error`: null hoặc chuỗi tối đa 500 ký tự.
- Gateway ID phải bằng `device.device_id`.
- MQTT topic phải khớp `device.topic`; WebSocket path/device credential phải khớp gateway ID.

## 9. Mã lỗi nghiệp vụ

| HTTP | Code/ý nghĩa |
|---:|---|
| 403 | `anchor_config_permission_required` |
| 404 | Anchor/location/group không tồn tại trong scope caller |
| 409 | MAC Address/Hardware ID đã tồn tại hoặc `anchor_name_exists` |
| 422 | MAC Address, name, coordinate hoặc body không hợp lệ |

## 10. Migration MAC Address

- Migration `013_anchor_mac_address.sql` thêm `anchor.mac_address VARCHAR(17) NULL`
  và unique index theo chiến lược additive, không xóa `hardware_id`.
- Chỉ backfill các `hardware_id` cũ vốn đã đúng định dạng MAC; ID số/legacy giữ
  `mac_address=null` để quản trị viên gán qua editor.
- Dữ liệu mới từ web luôn dùng `mac_address`. API vẫn đọc/ghi field legacy trong thời
  gian chuyển đổi để tránh làm hỏng Gateway hoặc integration cũ.

## 11. Tham chiếu

- [`../../alltasks.md`](../../alltasks.md)
- [`../srs/anchor-configuration-spec.md`](../srs/anchor-configuration-spec.md)
- [`../adr/ADR-0007-phan-quyen-va-dong-bo-cau-hinh-anchor.md`](../adr/ADR-0007-phan-quyen-va-dong-bo-cau-hinh-anchor.md)
