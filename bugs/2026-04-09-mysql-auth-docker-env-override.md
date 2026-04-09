# Bug Log - Xung đột xác thực MySQL trong Docker

## Thông tin chung

- Thời gian hoàn tất: 2026-04-09
- Khu vực bị ảnh hưởng: database_service, app_service
- Mức độ: Blocker (backend không thể kết nối DB)

## Mô tả lỗi chính xác

Trong quá trình chạy hệ thống bằng Docker Compose, gặp 2 nhóm lỗi liên tiếp:

1. Đăng nhập root MySQL thất bại:

- Lệnh: docker compose exec db mysql -uroot -pkhanh iot
- Lỗi: ERROR 1045 (28000): Access denied for user 'root'@'localhost'

2. Backend không kết nối được DB bằng tài khoản ứng dụng:

- Log backend: Database not ready (x/60): (pymysql.err.OperationalError) (1045, "Access denied for user 'iot_user'@'172.20.0.5' (using password: YES)")

Nguyên nhân gốc:

- Biến môi trường trong CMD đang ghi đè giá trị trong file .env.
- docker compose config cho thấy MYSQL*ROOT_PASSWORD và MYSQL_PASSWORD đang là giá trị cũ (CHANGE_ME*...), không phải giá trị vừa sửa trong file .env.
- Sau khi root đã vào được, tài khoản iot_user vẫn sai password hoặc host privilege so với backend.
- Có thêm độ lệch tên schema: script tạo demo_iot trong khi backend dùng DB_NAME=iot.

## Dấu hiệu nhận biết

- docker compose config không khớp với file .env đang mở.
- Root login fail dù đã down -v và up -d lại.
- Backend lặp lại "Database not ready" kèm lỗi 1045 cho iot_user.
- Đăng nhập vào iot rồi SHOW TABLES trả về rỗng.

## Các bước giải quyết (step by step)

1. Kiểm tra cấu hình Compose thực tế:

- Chạy: docker compose config
- Xác nhận giá trị environment đang bị ghi đè.

2. Kiểm tra và xóa biến môi trường đang ghi đè trong CMD:

- Kiểm tra: set MYSQL_ROOT_PASSWORD, set MYSQL_PASSWORD, set MYSQL_DATABASE...
- Xóa trong session hiện tại:
  - set MYSQL_ROOT_PASSWORD=
  - set MYSQL_DATABASE=
  - set MYSQL_USER=
  - set MYSQL_PASSWORD=
  - set MYSQL_PORT=

3. Ép Compose đọc đúng file env:

- Chạy trong database_service:
  - docker compose --env-file .env down -v
  - docker compose --env-file .env up -d
  - docker compose --env-file .env config
- Xác nhận MYSQL_ROOT_PASSWORD đã là khanh.

4. Thử lại đăng nhập root:

- docker compose --env-file .env exec db mysql -uroot -pkhanh iot
- Kết quả: login thành công.

5. Sửa quyền hoặc mật khẩu tài khoản ứng dụng trong MySQL:

- Tạo hoặc đổi mật khẩu và cấp quyền:
  - CREATE USER IF NOT EXISTS 'iot_user'@'%' IDENTIFIED BY 'CHANGE_ME_APP_PASSWORD';
  - ALTER USER 'iot_user'@'%' IDENTIFIED BY 'CHANGE_ME_APP_PASSWORD';
  - GRANT ALL PRIVILEGES ON iot.\* TO 'iot_user'@'%';
  - FLUSH PRIVILEGES;
- Test: mysql -uiot_user -pCHANGE_ME_APP_PASSWORD -D iot -e "SELECT 1;"

6. Đồng bộ backend env và restart backend:

- app_service/.env:
  - DB_USER=iot_user
  - DB_PASSWORD=CHANGE_ME_APP_PASSWORD
  - DB_NAME=iot (hoặc đổi toàn bộ sang demo_iot nếu chọn schema đó)
- Restart:
  - docker compose up -d --force-recreate backend
  - docker compose logs -f backend

7. Xử lý sai khác schema (nếu có):

- schema.sql đang tạo demo_iot.
- Cần thống nhất 1 tên duy nhất giữa SQL script và backend env (iot hoặc demo_iot).

## Kết quả sau xử lý

- Đăng nhập root thành công.
- Lỗi 1045 cho iot_user được xử lý sau khi đồng bộ password và privilege.
- Hệ thống có thể tiếp tục startup backend sau khi env và schema được thống nhất.

## Bài học rút ra

- Trên Windows CMD, biến môi trường session có thể ghi đè file .env của Docker Compose.
- Luôn kiểm tra docker compose config để biết giá trị thực tế container đang nhận.
- Luôn dùng --env-file .env cho các lệnh down, up, config trong luồng vận hành quan trọng.
- Không giải quyết được lỗi auth nếu chỉ reset volume mà chưa xử lý biến môi trường bị ghi đè.
