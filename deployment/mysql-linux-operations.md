# Thao Tác MySQL Trên Máy Chủ Linux (`database_service`)

- Mã tài liệu: DEPLOY-lamitek-003
- Phiên bản: 1.0.0
- Ngày cập nhật: 2026-04-05

## 1. Mục tiêu

Hướng dẫn **vào MySQL từ máy chủ Linux** khi DB chạy bằng Docker Compose trong `database_service`: đăng nhập bằng **root** hoặc **user ứng dụng**, sau đó **xem / lọc / cập nhật** dữ liệu một cách an toàn.

Giả định: service MySQL trong compose có tên **`db`** (theo `database_service/docker-compose.yml`), biến trong `.env` khớp `database_service/.env.example` (`MYSQL_ROOT_PASSWORD`, `MYSQL_DATABASE`, `MYSQL_USER`, `MYSQL_PASSWORD`, `MYSQL_PORT`).

## 2. Chuẩn bị

- SSH vào **máy chủ đang chạy** `database_service`, vào thư mục repo (ví dụ `~/database_service`).
- Container DB đang chạy: `docker compose ps` (cột `STATE` `running` cho service `db`).
- Biết **tên database** (thường `MYSQL_DATABASE`, ví dụ `iot`). Nếu script trong `sql/` tạo schema tên khác (ví dụ `demo_iot`), dùng đúng tên đó trong `USE ...;` — kiểm tra bằng `SHOW DATABASES;`.

## 3. Vào MySQL bằng Docker (khuyến nghị)

Mọi lệnh dưới đây chạy **trên máy host**, trong thư mục có `docker-compose.yml` của `database_service`.

### 3.1 Với tài khoản **root**

Root dùng cho **quản trị** (tạo user, grant, sửa schema, xử lý sự cố).

**Cách 1 — MySQL hỏi mật khẩu (an toàn hơn cho lịch sử shell):**

```bash
docker compose exec db mysql -uroot -p
```

Nhập `MYSQL_ROOT_PASSWORD` khi được hỏi (ký tự không hiện — bình thường).

**Cách 2 — Truyền mật khẩu qua biến môi trường (chỉ dùng tạm, tránh log/script công khai):**

```bash
set -a && source .env && set +a
docker compose exec -T db mysql -uroot -p"$MYSQL_ROOT_PASSWORD"
```

**Cách 3 — Một lệnh SQL không tương tác (kiểm tra nhanh):**

```bash
docker compose exec -T db mysql -uroot -p"$MYSQL_ROOT_PASSWORD" -e "SELECT VERSION();"
```

(Trên một số shell, cần `export` biến từ `.env` thủ công thay cho `source`.)

### 3.2 Với tài khoản **ứng dụng** (`MYSQL_USER`)

Dùng để **mô phỏng quyền** giống backend (kiểm tra kết nối, truy vấn nghiệp vụ), **không** cần root cho thao tác đọc/ghi dữ liệu thông thường.

```bash
docker compose exec db mysql -u"${MYSQL_USER:-iot_user}" -p"${MYSQL_PASSWORD}" "${MYSQL_DATABASE:-iot}"
```

Hoặc chỉ `-p` rồi gõ mật khẩu:

```bash
docker compose exec db mysql -u iot_user -p iot
```

(Thay `iot_user` / `iot` theo `.env` thực tế.)

### 3.3 Vào MySQL từ **máy khác** (ví dụ máy App)

Khi đã cài client `mysql` trên host và **mở firewall / Security Group** tới cổng DB (thường `3306`):

```bash
mysql -h <DB_PRIVATE_OR_PUBLIC_IP> -P <MYSQL_PORT> -u iot_user -p iot
```

Chỉ dùng trên **mạng tin cậy**; trên production nên **không** mở `3306` ra internet — xem `docs/deployment/docker-linux-deployment.md` (Security Group).

## 4. Lệnh MySQL cơ bản sau khi đăng nhập

```sql
SHOW DATABASES;
USE `iot`;                    -- thay bằng MYSQL_DATABASE của bạn
SHOW TABLES;
SHOW CREATE TABLE `user`\G    -- ví dụ bảng user; tên `user` là từ khóa → nên có backtick
DESCRIBE `user`;
```

Thoát client: `exit` hoặc `Ctrl+D`.

## 5. Lọc và xem dữ liệu (SELECT)

### 5.1 Điều kiện, sắp xếp, giới hạn

```sql
USE `iot`;

-- Vài dòng mới nhất theo khóa (ví dụ)
SELECT user_id, username, status, role
FROM `user`
WHERE status = 'active'
ORDER BY user_id DESC
LIMIT 20;

-- Tìm theo chuỗi (partial match)
SELECT user_id, username, email
FROM `user`
WHERE username LIKE 'admin%';

-- Đếm theo nhóm
SELECT status, COUNT(*) AS cnt
FROM `user`
GROUP BY status;
```

### 5.2 Lọc theo thời gian / NULL (nếu cột tồn tại)

```sql
SELECT user_id, username, expired_at
FROM `user`
WHERE expired_at IS NULL OR expired_at > CURDATE();

SELECT device_id, devicename, location, device_type, status
FROM device
ORDER BY device_id;
```

Điều chỉnh tên bảng/cột cho **đúng schema** đang triển khai.

## 6. Các trường hợp thao tác dữ liệu thường gặp

| Mục đích | Gợi ý SQL / lưu ý |
|----------|-------------------|
| **Chỉ đọc / kiểm tra** | `SELECT` với `WHERE`, `LIMIT`, không `UPDATE`/`DELETE` trên production nếu không cần. |
| **Đếm / thống kê** | `COUNT(*)`, `GROUP BY`, `HAVING`. |
| **Sửa một bản ghi xác định** | `UPDATE ... WHERE user_id = ?` — **luôn có `WHERE`** để tránh sửa cả bảng. |
| **Vô hiệu hóa tài khoản (soft)** | `UPDATE user SET status = 'deactive' WHERE user_id = ?` (nếu schema có cột `status`). |
| **Xóa cứng** | `DELETE FROM ... WHERE ...` — rủi ro cao; ưu tiên backup trước (`mysqldump`). |
| **Thêm bản ghi thủ công** | `INSERT INTO ...` — chú ý khóa ngoại, `UNIQUE`, mật khẩu đã hash (không lưu plain text). |
| **Sao lưu nhanh trước khi sửa** | Trên host: `docker compose exec db mysqldump -uroot -p"$MYSQL_ROOT_PASSWORD" iot > backup_iot_$(date +%F).sql` |

### 6.1 Giao dịch (khi cần nhiều bước cùng lúc)

```sql
START TRANSACTION;
-- UPDATE / INSERT / DELETE
-- Kiểm tra kết quả bằng SELECT
COMMIT;
-- hoặc ROLLBACK; nếu sai
```

## 7. An toàn và thực hành tốt

- **Ưu tiên user ứng dụng** cho thao tác nghiệp vụ; **root** chỉ khi cần quản trị.
- Không lưu mật khẩu root vào script đẩy lên Git; hạn chế `-p'MậtKhẩu'` trong lệnh có thể lưu vào `history` (dùng `-p` tương tác hoặc `mysql_config_editor` nếu cần).
- Trên production: **backup định kỳ** trước khi `UPDATE`/`DELETE` hàng loạt.
- Tên bảng như `user` là **từ khóa** trong MySQL — luôn dùng **backtick**: `` `user` ``.

## 8. Tài liệu liên quan

- Triển khai DB + Docker: `docs/deployment/docker-linux-deployment.md` (mục `database_service`).
- Lệnh Docker/Compose tổng quát: `docs/deployment/docker-linux-tutorial.md`.
