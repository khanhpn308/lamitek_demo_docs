# TÀI LIỆU KẾ HOẠCH VÀ KỊCH BẢN KIỂM THỬ (TESTING DOCUMENT)

## 1. Mục tiêu kiểm thử

Đảm bảo luồng dữ liệu thời gian thực không bị gián đoạn, độ trễ thấp và các container hoạt động ổn định dưới tải.

## 2. Kịch bản kiểm thử (Test Cases)

### Test Case 01: Kiểm tra tính toàn vẹn và giải mã của Binary Payload

- **Mô tả:** Đảm bảo FastAPI đọc và giải mã đúng cấu trúc C-Struct gửi từ ESP32.
- **Các bước:**
  1. Lập trình ESP32 gửi một gói nhị phân fix cứng giá trị (VD: Nhiệt độ: 25.5, Rung động: 4.2).
  2. Đọc log tại Backend FastAPI sau hàm `struct.unpack`.
- **Kết quả mong đợi:** Dữ liệu log trên Backend trùng khớp hoàn toàn với dữ liệu gốc, không bị sai lệch kiểu dữ liệu (Float/Int).

### Test Case 02: Đo lường độ trễ End-to-End (Latency Profiling)

- **Mô tả:** Đo độ trễ theo từng chặng truyền và tổng thể end-to-end, không chỉ đo một điểm nhận cuối.
- **Mục tiêu:**
  1. Định lượng đầy đủ các chặng: Node -> Gateway -> Server -> Frontend.
  2. Đánh giá theo nhiều chỉ số thống kê: trung bình, p95, p99, max.
  3. Xác minh tiêu chí NFR độ trễ trong điều kiện tiêu chuẩn.
- **Điều kiện tiên quyết:**
  1. Đồng bộ thời gian cho Gateway/Server bằng NTP (UTC), sai lệch clock < 50ms.
  2. Payload uplink có đủ trường timestamp:
     - `event_timestamp_ms` (Node tạo sự kiện),
     - `gateway_timestamp_ms` (Gateway nhận),
     - `mark_time_ms` (Server nhận, được ghi tại backend).
  3. Bật chế độ test theo đúng cặp `gateway_id`, `node_id`, `protocol` trên trang TEST admin.
- **Biến đo và công thức:**
  1. `delay_node_to_gateway_ms = gateway_timestamp_ms - event_timestamp_ms`.
  2. `delay_gateway_to_server_ms = mark_time_ms - gateway_timestamp_ms`.
  3. `delay_node_to_server_ms = delay_node_to_gateway_ms + delay_gateway_to_server_ms`.
  4. (Tuỳ chọn mở rộng frontend) `delay_server_to_frontend_ms = frontend_received_ms - mark_time_ms`.
  5. (Tuỳ chọn mở rộng frontend) `delay_end_to_end_ms = frontend_received_ms - event_timestamp_ms`.
- **Các bước:**
  1. Cấu hình ESP32 gửi payload định kỳ 300ms/lần trong 10 phút.
  2. Thu thập tối thiểu 1000 mẫu hợp lệ vào bảng `test_logs`.
  3. Loại bỏ 100 mẫu warm-up đầu tiên để tránh sai lệch lúc khởi tạo kết nối.
  4. Xuất dữ liệu các cột độ trễ từ `test_logs` (hoặc truy vấn DB trực tiếp) để tính thống kê.
  5. Tính các chỉ số cho từng cột độ trễ: `mean`, `median`, `p95`, `p99`, `max`, `std`.
  6. Lặp lại theo 3 bối cảnh:
     - Baseline: 1 node,
     - Normal load: 10 nodes,
     - Stress profile: 50 nodes (đồng bộ với Test Case 03).
- **Kết quả mong đợi:**
  1. `delay_node_to_server_ms` trung bình sau 1000 mẫu < 500ms.
  2. `delay_node_to_server_ms` p95 < 800ms.
  3. Tỉ lệ mẫu lỗi/không giải mã được < 1%.
  4. Không có outlier âm (delay < 0) sau khi đã đồng bộ thời gian NTP.
- **Đầu ra bắt buộc (Artifacts):**
  1. File CSV/JSON chứa ít nhất: `gateway_id`, `node_id`, `event_timestamp_ms`, `gateway_timestamp_ms`, `mark_time_ms`, các cột delay.
  2. Báo cáo tóm tắt theo từng bối cảnh tải (Baseline/Normal/Stress), gồm bảng thống kê và nhận xét nguyên nhân nếu vượt ngưỡng.

### Test Case 03: Kiểm tra chịu tải (Stress Test) Gateway và WebSocket

- **Mô tả:** Đảm bảo Pi 4 và Server không bị nghẽn khi mở rộng số node.
- **Các bước:**
  1. Dùng tool (như JMeter hoặc script Python) mô phỏng 50 node gửi dữ liệu liên tục 300ms/lần vào Gateway.
  2. Mở Dashboard React và quan sát tính mượt mà của Recharts.
- **Kết quả mong đợi:** Mức chiếm dụng CPU/RAM của Pi 4 ổn định. Biểu đồ trên React không bị giật lag, WebSocket không bị rớt kết nối (Timeout).

### Test Case 04: Kiểm tra định tuyến cơ sở dữ liệu (Database Routing Test)

- **Mô tả:** Xác minh hệ thống tuân thủ Polyglot Persistence.
- **Các bước:**
  1. Đăng ký một User mới trên hệ thống.
  2. Gửi một luồng dữ liệu cảm biến (Telemetry) mới.
  3. Truy vấn trực tiếp vào MySQL và InfluxDB.
- **Kết quả mong đợi:** User mới xuất hiện trong table của MySQL. Dữ liệu cảm biến KHÔNG nằm trong MySQL mà phải nằm ở các bucket của InfluxDB.

### Test Case 05: Kiểm tra cơ chế bù trừ khi mất mạng (Network Recovery)

- **Mô tả:** Kiểm tra hệ thống khi liên kết Pi 4 (Gateway) -> Cloud bị đứt.
- **Các bước:**
  1. Ngắt mạng Internet của Pi 4 trong 1 phút (ESP32 vẫn gửi vào Pi 4).
  2. Phục hồi mạng.
- **Kết quả mong đợi:** _Trường hợp lý tưởng:_ Pi 4 có cơ chế lưu tạm (buffer/QoS) và đẩy bù dữ liệu lên InfluxDB kèm đúng timestamp cũ khi có mạng lại. _Trường hợp tối thiểu:_ Hệ thống tự động kết nối lại WebSocket mượt mà và tiếp tục vẽ biểu đồ thời gian thực.
