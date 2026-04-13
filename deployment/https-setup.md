# Hướng Dẫn Cấu Hình HTTPS Cho `app_service` (Nginx + Let’s Encrypt)

- Mã tài liệu: DEPLOY-lamitek-002
- Phiên bản: 1.0.0
- Ngày cập nhật: 2026-04-05

## 1. Mục tiêu

Bật **HTTPS** cho ứng dụng IoT (React + Nginx trong Docker, FastAPI phía sau), dùng chứng chỉ **Let’s Encrypt** (miễn phí, được trình duyệt tin cậy).

Tài liệu triển khai tổng thể: `docs/deployment/docker-linux-deployment.md` (Security Group, clone repo, Docker).

## 2. Điều kiện trước khi làm

| Yêu cầu | Ghi chú |
|--------|---------|
| **Tên miền** | Let’s Encrypt cấp chứng chỉ cho **domain**, không thay thế được việc truy cập bằng **chỉ IP** theo quy trình chuẩn này. Cần ít nhất một hostname (ví dụ `iot.example.com`). |
| **DNS** | Bản ghi **A** trỏ hostname tới **public IPv4** của EC2 (hoặc ALB nếu dùng load balancer). |
| **Security Group (máy App)** | **Inbound TCP 80** (xác thực Let’s Encrypt, redirect HTTP→HTTPS) và **TCP 443** (HTTPS). |
| **Mã nguồn** | Trong repo `app_service` đã có `nginx/prod.https.conf` (redirect 80→443, TLS 443, proxy `/api/` và `/ws/` tới backend). |

## 3. Luồng tổng quát

1. Trỏ DNS domain → IP máy chủ.
2. Cài **Certbot** trên EC2, lấy chứng chỉ (thường dùng chế độ **standalone** — cần **giải phóng cổng 80** lúc cấp phát).
3. Gắn file `fullchain.pem` / `privkey.pem` vào container **frontend** (Nginx) và dùng `prod.https.conf` thay cho `prod.conf`.
4. Cập nhật **`.env`** (CORS, v.v.) sang URL `https://...`.
5. Thiết lập **gia hạn** chứng chỉ và reload Nginx.

## 4. Chuẩn bị file Nginx HTTPS

File mẫu: `app_service/nginx/prod.https.conf`.

Thay **cả hai** chỗ `YOUR_DOMAIN` bằng hostname thật (ví dụ `iot.example.com`), trùng với domain dùng trong Certbot (`-d`).

Trên máy chủ có thể:

- Sửa trực tiếp bản sao trong thư mục clone, **hoặc**
- Sao chép: `cp nginx/prod.https.conf nginx/prod.https.active.conf`, chỉnh `YOUR_DOMAIN` trong file mới và dùng đường dẫn file này ở bước 6.

## 5. Lấy chứng chỉ Let’s Encrypt (Certbot standalone)

Ví dụ **Amazon Linux 2023**:

```bash
sudo dnf install -y certbot
```

Chế độ **standalone** bắt Certbot tự lắng nghe cổng **80**. Cần **dừng container đang chiếm 80** (thường là service `frontend`):

```bash
cd ~/app_service   # hoặc đường dẫn thư mục chứa docker-compose.yml
docker compose stop frontend
```

Cấp chứng chỉ (thay domain và email):

```bash
sudo certbot certonly --standalone -d iot.example.com --email admin@example.com --agree-tos --non-interactive
```

Sau khi xong, khởi động lại stack (sau bước 6 đã cấu hình volume):

```bash
docker compose start frontend   # tạm thời; sau bước 6 dùng up -d đầy đủ
```

Đường dẫn mặc định:

- `/etc/letsencrypt/live/iot.example.com/fullchain.pem`
- `/etc/letsencrypt/live/iot.example.com/privkey.pem`

## 6. Gắn chứng chỉ và cấu hình HTTPS vào Docker Compose

File `app_service/docker-compose.yml` mặc định chỉ map cổng **80** cho `frontend` và dùng `nginx/prod.conf` được **COPY** trong image.

Để bật HTTPS **không sửa Dockerfile**, nên:

- Map thêm **443:443**
- **Gắn volume** ghi đè `/etc/nginx/conf.d/default.conf` bằng bản `prod.https.conf` đã đổi domain
- **Gắn volume** hai file chứng chỉ vào `/etc/nginx/ssl/` (khớp `prod.https.conf`)

Cách gọn: tạo **`docker-compose.override.yml`** cạnh `docker-compose.yml` (Docker Compose tự merge). Thay `iot.example.com` và tên file conf cho đúng:

```yaml
services:
  frontend:
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/prod.https.conf:/etc/nginx/conf.d/default.conf:ro
      - /etc/letsencrypt/live/iot.example.com/fullchain.pem:/etc/nginx/ssl/fullchain.pem:ro
      - /etc/letsencrypt/live/iot.example.com/privkey.pem:/etc/nginx/ssl/privkey.pem:ro
```

**Lưu ý:** Khi `ports` được khai báo trong `override`, Compose **thay thế** danh sách `ports` của service `frontend` — không còn dùng biến `FRONTEND_HTTP_PORT` cho service này; cố định **80** và **443** cho HTTPS chuẩn.

Áp dụng:

```bash
docker compose up -d
```

Kiểm tra:

```bash
docker compose ps
curl -sI https://iot.example.com | head -n 5
```

## 7. Biến môi trường (`.env` trên máy chủ)

Cập nhật tối thiểu:

| Biến | Gợi ý |
|------|--------|
| `CORS_ORIGINS` | `https://iot.example.com` (không dùng `http://` hoặc IP công khai nếu đã chuyển hẳn sang HTTPS). |
| `API_URL` | Ghi chú nội bộ; nếu frontend build dùng `VITE_API_URL`, cần rebuild image sau khi đổi — thường nên để API qua cùng host qua Nginx (`/api/`) để tránh lệch HTTP/HTTPS. |

Sửa `.env` rồi:

```bash
docker compose up -d --build
```

(chỉ bắt buộc `--build` nếu thay đổi biến build-time của frontend.)

## 8. Gia hạn chứng chỉ và reload Nginx

Let’s Encrypt có chu kỳ ngắn. Kiểm tra gia hạn:

```bash
sudo certbot renew --dry-run
```

Sau mỗi lần renew thành công, cần **reload Nginx** trong container để đọc lại file (đường dẫn mount thường cập nhật tại chỗ):

```bash
docker compose exec frontend nginx -s reload
```

Có thể đăng ký **hook** sau renew (tuỳ bản Certbot / hệ điều hành), ví dụ script gọi `docker compose exec frontend nginx -s reload`.

## 9. Xử lý sự cố thường gặp

| Hiện tượng | Hướng xử lý |
|------------|-------------|
| Certbot báo cổng 80 đang bận | Dừng `frontend` hoặc mọi tiến trình đang giữ `:80`, chạy lại `certbot`. |
| `502` / không vào được HTTPS | Kiểm tra volume mount đúng đường dẫn file `.pem`, domain trong `prod.https.conf` trùng cert, `docker compose logs frontend`. |
| Trình duyệt vẫn “Not secure” | Truy cập đúng `https://domain`, không chỉ IP; kiểm tra chứng chỉ hết hạn (`openssl s_client` hoặc công cụ online). |
| Mixed content | Đảm bảo SPA không gọi API bằng `http://` khi trang đang là `https://`. |

## 10. Phương án thay thế (tóm tắt)

- **AWS Application Load Balancer + ACM:** TLS kết thúc tại ALB, EC2 chỉ nhận HTTP phía sau — phù hợp môi trường cần HA và quản lý chứng chỉ tập trung.
- **Chứng chỉ tự ký:** chỉ phục vụ thử nghiệm nội bộ; trình duyệt vẫn cảnh báo, không thay được trải nghiệm “tin cậy” như Let’s Encrypt + domain.

---

*Tài liệu này mô tả cấu hình phổ biến trên một máy EC2 đơn; điều chỉnh đường dẫn và tên domain khi áp dụng.*
