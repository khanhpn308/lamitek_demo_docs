# TÀI LIỆU ĐẶC TẢ YÊU CẦU HỆ THỐNG (SRS)

**Dự án:** Hệ thống giám sát tình trạng thiết bị IoT công nghiệp
**Phiên bản:** 1.0

## 1. Tổng quan hệ thống

Hệ thống IoT cung cấp giải pháp giám sát thời gian thực các thông số: nhiệt độ, điện áp, dòng điện và rung động (vận tốc mm/s). Dữ liệu được xử lý tại biên (Edge Computing) bởi ESP32, gom cụm tại Raspberry Pi 4 (Gateway), và truyền lên máy chủ để trực quan hóa thông qua Dashboard.

## 2. Kiến trúc hệ thống

- **Lớp Thiết bị (Device Layer):** Cụm ESP32 (5-10 nodes) đọc cảm biến, tính toán vận tốc rung động, đóng gói thành Binary Payload.
- **Lớp Gateway (Edge Layer):** Raspberry Pi 4 chạy Eclipse Mosquitto (MQTT Broker). Nhận dữ liệu qua chuẩn MQTT (TCP port 1883).
- **Lớp Máy chủ (Cloud/Server Layer):** Dockerized.
  - Backend: Python / FastAPI. Giải mã Binary, xác thực, phân phối dữ liệu.
  - Database tĩnh: MySQL (Quản lý User, Gateway ID, Node ID).
  - Database động (Time-Series): InfluxDB (Lưu trữ telemetry với retention policy dọn dẹp định kỳ).
- **Lớp Ứng dụng (Application Layer):** React kết hợp Recharts, nhận stream dữ liệu qua WebSocket.

## 3. Yêu cầu chức năng

- **FR1 (Edge Computing):** ESP32 phải tự tính toán chuyển đổi tín hiệu gia tốc thành vận tốc rung động (mm/s).
- **FR2 (Data Ingestion):** Tần suất gửi dữ liệu từ Node -> Gateway là 300ms/lần.
- **FR3 (Real-time Dashboard):** Frontend hiển thị dữ liệu dạng đồ thị biểu diễn theo thời gian thực (Live line charts).
- **FR4 (Dual Database):** FastAPI phải ghi dữ liệu telemetry vào InfluxDB và quản lý phiên đăng nhập/thiết bị bằng MySQL.

## 4. Yêu cầu phi chức năng

- **NFR1 (Độ trễ - Latency):** Độ trễ End-to-End từ lúc cảm biến thay đổi trạng thái đến khi hiển thị trên giao diện web phải được đo theo phương pháp chuẩn tại `docs/testing/testcases.md` (Test Case 02), với ngưỡng tối thiểu:
  - `delay_node_to_server_ms` trung bình < 500ms,
  - `delay_node_to_server_ms` p95 < 800ms,
  - áp dụng trong điều kiện mạng tiêu chuẩn và có đồng bộ thời gian NTP.
- **NFR2 (Băng thông):** Bắt buộc sử dụng Binary payload (C-struct) giữa Node và Gateway để tối ưu hóa gói tin.
- **NFR3 (Triển khai):** Toàn bộ Backend, MySQL, InfluxDB phải được đóng gói bằng Docker và có file `docker-compose.yml`.
