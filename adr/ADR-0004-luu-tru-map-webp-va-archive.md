# ADR-0004: Lưu trữ Map WebP và lịch sử archive trong MySQL

- **Trạng thái**: Accepted; phần định dạng/kích thước/dung lượng upload được
  thay thế bởi [ADR-0006](ADR-0006-hop-dong-upload-anh-ban-do.md)
- **Ngày**: 2026-07-22
- **Người quyết định**: Phạm Ngọc Khánh

## Bối cảnh

GPS Tracking cần cho phép người dùng tự tải bản đồ lên và chia sẻ bản đồ theo nhóm. Hệ thống phải giữ được ảnh sau khi người dùng xóa map để phục vụ lịch sử quản trị, đồng thời `location` đã xóa phải có thể được sử dụng lại cho một map mới.

Các ràng buộc:

- Map là WebP tĩnh, tối đa 5 MiB, width 800 px và height 1–8000 px.
- Chỉ map active được dùng để hiển thị GPS.
- Xóa map, group hoặc owner không được làm mất BLOB/metadata lịch sử.
- Clean install và existing MySQL volume phải cùng nhận được schema.
- Các thao tác archive/cascade sau này phải rollback được như một đơn vị.

## Quyết định

1. Lưu byte ảnh trực tiếp trong MySQL bằng `MEDIUMBLOB`.
2. Dùng hai bảng:
   - `locations_using` cho map active.
   - `locations_deleted` cho bản archive đầy đủ.
3. Không dùng cột `status` để biểu diễn lifecycle map.
4. `locations_using.location` là unique toàn hệ thống theo collation không phân biệt hoa/thường; bảng archive không giữ unique này để location có thể được tái sử dụng.
5. `locations_deleted` giữ snapshot và không có foreign key tới user/group.
6. Archive dùng row lock, insert archive trước, delete active sau và không tự commit.
7. Quyền truy cập được gom vào policy chung; admin có toàn quyền, user thường phụ thuộc trạng thái owner và membership `accepted`.
8. Backend giải mã và kiểm tra byte ảnh bằng Pillow; metadata do client gửi không được coi là đáng tin cậy.

## Các phương án đã cân nhắc

### Giữ ảnh trên filesystem

- Ưu điểm: đơn giản, không tăng dung lượng database.
- Nhược điểm: khó đồng bộ giữa container/replica, backup metadata và file dễ lệch nhau, archive nguyên tử với dữ liệu quan hệ phức tạp.
- Kết luận: không chọn cho ảnh do người dùng quản lý.

### Object storage

- Ưu điểm: phù hợp khi số lượng ảnh và lưu lượng rất lớn.
- Nhược điểm: thêm một dịch vụ, credential, quy trình cleanup và transaction phân tán; vượt nhu cầu hiện tại với file tối đa 5 MiB.
- Kết luận: chưa chọn; có thể xem xét lại khi quy mô tăng.

### Một bảng có cột `status`

- Ưu điểm: schema ít bảng hơn.
- Nhược điểm: unique `location` sẽ cản trở việc tái sử dụng; mọi truy vấn GPS phải luôn lọc status; FK có thể làm mất lịch sử khi xóa account/group.
- Kết luận: không chọn.

### Archive chỉ metadata, bỏ BLOB

- Ưu điểm: tiết kiệm dung lượng.
- Nhược điểm: không đáp ứng yêu cầu giữ nguyên ảnh đã xóa vô thời hạn.
- Kết luận: không chọn.

## Hệ quả

### Tích cực

- BLOB và metadata được archive trong cùng transaction.
- Query GPS chỉ đọc bảng active và không cần điều kiện status.
- Location được tái sử dụng sau khi archive.
- Lịch sử không phụ thuộc lifecycle của user/group.
- Backup MySQL chứa cả metadata và ảnh.

### Tiêu cực

- Database và backup tăng dung lượng theo số map đã xóa.
- Không có purge/restore ở phiên bản đầu nên cần giám sát tốc độ tăng của `locations_deleted`.
- Endpoint ảnh phải tránh prefetch BLOB và phải kiểm tra quyền trên từng request.
- Nếu quy mô tăng lớn, việc chuyển sang object storage sẽ cần migration dữ liệu và thay đổi transaction model.

## Kiểm chứng

- Schema đã được dựng trên MySQL 8.4 cho cả clean database và existing volume.
- Unit tests bao phủ validator, permission matrix và archive rollback.
- Toàn bộ backend test tại thời điểm chấp nhận ADR: `26 passed`.
