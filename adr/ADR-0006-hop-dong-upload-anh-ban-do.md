# ADR-0006: Hợp đồng upload ảnh bản đồ

- **Trạng thái**: Accepted
- **Ngày**: 2026-07-27
- **Người quyết định**: Phạm Ngọc Khánh
- **Thay thế**: Phần định dạng, kích thước và dung lượng trong ADR-0004

## Bối cảnh

Hợp đồng ban đầu chỉ nhận WebP tĩnh, width đúng 800 px, height 1–8000 px và tối
đa 5 MiB. Người dùng cần tải thêm PNG/JPG và sử dụng bản đồ có tỷ lệ, độ phân
giải khác nhau mà không phải chuyển đổi hoặc resize trước.

MySQL đang lưu ảnh bằng `MEDIUMBLOB`, đủ cho giới hạn mới dưới 10 MiB. Hai bảng
active/archive vẫn cần cùng ràng buộc để archive không thất bại sau khi upload.

## Quyết định

1. Chấp nhận ảnh tĩnh thuộc ba định dạng giải mã thực tế: WebP, PNG và JPEG.
2. Chấp nhận extension `.webp`, `.png`, `.jpg`, `.jpeg` với MIME tương ứng
   `image/webp`, `image/png`, `image/jpeg`.
3. Backend phải đối chiếu extension, MIME và format Pillow giải mã được. File
   giả định dạng, file hỏng, ảnh động hoặc các định dạng khác bị từ chối.
4. Không đặt giới hạn nghiệp vụ cho width/height; metadata phải là số dương.
   Cơ chế chống decompression bomb của Pillow vẫn được giữ như lớp bảo vệ an toàn.
5. Dung lượng hợp lệ là `1 <= file_size_bytes < 10 * 1024 * 1024`. File đúng
   10 MiB hoặc lớn hơn bị từ chối.
   Nginx nhận request tối đa 11 MiB để chừa multipart overhead; backend vẫn áp
   dụng biên dung lượng ảnh nghiêm ngặt.
6. `locations_using` và `locations_deleted` dùng cùng CHECK constraint mới.
   Clean install dùng `schema.sql`; volume cũ được sửa idempotent khi backend startup.
7. Endpoint ảnh trả `Content-Type` đã được backend suy ra từ nội dung thực tế.
   Compatibility path `/floorplans/{location}.webp` giữ nguyên để không phá client cũ,
   nhưng hậu tố path không còn là nguồn chân lý cho MIME.

## Hệ quả

### Tích cực

- Người dùng có thể dùng ảnh từ công cụ thiết kế/chụp phổ biến mà không cần đổi sang WebP.
- Bản đồ có mọi tỷ lệ và độ phân giải hợp lệ đều được lưu, preview và hiển thị.
- Metadata MIME trong database là dữ liệu tin cậy do backend suy ra.
- Contract frontend, API và MySQL thống nhất cùng một biên dung lượng.

### Tiêu cực

- PNG/JPEG có thể lớn hơn WebP tương đương, làm database và backup tăng nhanh hơn.
- Client chỉ hiểu WebP ở compatibility path cũ có thể không hiển thị map PNG/JPEG;
  client hiện tại dùng endpoint `/api/maps/{map_id}/image` và đọc MIME response.
- Ảnh độ phân giải rất lớn vẫn có thể bị Pillow từ chối bởi giới hạn bảo vệ giải nén,
  dù không có giới hạn width/height nghiệp vụ.

## Kiểm chứng

- Unit/integration test bao phủ WebP, PNG, JPG/JPEG, kích thước tùy ý, MIME/extension
  không khớp, ảnh giả định dạng, ảnh động và biên đúng 10 MiB.
- Backend: `69 passed`.
- Frontend: `37 passed` trên 12 test files; production build pass.
- Existing MySQL volume đã đổi CHECK constraint và backend container khởi động healthy.
