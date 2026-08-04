# Nền tảng Map WebP — Phase 1

- **Trạng thái**: Hoàn thành
- **Ngày hoàn thành**: 2026-07-22
- **Phạm vi**: Schema, ORM, dependency, access policy, WebP validator và archive transaction
- **Ngoài phạm vi**: REST API, giao diện upload/quản lý nhóm và tích hợp GPS end-to-end

> Tài liệu này ghi lại contract tại thời điểm Phase 1. Contract upload hiện hành
> đã được mở rộng sang WebP/PNG/JPEG, mọi kích thước và file nhỏ hơn 10 MiB theo
> [ADR-0006](../adr/ADR-0006-hop-dong-upload-anh-ban-do.md).

## 1. Mục tiêu

Phase 1 tạo lớp nền an toàn để các phase sau cho phép người dùng tải bản đồ lên, chia sẻ theo nhóm và hiển thị thiết bị khi payload `location` khớp bản đồ.

Các nguyên tắc chính:

- Ảnh bản đồ đang dùng được lưu trong MySQL thay vì filesystem.
- Bản ghi active và archive nằm ở hai bảng riêng; không dùng cột `status`.
- Quyền xem/quản lý nhóm được tập trung trong một policy dùng chung.
- Backend giải mã nội dung ảnh thật và không tin extension, MIME hoặc kiểm tra phía frontend.
- Archive là thao tác nguyên tử do transaction của caller quản lý.

## 2. Mô hình dữ liệu

### `map_group`

- Mỗi nhóm có đúng một owner.
- Tên nhóm duy nhất theo từng owner.
- `owner_user_id` dùng `ON DELETE RESTRICT` để không thể xóa owner trước khi xử lý map.
- `created_by_user_id` có thể là admin tạo thay và dùng `ON DELETE SET NULL`.

### `map_group_membership`

- Khóa chính ghép `(group_id, user_id)`.
- Trạng thái lời mời: `pending`, `accepted`, `rejected`.
- Chỉ membership `accepted` được dùng làm quyền xem.

### `locations_using`

- Chỉ chứa map đang hoạt động.
- `location` duy nhất toàn hệ thống theo collation không phân biệt hoa/thường của MySQL.
- Ảnh được lưu bằng `MEDIUMBLOB`.
- Lưu filename, MIME, SHA-256, dung lượng, width, height, group, owner, người upload và thời điểm tạo.
- FK tới group và owner dùng `RESTRICT`, buộc các luồng xóa phải archive map trước.

### `locations_deleted`

- Lưu nguyên BLOB và toàn bộ metadata của map đã xóa.
- Giữ snapshot tên group, owner, người upload, người xóa và lý do xóa.
- Không có foreign key để lịch sử không bị mất khi group hoặc tài khoản bị xóa ở phase sau.
- Các lý do được hỗ trợ: `map_deleted`, `group_deleted`, `owner_deleted`.

## 3. Access policy

Module: `app_service/backend/app/core/map_access.py`

- Admin có quyền xem và quản lý mọi nhóm.
- User thường chỉ quản lý nhóm mình sở hữu khi tài khoản còn active.
- Member chỉ được xem nhóm sau khi lời mời ở trạng thái `accepted`.
- User thường không xem được nhóm nếu bản thân hoặc owner bị deactive/hết hạn.
- Ngày `expired_at` được coi là hết hiệu lực, nhất quán với logic tài khoản hiện tại.

Policy này là nguồn dùng chung cho các endpoint nhóm, map và ảnh sẽ được thêm ở các phase tiếp theo.

## 4. WebP validator

Module: `app_service/backend/app/core/webp_validator.py`

Validator kiểm tra trên byte thực bằng Pillow:

- File không rỗng và không vượt quá 5 MiB.
- Extension, nếu được cung cấp, phải là `.webp`.
- MIME, nếu được cung cấp, phải là `image/webp`.
- Nội dung phải giải mã được và có format WebP.
- Ảnh phải tĩnh, width chính xác 800 px, height từ 1 đến 8000 px.
- File phải decode đầy đủ, không bị cắt ngắn hoặc hỏng.
- Metadata tin cậy trả về gồm width, height, dung lượng và SHA-256.

Mã lỗi nội bộ gồm `empty_file`, `file_too_large`, `unsupported_media_type`, `invalid_webp`, `invalid_width`, `invalid_height` và `animated_webp`. Việc ánh xạ các mã này sang HTTP response thuộc Phase 3.

## 5. Archive transaction

Module: `app_service/backend/app/core/map_archive.py`

Trình tự archive:

1. Kiểm tra lý do xóa và người thực hiện.
2. Đọc `locations_using` bằng `SELECT ... FOR UPDATE`.
3. Nạp group, owner và người upload để tạo snapshot.
4. Chèn đầy đủ BLOB/metadata vào `locations_deleted`, sau đó flush.
5. Xóa bản ghi khỏi `locations_using`, sau đó flush.
6. Trả bản ghi archive; caller quyết định `commit` hoặc `rollback`.

Helper không tự commit. Vì vậy việc archive map, xóa group hoặc xóa owner có thể nằm trong cùng một transaction ở các phase sau.

## 6. Khởi tạo và tương thích database

- Clean install dùng `database_service/sql/schema.sql`.
- Existing volume dùng `Base.metadata.create_all()` trong lifecycle backend để tạo các bảng còn thiếu.
- Schema production dùng InnoDB, `utf8mb4_0900_ai_ci`, `MEDIUMBLOB`, unique/index/FK và CHECK constraints.
- Dependencies mới: Pillow cho decode WebP, `python-multipart` cho upload multipart ở phase sau và pytest cho test backend.

## 7. Kiểm thử đã thực hiện

- Validator dùng ảnh WebP/PNG thật do Pillow tạo, bao phủ type giả, kích thước biên, file quá lớn và ảnh động.
- Permission matrix bao phủ admin, owner, accepted/pending/rejected member, user inactive và owner inactive.
- Archive tests xác minh copy đủ dữ liệu/snapshot, không tự commit, rollback phục hồi active row và lỗi không tạo archive.
- Toàn bộ backend test: `26 passed`.
- MySQL 8.4: clean bootstrap thành công và backend startup thành công trên existing volume.

## 8. Công việc tiếp theo

- Phase 2 đã hoàn thành: Group CRUD, dialog quản lý nhóm và invitation/membership API.
- Phase 3: Upload WebP, endpoint ảnh, lựa chọn map trên GPS và archive từ UI.
- Phase 4 đã hoàn thành: cascade an toàn khi xóa group/owner và seed các map hệ thống.

Xem thêm [tài liệu Phase 2](map-webp-phase2.md), [tài liệu Phase 4](map-webp-phase4.md),
[ADR-0004](../adr/ADR-0004-luu-tru-map-webp-va-archive.md) và
[task đã hoàn thành](../../alltasks-done.md).
