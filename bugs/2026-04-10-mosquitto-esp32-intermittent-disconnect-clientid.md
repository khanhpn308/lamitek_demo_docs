# Bug Log - ESP32 kết nối MQTT không ổn định do trùng Client ID

## Thông tin chung

- Thời gian hoàn tất: 2026-04-10
- Khu vực bị ảnh hưởng: app_service (Mosquitto), firmware ESP32
- Mức độ: High (dữ liệu realtime có thể bị gián đoạn ngắn)

## Mô tả lỗi chính xác

Trong lúc theo dõi log Mosquitto, hệ thống ghi nhận ESP32 đã kết nối thành công nhưng sau đó xuất hiện trạng thái đóng kết nối:

- `New client connected ... as ESP32_4dCBDC`
- `Client ESP32_4dCBDC already connected, closing old connection`
- `Client ESP32_4dCBDC closed its connection`

Triệu chứng gây hiểu nhầm:

- Phía ESP32 vẫn báo đang kết nối và tiếp tục publish dữ liệu.
- Backend subscriber vẫn có thời điểm reconnect.

Nguyên nhân gốc:

- Trùng Client ID (`ESP32_4dCBDC`) giữa các phiên kết nối gần nhau (reconnect nhanh hoặc có kết nối cũ chưa timeout), khiến broker đóng phiên cũ theo chuẩn MQTT.
- Keep-alive/reconnect chưa được cấu hình tối ưu nên trạng thái connect/disconnect xuất hiện thường xuyên trong log.

## Dấu hiệu nhận biết

- Log có cặp thông báo `already connected, closing old connection`.
- Dữ liệu vẫn lên nhưng kết nối dao động, có thể bỏ lỡ một số bản tin QoS thấp trong thời điểm chuyển phiên.
- Không có lỗi auth ở broker; lỗi nằm ở quản lý session MQTT phía client.

## Các bước giải quyết (step by step)

1. Xác nhận tình trạng broker:

- Kiểm tra log Mosquitto để xác định kết nối thành công trước khi bị đóng.
- Phân biệt lỗi auth với lỗi session: ở đây không có `Not authorized` hoặc `Bad username/password`.

2. Đối chiếu trạng thái thực tế từ ESP32:

- Xác nhận firmware chỉ publish khi `client.connected()` là true.
- Đối chiếu timestamp publish với timestamp reconnect trong broker log.

3. Chuẩn hóa Client ID trên ESP32:

- Bảo đảm mỗi thiết bị dùng Client ID duy nhất và ổn định.
- Nếu có khả năng chạy nhiều instance/test song song, thêm hậu tố duy nhất (ví dụ MAC hoặc chip ID) để tránh trùng.

4. Cải thiện vòng lặp reconnect:

- Chỉ gọi reconnect khi thật sự mất kết nối.
- Thêm backoff ngắn (ví dụ 1-3 giây) để tránh reconnect dồn dập.
- Giữ một luồng kết nối duy nhất đến broker.

5. Điều chỉnh keep-alive:

- Tăng keep-alive phù hợp (ví dụ 120 giây) để giảm ngắt kết nối giả do timeout mạng ngắn.
- Đảm bảo vòng lặp MQTT được gọi đều để gửi ping đúng hạn.

6. Xác thực sau fix:

- Theo dõi lại log Mosquitto tối thiểu 10-15 phút.
- Tiêu chí pass: không còn chuỗi `already connected, closing old connection` lặp lại bất thường; dữ liệu publish liên tục, không mất nhịp.

## Kết quả sau xử lý

- ESP32 duy trì kết nối ổn định hơn với broker.
- Dữ liệu publish liên tục đúng kỳ vọng.
- Giảm đáng kể hiện tượng đóng/mở session MQTT lặp lại trong log.

## Bài học rút ra

- Trong MQTT, trùng Client ID sẽ luôn khiến broker đóng phiên cũ.
- Log có `connected` rồi `closed` không đồng nghĩa lỗi auth; cần đọc theo chuỗi sự kiện session.
- Với hệ thống IoT realtime, cần chuẩn hóa Client ID + reconnect strategy ngay từ đầu để tránh nhiễu vận hành.
