# Đặc tả tính năng cấu hình Anchor

- **Mã tài liệu**: SRS-IOT-ANCHOR-001
- **Phiên bản**: 1.2.0
- **Ngày chốt**: 2026-08-09
- **Trạng thái**: Phase 0–6 implemented; production rollout thực hiện theo runbook
- **Phạm vi**: React frontend, FastAPI backend, MySQL và MQTT

## 1. Mục đích

Cho phép người dùng được cấp quyền khai báo vị trí các anchor trên floor map, chia sẻ quyền xem thông qua group hiện có và đồng bộ cấu hình đáng tin cậy xuống mọi gateway tại cùng location.

Anchor là thiết bị tính toán trực tiếp với node ESP32 để suy ra vị trí. Anchor chuyển kết quả tới gateway; gateway mới gửi dữ liệu lên backend. Feature này chỉ cấu hình anchor và chuyển delta/replace xuống gateway, không triển khai logic gateway cấu hình phần cứng bên dưới.

## 2. Thuật ngữ

- **Anchor**: thiết bị có MAC Address, tên và tọa độ x/y/z.
- **Location/Map**: dòng `locations_using`, chứa ảnh floor map và tên location canonical.
- **Group owner**: owner của `map_group` chứa map.
- **Viewer/member**: user đã accept lời mời `map_group_membership`.
- **Gateway**: dòng `device` có `device_type='gateway'`.
- **Delta**: danh sách Anchor thay đổi, mỗi phần tử có `action=upsert|delete`.
- **Replace**: toàn bộ danh sách Anchor active, chỉ dùng để bootstrap/resync/clear map.
- **ACK**: gói gateway xác nhận đã áp dụng hoặc từ chối một revision.
- **Outbox**: bảng lưu event cấu hình và trạng thái giao nhận để tách transaction MySQL khỏi MQTT.

## 3. Phạm vi quyền

### 3.1 Cấp quyền

- `user.can_config_anchor` có giá trị `yes` hoặc `no`, mặc định `no`.
- Admin chọn quyền khi tạo user và thay đổi bằng switch tại trang quản lý user.
- Admin luôn có quyền config anchor, không phụ thuộc giá trị cột.
- Cờ không nằm trong JWT; backend đọc user mới nhất từ DB cho mọi mutation.
- Thu hồi cờ chặn mutation tiếp theo nhưng không xóa, disable hoặc chuyển ownership dữ liệu.

### 3.2 Quyền theo group

- Giữ nguyên quyền group/map đang có.
- Owner active có cờ `yes` được CRUD/resync anchor trong group mình sở hữu.
- Accepted member chỉ xem marker, không có hành động khi click.
- Member có cờ `yes` vẫn không được sửa anchor của group người khác.
- Admin được xem và config mọi anchor.
- Pending/rejected member và outsider không được xem anchor.

## 4. Mô hình dữ liệu nghiệp vụ

### 4.1 Anchor

- Có khóa DB `anchor_id` tự tăng và `mac_address` bất biến dùng với gateway.
- `mac_address` được chuẩn hóa uppercase, đúng sáu octet hexadecimal
  `XX:XX:XX:XX:XX:XX` và unique toàn hệ thống.
- `hardware_id` được giữ tạm thời để tương thích dữ liệu/client cũ; client mới không
  tạo Anchor bằng field này.
- `name` dài 1–100, unique không phân biệt hoa/thường trong các anchor active của cùng map.
- `x/y` là phần trăm trong `[0,100]`; gốc dưới-trái; mặc định `(50,50)`.
- `z` là số hữu hạn, mặc định `0`, được nhập/lưu/gửi nhưng chưa tham gia render 2D.
- X/Y/Z được chuẩn hóa tối đa 2 chữ số thập phân; API làm tròn half-up trước khi lưu.
- Editor dùng bước `0.01`; cả marker từ dữ liệu đã lưu và thao tác kéo trên map đều
  snap vào lưới tọa độ `0.01`.
- Anchor gắn bất biến với một `location_id`; muốn đổi map phải xóa rồi tạo lại.
- Xóa là soft-delete: `status='inactive'`, lưu người/thời điểm xóa, loại khỏi UI và snapshot.
- Không có restore trong phiên bản này; MAC Address đã xóa vẫn được giữ duy nhất.

### 4.1.1 Chuyển đổi dữ liệu legacy

- Migration `013_anchor_mac_address.sql` thêm cột nullable và unique index, không drop
  hoặc đổi kiểu `hardware_id` trong cùng rollout.
- Chỉ backfill ID cũ đã đúng định dạng MAC; ID dạng số hoặc legacy giữ MAC null.
- Editor cho phép gán MAC hợp lệ đúng một lần cho Anchor legacy; sau đó khóa field.
- REST/database tiếp tục giữ `hardware_id` để tương thích dữ liệu cũ; MQTT chỉ gửi
  `mac_address` cho firmware và không phát `hardware_id` legacy.

### 4.2 Quan hệ group

- Không có `anchor_group` hoặc `anchor_group_member` riêng.
- Anchor thuộc map; map thuộc `map_group`; quyền xem dùng `map_group_membership` hiện có.
- Lời mời vẫn dùng `pending → accepted/rejected`.
- Owner có thể nhập tối đa 50 username trong một batch; backend trả kết quả riêng cho từng username.

### 4.3 Trạng thái hiện thực Phase 0

- Clean-install DDL và migration existing-volume đã có tại
  `database_service/sql/schema.sql` và `011_anchor_configuration.sql`.
- ORM đã đăng ký `Anchor`, `AnchorConfigOutbox`, `AnchorConfigDelivery`,
  `User.can_config_anchor` và `Device.last_seen_at`.
- Startup patch tự thêm/backfill hai cột trên volume MySQL cũ; schema chạy được trên
  SQLite test và MySQL 8.4.
- ERD `D:\iot\mysql\erd.mwb` đã loại `anchor_group_member` và khớp nền tảng dữ liệu.
- Dispatcher, ACK consumer và lifecycle Anchor chưa được kích hoạt; các hành vi đó
  vẫn thuộc Phase 4–5.

### 4.4 Trạng thái hiện thực Phase 1

- Register, login, `/auth/me` và `UserPublic` đã truyền `can_config_anchor`; JWT không
  chứa cờ này.
- Admin đã có endpoint cấp/thu hồi quyền; Pydantic chỉ nhận literal `yes|no`.
- Helper permission đã hiện thực admin bypass và điều kiện owner + active + chưa hết
  hạn + cờ `yes`; member không được config dù có cờ.
- Trang quản lý user đã có switch tạo/cập nhật quyền, admin copy cố định và rollback
  khi API lỗi.

### 4.5 Trạng thái hiện thực Phase 2

- REST CRUD, management search/filter/pagination và policy view/mutation đã triển khai.
- Mỗi create/update/delete có thay đổi tạo delta outbox atomic, deterministic; PATCH no-op
  không tạo revision. Resync có đích tạo full replace; request handler không publish MQTT trực tiếp.
- GPS Dashboard tải Anchor theo map; marker viewer chỉ xem, marker config có thể click/kéo.
- Editor hỗ trợ draft `(50,50,0)`, input x/y/z, đổi tên, soft-delete, Cancel và xử lý 403.
- Dialog danh sách quản lý server-side, list-to-map navigation và bulk invitation đã
  hoàn thành ở Phase 3; dispatcher/ACK/status thuộc Phase 4.

### 4.6 Trạng thái hiện thực Phase 3

- Dialog quản lý gọi search/filter/pagination phía server, chỉ hiển thị với admin hoặc
  user có cờ `yes`; backend tiếp tục giới hạn user thường vào group sở hữu.
- Chọn row tải đúng map và danh sách Anchor mới rồi mới mở editor; row stale không được
  dùng làm dữ liệu chỉnh sửa và navigation bị hủy khi quyền/resource thay đổi.
- Bulk invitation nhận tối đa 50 username, lookup exact-case và trả partial result cho
  từng username; UI hỗ trợ danh sách phân cách bằng dòng mới hoặc dấu phẩy.
- Accepted member xem Anchor sau khi accept; pending/rejected không có quyền xem.

### 4.7 Trạng thái hiện thực Phase 4

- Dispatcher claim delivery bằng lease, compose payload riêng theo revision đã ACK của từng Gateway,
  publish QoS 1 retained theo unique topic
  và retry theo schedule mà không làm mất job khi broker/backend restart.
- Gateway active được reconcile theo location; thiếu hoặc trùng publish topic là `misconfigured`.
- MQTT/WS ACK được xác thực theo schema, gateway, topic/path, location và revision; ACK
  idempotent và revision cũ không hạ trạng thái revision mới.
- Presence tracker dùng server time, throttle ghi DB 5 giây và timeout online 30 giây.
- Status/resync API cùng UI badge/per-Gateway/polling 5 giây đã được triển khai.

### 4.8 Trạng thái hiện thực Phase 5

- Map/group/user lifecycle soft-delete Anchor và tạo empty snapshot trước khi archive map.
- `anchor.location_id` là snapshot ID, không còn FK tới active map, để giữ inactive audit row.
- Migration `012_anchor_location_snapshot.sql` và startup repair xử lý existing MySQL volume.
- Device admin mutation và dispatcher định kỳ reconcile latest revision khi Gateway đổi
  location/topic/status hoặc được thêm sau snapshot.

### 4.9 Trạng thái hiện thực Phase 6

- Full backend/frontend, Docker build, MySQL migration và Mosquitto retained/ACK integration pass.
- Gateway simulator CLI cùng runbook rollout/chẩn đoán/rollback đã được bàn giao.
- Chrome responsive smoke xác nhận sync panel, resync request và console sạch.
- Package lock không còn high/critical; Mosquitto bắt buộc password authentication.

## 5. Luồng frontend

### 5.1 Tạo anchor

1. User mở một map thuộc group mình sở hữu.
2. Nút `+ Anchor` chỉ xuất hiện với admin hoặc owner có cờ `yes`.
3. Bấm nút tạo marker draft tại `(50,50,0)`.
4. User nhập name/MAC Address/x/y/z hoặc kéo marker.
5. Drag chỉ thay đổi draft, snap x/y theo `0.01`; không gọi API trong khi di chuyển.
6. Bấm Lưu gửi một POST; Hủy không thay đổi DB/outbox.

### 5.2 Sửa/xóa

- Config user click marker để mở cùng editor.
- MAC Address và map bị khóa sau khi đã gán; Anchor legacy được gán MAC đúng một lần.
- Save gửi một PATCH; chỉ sinh revision nếu dữ liệu chuẩn hóa thực sự thay đổi.
- Delete có xác nhận, soft-delete anchor và sinh delta `delete`.
- Viewer thấy marker/label nhưng click không làm gì.

### 5.3 Danh sách quản lý

- Nút “Quản lý Anchor” chỉ hiện với admin hoặc user có cờ `yes`.
- User thường chỉ thấy anchor trong group mình sở hữu; admin thấy tất cả.
- Search server-side theo name, DB ID, MAC Address và hardware ID legacy; lọc group/map; phân trang 25, tối đa 100.
- Click kết quả tự chuyển dashboard tới đúng group/map, chờ ảnh và anchor tải xong rồi mở editor để có thể kéo marker.

### 5.4 Trạng thái gateway

- UI poll status mỗi 5 giây khi user đang config map.
- Hiển thị badge tổng quan và từng gateway: device ID/name, online/offline, last seen, target/applied revision, pending/applied/rejected/misconfigured và lỗi.
- Nút “Gửi lại cấu hình” nằm trên từng Gateway và tạo full `replace` chỉ cho Gateway đó.

## 6. Lưu và đồng bộ gateway

### 6.1 Transaction

- Create/update/delete có thay đổi tạo delta trong cùng transaction với thay đổi DB; resync có đích tạo replace.
- HTTP handler không publish MQTT trực tiếp.
- Commit DB thành công nghĩa là anchor và outbox cùng tồn tại; rollback loại cả hai.
- Với từng Gateway, backend coalesce mọi event sau `applied_revision`; action cuối của mỗi Anchor thắng.
- Delivery mới chỉ supersede delivery cũ chưa ACK của cùng Gateway và lưu payload wire bất biến để retry.

### 6.2 Chọn gateway

- Chọn mọi device active có loại gateway và location khớp map sau trim/case-normalization.
- Payload luôn dùng location canonical từ `locations_using`.
- Publish qua từng `device.publish_topic` không rỗng.
- Gateway tạo mới dùng mặc định `gateway/{device_id}/backend_receive` cho uplink và
  `gateway/{device_id}/backend_send` cho downlink, trừ khi admin nhập topic riêng.
- Mỗi Gateway active phải có `publish_topic` riêng; create/update trùng topic bị từ chối 409,
  dữ liệu legacy trùng topic bị đánh dấu `misconfigured`.
- Không có gateway vẫn lưu cấu hình và giữ trạng thái `no_gateway`.

### 6.3 MQTT

- Delta và replace đều dùng schema `anchor_config.v1`, QoS 1 và retained.
- `delta` chỉ chứa Anchor cần xử lý; `replace` chỉ dùng khi bootstrap Gateway, resync có đích hoặc clear map.
- Gateway chưa từng ACK baseline luôn nhận replace hiện tại; sau ACK có thể nhận revision delta không liên tiếp.
- Broker failure được retry `5s, 15s, 30s, 60s, 300s`, sau đó mỗi 300s.
- Không drop delivery do hết số lần retry.
- Gateway offline vẫn được publish retained; trạng thái chỉ synced sau ACK applied.

### 6.4 Liveness và ACK

- Mọi telemetry/ACK hợp lệ có `gateway_id=device.device_id` cập nhật `last_seen_at` theo UTC server.
- Ghi MySQL được throttle 5 giây/gateway.
- Gateway online nếu có uplink trong 30 giây gần nhất; timeout cấu hình được.
- MQTT packet phải đến từ `device.topic` tương ứng; WebSocket packet phải khớp device ID đã xác thực.
- ACK `applied` xác nhận revision; `rejected` lưu lỗi và cần revision/resync mới sau khi sửa nguyên nhân.
- ACK cũ hoặc trùng không thay đổi aggregate revision hiện tại.
- Delivery đã `applied`/`rejected` là terminal và không bị ACK mâu thuẫn đến sau ghi đè;
  ACK cho delivery `superseded` chỉ cập nhật dấu thời gian/lỗi audit.

## 7. Lifecycle cha

- Khi xóa map, group hoặc owner, backend tự soft-delete anchor active trước khi xóa `locations_using`.
- Mỗi location bị xóa sinh replace rỗng để gateway loại toàn bộ anchor.
- Cleanup lifecycle không yêu cầu `can_config_anchor`, nhằm giữ quyền xóa map/group hiện tại.
- Outbox lưu snapshot location độc lập FK với active map để vẫn publish sau khi map đã được archive/xóa.

## 8. Trạng thái đồng bộ

- `synced`: mọi gateway active đã ACK applied revision hiện tại.
- `partial`: có gateway applied và gateway khác pending/offline.
- `pending`: chưa gateway nào applied.
- `error`: có gateway rejected hoặc misconfigured.
- `no_gateway`: không có gateway phù hợp.

## 9. Yêu cầu phi chức năng

- Không lộ anchor/group qua IDOR; resource ngoài quyền trả 404.
- Owner thiếu cờ nhận 403 cho mutation.
- Snapshot được sắp theo anchor ID để deterministic và dễ test.
- Dispatcher tiếp tục pending delivery sau restart backend.
- Polling phải cleanup khi đổi map/logout/unmount.
- Không làm gián đoạn luồng realtime device/node đang có.
- API list phải phân trang và giới hạn tối đa 100 record/request.

## 10. Tiêu chí chấp nhận

- Admin cấp/thu hồi quyền qua UI và hiệu lực ở request kế tiếp.
- Owner yes tạo anchor bằng input hoặc kéo; mặc định chính xác `(50,50,0)`.
- Owner sửa name/x/y/z hoặc soft-delete và mỗi lần chỉ sinh một revision.
- Accepted member thấy marker nhưng không thể mở editor hoặc mutation API.
- Tạo mới chỉ chấp nhận MAC chuẩn; lowercase được chuẩn hóa uppercase, MAC sai bị từ chối.
- Anchor ID số cũ có thể nhận MAC đúng một lần mà không mất dữ liệu lịch sử.
- Danh sách quản lý tìm được theo name, DB ID và MAC Address trong đúng scope.
- Multi-gateway nhận cùng snapshot; trạng thái partial cho tới khi tất cả ACK.
- No gateway/broker offline không làm mất dữ liệu hoặc outbox.
- ACK applied chuyển UI sang synced; rejected hiển thị error; resync tạo revision mới.
- Xóa map/group/owner tạo snapshot rỗng và không bị FK anchor chặn.
- Migration chạy mới/chạy lại an toàn; ERD Workbench khớp DDL.

## 11. Ngoài phạm vi

- Firmware gateway và logic gateway cấu hình anchor vật lý.
- Di chuyển anchor giữa map.
- Restore anchor đã xóa.
- Phân vai editor/viewer riêng trong membership.
- 3D map hoặc dùng z để render.
- WebSocket scoped riêng cho trạng thái anchor; phiên bản đầu dùng polling.

## 12. Truy vết

- Kế hoạch implementation: [`../../alltasks.md`](../../alltasks.md)
- ADR: [`../adr/ADR-0007-phan-quyen-va-dong-bo-cau-hinh-anchor.md`](../adr/ADR-0007-phan-quyen-va-dong-bo-cau-hinh-anchor.md)
- API/MQTT contract: [`../api/anchor-configuration-api.md`](../api/anchor-configuration-api.md)
