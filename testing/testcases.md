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
