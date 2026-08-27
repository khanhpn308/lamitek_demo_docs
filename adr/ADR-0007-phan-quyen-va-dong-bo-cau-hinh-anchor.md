# ADR-0007: Phân quyền và đồng bộ cấu hình Anchor

- **Trạng thái**: Accepted
- **Ngày**: 2026-08-07; cập nhật quyết định delta-per-Gateway 2026-08-09
- **Người quyết định**: Phạm Ngọc Khánh
- **Liên quan**: ADR-0002, ADR-0003, ADR-0004

## Bối cảnh

Hệ thống cần cho phép một số user cấu hình anchor trên floor map, chia sẻ marker cho thành viên group và gửi cấu hình xuống mọi gateway tại cùng location. 

MySQL và MQTT không thể commit trong cùng một transaction. Publish trực tiếp trong REST handler có thể tạo hai trạng thái sai: gateway nhận dữ liệu đã rollback, hoặc DB đã commit nhưng broker offline làm mất cấu hình. Nhiều gateway tại một location cũng cần được theo dõi riêng, và publish thành công tới broker không chứng minh gateway đã áp dụng cấu hình.

## Quyết định

1. Dùng chung `map_group` và `map_group_membership`; không tạo group/membership riêng cho anchor.
2. Anchor thuộc một `locations_using`; quyền xem kế thừa từ group của map.
3. Admin luôn config được. User thường phải đồng thời là owner group, active và có `can_config_anchor='yes'`.
4. Accepted member chỉ xem marker; cờ cá nhân không cấp quyền sửa group người khác.
5. Mutation có thay đổi tạo event `delta` và transactional outbox trong cùng transaction MySQL; PATCH no-op không tạo revision.
6. Dispatcher compose payload riêng theo `applied_revision` của từng Gateway, coalesce theo Anchor ID (last action wins), rồi publish QoS 1 + retain.
7. Gateway liveness được xác định từ uplink gần nhất; trạng thái applied chỉ được xác nhận bằng ACK revision.
8. Revision mới supersede delivery cũ chưa ACK của cùng Gateway sau khi gộp các delta cần thiết; retry dùng payload delivery bất biến.
9. Full `replace` chỉ dùng cho bootstrap Gateway, resync có đích và xóa map/group/owner; xóa Anchor thông thường là delta `delete`.
10. UI status dùng polling 5 giây thay vì `/ws/global`, vì kênh global hiện không scope theo group/location.
11. Khi map bị archive, `anchor.location_id` được giữ như snapshot ID không FK; như vậy
    Anchor đã soft-delete còn tồn tại để audit trong khi active location có thể bị xóa.

## Các phương án đã loại

### Tạo `anchor_group` riêng

- Tạo hai hệ thống group, invitation và permission gần như trùng nhau.
- User có thể xem map nhưng không xem anchor hoặc ngược lại, làm mô hình quyền khó hiểu.
- Bị loại vì anchor luôn nằm trên một map đã thuộc group.

### Chia sẻ trực tiếp từng anchor qua `anchor_group_member`

- Không biểu diễn được tên/owner/lifecycle của một group thật.
- Không đáp ứng yêu cầu tạo, sửa, xóa group và thêm nhiều người.
- Làm số bản ghi permission tăng theo số anchor × user.

### Publish MQTT trực tiếp rồi commit DB

- Một số gateway có thể đã nhận trước khi gateway khác lỗi và DB rollback.
- Không có cơ chế retry bền vững sau restart.

### Commit DB rồi publish best-effort

- Broker offline làm mất cấu hình; user không có trạng thái hoặc cách retry đáng tin cậy.

### Chỉ dùng telemetry để coi là synced

- Uplink chỉ chứng minh gateway online, không chứng minh revision đã được parse/apply.
- ACK revision bắt buộc để phân biệt `published` và `applied`.

### Full snapshot cho mọi mutation

- Bị loại vì thay đổi một Anchor buộc Gateway reset và xử lý lại mọi Anchor không đổi.
- Delta bị bỏ lỡ được giải quyết bằng ACK revision, outbox bền vững và coalesce mọi event sau revision cuối Gateway đã áp dụng.
- Gateway chưa có baseline hoặc cần phục hồi dùng full `replace` có đích; retained message vẫn bảo đảm bootstrap.

## Hệ quả

### Tích cực

- Không phá quyền group/map hiện tại và không nhân đôi invitation logic.
- DB luôn là source of truth; outbox bảo toàn cấu hình khi broker/gateway offline.
- Delta/replace + revision + ACK hỗ trợ nhiều Gateway có tiến độ khác nhau, retry và hội tụ rõ ràng.
- Member có mô hình chỉ-view đơn giản; owner/admin chịu trách nhiệm config.
- Soft-delete giữ audit và lifecycle map có thể phát snapshot rỗng an toàn.

### Tiêu cực

- Thêm outbox, delivery worker, ACK parser và state machine làm backend phức tạp hơn.
- Gateway firmware phải hỗ trợ cả `operation=delta` với `action=upsert|delete`, `operation=replace`, retained payload và ACK.
- Soft-deleted hardware ID chưa thể tái sử dụng nếu không có restore/purge sau này.
- Polling tạo request định kỳ; WebSocket scoped có thể cần ở phiên bản lớn hơn.

## Kiểm chứng bắt buộc

- Permission matrix được unit/API test.
- Migration/ERD khớp và idempotent.
- Integration test với Mosquitto và gateway simulator bao phủ QoS 1 retain, retry, restart, applied/rejected ACK và multi-gateway.
- Browser/component test bao phủ draft 50:50, drag, explicit save/cancel, viewer read-only và status polling cleanup.
