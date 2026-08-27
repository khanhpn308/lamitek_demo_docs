# Bố cục GPS Dashboard

- **Trạng thái**: Đã triển khai pha UI
- **Ngày cập nhật**: 2026-08-08
- **Route**: `/dashboard/gps`

## Mục tiêu

GPS Dashboard ưu tiên không gian quan sát bản đồ, đồng thời tách rõ thao tác cấu hình
khỏi dữ liệu realtime. Giao diện dùng toàn bộ chiều ngang khả dụng, không đặt dashboard
trong card có `max-width`, margin lớn, bo góc và shadow bao quanh như bố cục cũ.

## Cấu trúc thông tin

Trên màn hình từ `1024px`, workspace gồm ba vùng theo thứ tự đọc trái sang phải:

1. **Hệ thống — 240px**: Thêm bản đồ, Thêm Anchor, Quản lý Anchor, trạng thái đồng bộ
   Gateway, Xóa bản đồ và Quản lý nhóm. Các thao tác xếp dọc, target tối thiểu 44px.
2. **Không gian bản đồ — co giãn**: toolbar chọn nhóm/map/tìm thiết bị, thông tin map,
   đồng hồ và ảnh mặt bằng realtime. Ảnh mặt bằng tự fit theo cả chiều rộng lẫn chiều cao
   còn lại, giữ đúng tỷ lệ gốc và luôn hiển thị trọn vẹn mà không cần thanh cuộn.
3. **Thiết bị hiển thị — 300px**: danh sách tên/ID thiết bị thuộc map đang chọn.

Dưới `1024px`, hai vùng phụ được chuyển thành drawer mở từ nút **Hệ thống** và
**Thiết bị** trong toolbar. Map tiếp tục chiếm toàn bộ chiều ngang còn lại.

Dashboard submenu vẫn nằm ngang và luôn hiển thị `Telemetry` cùng
`Asset & worker Tracking`.

## Nguyên tắc thị giác

- **Visual hierarchy**: map là vùng nội dung chính; cấu hình và danh sách là vùng phụ.
- **Proximity và grouping**: thao tác cùng nghiệp vụ được gom trong sidebar Hệ thống;
  bộ lọc liên quan trực tiếp tới map nằm trên map.
- **Contrast có kiểm soát**: sidebar Hệ thống dùng nền navy; vùng map sáng; panel thiết bị
  trắng. Viền slate và shadow mềm phân khu mà không tạo hiệu ứng glass hoặc gradient.
- **Màu ngữ nghĩa**: xanh dương cho CTA/focus, đỏ cho xóa/lỗi, amber cho Anchor.
- **Khả năng tiếp cận**: landmark có accessible name, focus ring rõ, control chính cao
  tối thiểu 44px và drawer dùng dialog có focus management.

## Tương thích nghiệp vụ

- Nút **Thêm bản đồ** tiếp tục mở đúng luồng upload map hiện có; không đổi API hoặc
  payload.
- Quyền hiển thị Thêm Anchor, Quản lý Anchor và Xóa bản đồ giữ nguyên.
- Lọc thiết bị theo location và tìm theo tên/ID giữ nguyên.
- Đồng bộ Gateway vẫn dùng `AnchorSyncStatus`; dialog không còn nằm trong luồng layout
  của canvas nên không che khuất nội dung map.

## Ngoài phạm vi pha này

- Tạo tầng và schema/API/database cho tầng.
- Một tầng chứa nhiều map.
- Kéo-thả, resize, snap-to-grid và kiểm tra va chạm map.
- Lưu vị trí/kích thước map trên canvas.

Các mục trên cần đặc tả dữ liệu và migration riêng trước khi triển khai.

## Kiểm thử chấp nhận

- GPS shell chạm hai cạnh vùng nội dung và không còn `max-width`/card wrapper cũ.
- Ba landmark `Hệ thống`, `Không gian bản đồ`, `Thiết bị hiển thị` đúng thứ tự DOM.
- Toolbar hệ thống giữ thứ tự thao tác dọc và luồng upload map cũ.
- Màn hình hẹp có thể mở/đóng cả hai drawer bằng bàn phím.
- Không có nền lưới, crop ảnh hoặc thanh cuộn trong vùng mặt bằng.
- Map, device identity, tìm kiếm, Anchor và group regression tests tiếp tục pass.
