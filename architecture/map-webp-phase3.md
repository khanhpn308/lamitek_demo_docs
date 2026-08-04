# Upload, hiển thị và archive Map WebP — Phase 3

- **Trạng thái**: Hoàn thành
- **Ngày hoàn thành**: 2026-07-23
- **Phạm vi**: upload WebP, chọn map theo nhóm, phân phối BLOB có kiểm soát quyền, archive map và lịch sử admin
- **Ngoài phạm vi tại thời điểm hoàn thành**: cascade khi xóa group/account và seed bốn map hệ thống;
  các nội dung này đã được hoàn thành ở Phase 4.

> Tài liệu này ghi lại contract tại thời điểm Phase 3. Contract upload hiện hành
> đã được mở rộng sang WebP/PNG/JPEG, mọi kích thước và file nhỏ hơn 10 MiB theo
> [ADR-0006](../adr/ADR-0006-hop-dong-upload-anh-ban-do.md).

## 1. Kết quả

Phase 3 chuyển luồng GPS từ floorplan trên filesystem sang dữ liệu active trong MySQL:

- Owner nhóm hoặc admin upload WebP qua multipart; accepted member chỉ được xem.
- Frontend kiểm tra nhanh định dạng, dung lượng và kích thước trước khi gửi; backend luôn giải mã và kiểm tra lại nội dung thật.
- GPS dùng hai dropdown `Nhóm bản đồ → Khu vực (Map)` và chỉ tải BLOB của map đang chọn.
- Payload thiết bị tiếp tục khớp với `location` bằng trim và so sánh không phân biệt hoa/thường.
- Xóa map là archive copy-before-delete trong một transaction; ID đã archive không được phục vụ lại.
- Admin xem được lịch sử metadata có phân trang, không tải BLOB, preview, restore hoặc purge.

## 2. REST API

| Method | Endpoint | Quyền và mục đích |
|---|---|---|
| `GET` | `/api/map-groups/{group_id}/maps` | Admin, owner, accepted member; chỉ trả metadata |
| `POST` | `/api/map-groups/{group_id}/maps` | Owner/admin upload multipart `location` + `file` |
| `GET` | `/api/maps/{map_id}/image` | Kiểm tra quyền group cho từng request rồi trả WebP |
| `DELETE` | `/api/maps/{map_id}` | Owner/admin archive map |
| `GET` | `/api/admin/deleted-maps` | Admin xem metadata archive với `limit/offset` |
| `GET` | `/api/locations` | Compatibility API đọc location active có thể xem từ MySQL |
| `GET` | `/api/floorplans/{location}.webp` | Compatibility API đọc BLOB từ MySQL sau khi kiểm tra quyền |

Endpoint ảnh trả `Content-Type: image/webp`, `X-Content-Type-Options: nosniff`
và `Cache-Control: private, no-store`. Danh sách và lịch sử chọn cột metadata
tường minh để không vô tình nạp BLOB.

## 3. Validation và bảo mật

- File tối đa 5 MiB, extension `.webp`, MIME `image/webp`, WebP tĩnh, width đúng 800 px, height 1–8000 px.
- Backend đọc tối đa `5 MiB + 1 byte`, dùng Pillow giải mã, không tin tên file, MIME hay kết quả frontend.
- Tên file được loại bỏ path component; `location` được trim và giới hạn 255 ký tự.
- Location active duy nhất không phân biệt hoa/thường; race ở database được ánh xạ thành `409`.
- Upload giới hạn 30 request/giờ theo JWT user, IP chỉ là fallback.
- Lookup ngoài quyền trả `404` để hạn chế IDOR; lịch sử archive là admin-only.
- BLOB không xuất hiện trong JSON, log hay màn lịch sử.

## 4. Frontend

- `UploadMapDialog`: nút dấu cộng, chọn group có `can_manage`, location gateway,
  drag/drop hoặc chọn file, preview, trạng thái busy và cảnh báo tiếng Việt.
- `GPSDashboard`: tải group rồi metadata map; khi đổi location mới gọi endpoint BLOB
  cho đúng một map và thu hồi object URL cũ.
- Owner/admin có nút **Xóa bản đồ** kèm xác nhận.
- `DeletedMapsPanel`: tab admin-only trong dialog quản lý nhóm, bảng metadata và phân trang 20 bản ghi.
- `mapsApi` hỗ trợ JSON, multipart và Blob; `apiFetch` không tự gắn
  `Content-Type` cho `FormData` để trình duyệt tạo boundary đúng.

## 5. Kiểm thử

- Backend: `48 passed`, gồm upload hợp lệ/độc hại, duplicate, rate limit,
  quyền xem/xóa, compatibility API, archive, tái sử dụng location và lịch sử phân trang.
- Frontend: `26 passed`, gồm API multipart/Blob, validation file, dialog upload,
  dropdown nhóm/map, chỉ tải map đang chọn, location matching, xóa và tab admin.
- Vite production build thành công.
- Docker build backend/frontend thành công; browser smoke test xác minh tạo group,
  upload/preview/hiển thị WebP, archive, lịch sử admin và xóa group; console sạch.

## 6. Phase kế tiếp

- Phase 4 đã archive toàn bộ map khi xóa group/account và xử lý owner inactive xuyên suốt lifecycle.
- Phase 4 đã seed `Floor_1`…`Floor_4` vào MySQL theo cùng validator, có tính idempotent.

Xem thêm [Phase 1](map-webp-phase1.md), [Phase 2](map-webp-phase2.md),
[Phase 4](map-webp-phase4.md),
[ADR-0004](../adr/ADR-0004-luu-tru-map-webp-va-archive.md) và
[task đã hoàn thành](../../alltasks-done.md).
