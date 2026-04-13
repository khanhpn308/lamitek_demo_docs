# Hướng Dẫn Thao Tác Docker Trên Máy Chủ Linux

- Mã tài liệu: DEPLOY-lamitek-004
- Phiên bản: 1.1.0
- Ngày cập nhật: 2026-04-05

## 1. Mục tiêu

Tài liệu này tóm tắt các lệnh Docker và Docker Compose thường dùng khi vận hành `app_service` và `database_service` trên Linux (build, khởi động, dừng, xem log, gỡ lỗi).

**Quy ước:** Trong tài liệu dùng lệnh `docker compose` (plugin V2). Nếu máy chỉ có bản cũ `docker-compose` (V1), thay bằng `docker-compose` với cùng tham số.

**Ngữ cảnh:** Hầu hết lệnh Compose cần chạy **trong thư mục** có file `docker-compose.yml` (ví dụ `~/app_service` hoặc `~/database_service`).

---

## 2. Kiểm tra Docker đã sẵn sàng

```bash
docker --version
docker compose version
docker info
```

Kiểm tra quyền user với Docker (không cần `sudo`):

```bash
docker ps
```

Nếu lỗi permission, thêm user vào nhóm `docker` và đăng nhập lại:

```bash
sudo usermod -aG docker "$USER"
```

---

## 3. Docker Compose — Khởi động và dừng stack

### 3.1 Khởi động nền (detached)

```bash
docker compose up -d
```

### 3.2 Khởi động và build lại image (sau khi đổi Dockerfile hoặc mã nguồn)

```bash
docker compose up -d --build
```

Chỉ build lại một service (ví dụ `frontend`):

```bash
docker compose build frontend
docker compose up -d frontend
```

### 3.3 Dừng container (giữ container và volume)

```bash
docker compose stop
```

### 3.4 Dừng và xóa container (không xóa volume mặc định)

```bash
docker compose down
```

### 3.5 Dừng và xóa cả volume (cẩn thận — có thể mất dữ liệu DB)

```bash
docker compose down -v
```

Chỉ dùng khi bạn **chắc chắn** muốn xóa dữ liệu volume đã gắn với compose project.

### 3.6 Khởi động lại một hoặc nhiều service

```bash
docker compose restart
docker compose restart backend
```

---

## 4. Trạng thái, log và shell vào container

### 4.1 Danh sách container đang chạy

```bash
docker compose ps
docker ps
```

### 4.2 Log (theo dõi realtime)

```bash
docker compose logs -f
docker compose logs -f db
docker compose logs -f backend
```

Log gần đây (ví dụ 200 dòng cuối):

```bash
docker compose logs --tail=200 backend
```

### 4.3 Vào shell trong container (debug)

```bash
docker compose exec backend sh
docker compose exec db bash
```

Thoát: gõ `exit` hoặc `Ctrl+D`.

---

## 5. Build image (docker compose build)

Build toàn bộ service có `build:` trong `docker-compose.yml`:

```bash
docker compose build
```

Build không dùng cache (khi cần build sạch):

```bash
docker compose build --no-cache
```

Build một service cụ thể:

```bash
docker compose build frontend
```

---

## 6. Lệnh Docker tổng quát (ngoài Compose)

### 6.1 Liệt kê image

```bash
docker images
```

### 6.2 Xóa image không dùng (dọn dẹp)

```bash
docker image prune
```

### 6.3 Tài nguyên realtime

```bash
docker stats
```

### 6.4 Inspect container

```bash
docker inspect <container_id_or_name>
```

---

## 7. Dọn dẹp hệ thống (cẩn thận)

Các lệnh sau có thể xóa dữ liệu không còn tham chiếu; chỉ chạy khi hiểu rõ hậu quả.

```bash
docker system df
docker container prune
docker image prune -a
docker volume prune
```

**Không** chạy `docker volume prune` trên máy DB production nếu chưa backup.

---

## 8. Ánh xạ cổng và biến môi trường

- Cổng publish được khai báo trong `docker-compose.yml` và/hoặc biến trong `.env` (ví dụ `MYSQL_PORT`, `FRONTEND_HTTP_PORT`).
- Sau khi sửa `.env`, thường cần:

```bash
docker compose up -d
```

Hoặc recreate:

```bash
docker compose up -d --force-recreate
```

---

## 9. Gợi ý workflow vận hành ngắn

| Mục đích | Lệnh gợi ý |
|----------|------------|
| Triển khai lần đầu | `docker compose up -d --build` |
| Cập nhật code + build lại | `git pull` → `docker compose up -d --build` |
| Xem lỗi nhanh | `docker compose logs -f --tail=100 backend` |
| Tạm dừng không hủy volume | `docker compose stop` |
| Tắt hẳn stack (giữ volume) | `docker compose down` |
| Kiểm tra tài nguyên | `docker stats` |

---

## 10. MySQL trong container `database_service` (tóm tắt)

Vào MySQL trên **cùng máy** với stack DB (thư mục `~/database_service`):

```bash
cd ~/database_service
docker compose exec db mysql -uroot -p
```

Với user ứng dụng (thay user/database theo `.env`):

```bash
docker compose exec db mysql -u iot_user -p iot
```

**Lọc dữ liệu / thao tác nghiệp vụ** (SELECT, UPDATE, backup, giao dịch): xem **`docs/deployment/mysql-linux-operations.md`**.

---

## 11. Tài liệu liên quan

- Triển khai end-to-end trên Linux: `docs/deployment/docker-linux-deployment.md`
- Thao tác MySQL trên Linux (root, user, lọc dữ liệu, cập nhật an toàn): `docs/deployment/mysql-linux-operations.md`
- HTTPS (TLS): `docs/deployment/https-setup.md`
