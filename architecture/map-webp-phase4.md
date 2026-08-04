# Map WebP theo nhóm — Phase 4: Lifecycle và seed

- **Trạng thái**: Hoàn thành
- **Ngày hoàn thành**: 2026-07-23
- **Phạm vi**: cascade khi xóa group/account, visibility theo trạng thái owner và seed map hệ thống

## 1. Kết quả

Phase 4 khép kín vòng đời dữ liệu map. Việc xóa group hoặc tài khoản owner không còn
phụ thuộc thao tác archive thủ công. Bốn floorplan mặc định cũng được đưa vào MySQL
qua cùng validator với map upload, thay vì được phục vụ trực tiếp từ filesystem.

## 2. Cascade transaction

### Xóa group

`DELETE /api/map-groups/{group_id}` thực hiện tuần tự trong một transaction:

1. Khóa group bằng `SELECT ... FOR UPDATE`.
2. Khóa danh sách map active theo thứ tự ID ổn định.
3. Archive từng map sang `locations_deleted` với lý do `group_deleted`.
4. Dọn toàn bộ membership/invitation.
5. Hard-delete group và commit.

Nếu bất kỳ bước nào lỗi, API trả `409` và rollback toàn bộ. Request lặp lại sau khi
xóa thành công trả `404`, không tạo thêm bản archive.

### Xóa tài khoản

`DELETE /api/users/{user_id}` dùng cùng lifecycle helper cho mọi group do user sở hữu.
Map được archive với lý do `owner_deleted`; membership của user trong group của người
khác chỉ bị xóa. Các tham chiếu audit có thể nullable được tách khỏi user trước khi
xóa để lịch sử map vẫn tồn tại.

## 3. Visibility theo trạng thái owner

Policy Phase 1 tiếp tục là nguồn quyết định duy nhất:

- non-admin không thấy group/map nếu owner inactive hoặc hết hạn;
- admin vẫn xem và quản trị được;
- khi owner active lại, quyền xem của accepted member tự phục hồi mà không thay đổi
  membership hay dữ liệu map.

## 4. Seed map hệ thống

Sau khi tạo admin mặc định `AD00000`, startup gọi `ensure_default_maps`:

- lấy hoặc tạo group `System Debug Maps` thuộc `AD00000`;
- đọc read-only `Floor_1.webp`…`Floor_4.webp`;
- kiểm tra bằng cùng `validate_webp` dùng cho upload;
- chỉ chèn location chưa xuất hiện, không phân biệt hoa/thường, trong cả
  `locations_using` và `locations_deleted`;
- commit bốn map trong một transaction.

Docker image đóng gói floorplan seed tại `/app/seed_floorplans`. Biến môi trường
`FLOORPLAN_SEED_DIR` có thể trỏ tới nguồn seed khác. Nếu map đã archive, restart sẽ
không tái tạo map đó.

## 5. Verification

- Backend: `57 passed`.
- Frontend: `27 passed`; Vite production build pass.
- Docker backend build/startup pass.
- MySQL existing deployment có đủ `Floor_1`…`Floor_4`, width 800 px.
- Restart giữ nguyên ID của cả bốn map, xác nhận seed idempotent.
- Test bao phủ cascade group, rollback, request xóa lặp, cascade owner, visibility
  inactive/reactivate, first seed, active duplicate và archived-no-reseed.

## 6. Quyết định và giới hạn

- Filesystem chỉ là nguồn bootstrap read-only; endpoint cũ vẫn đọc MySQL.
- Không restore/purge map archive trong Phase 4.
- Group hệ thống không cần cờ đặc biệt: admin thấy mọi group, user thường chỉ thấy
  group được sở hữu hoặc có accepted membership, vì vậy không tự thấy group seed.

Xem thêm [Phase 3](map-webp-phase3.md),
[ADR-0004](../adr/ADR-0004-luu-tru-map-webp-va-archive.md) và
[task đã hoàn thành](../../alltasks-done.md).
