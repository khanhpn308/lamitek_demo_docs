# Hướng Dẫn Triển Khai Docker Trên Linux (App + Database + InfluxDB)

- Mã tài liệu: DEPLOY-lamitek-001
- Phiên bản: 1.6.7
- Ngày cập nhật: 2026-04-08

## 1. Mục tiêu

Tài liệu này hướng dẫn triển khai production trên máy chủ Linux cho cả:

- `database_service` (MySQL)
- `influxdb_service` (InfluxDB)
- `app_service` (FastAPI + React/Nginx)

## 2. Nguồn mã nguồn trên GitHub (HTTPS)

| Vai trò                          | URL clone (HTTPS)                                    |
| -------------------------------- | ---------------------------------------------------- |
| Máy chủ ứng dụng (App)           | `https://github.com/khanhpn308/app_service.git`      |
| Máy chủ cơ sở dữ liệu (Database) | `https://github.com/khanhpn308/database_service.git` |

Tham chiếu repository công khai:

- [khanhpn308/app_service](https://github.com/khanhpn308/app_service) — FastAPI + React
- [khanhpn308/database_service](https://github.com/khanhpn308/database_service) — MySQL

## 3. Giai đoạn A — Máy chủ mới: clone mã nguồn

Cài `git` trước khi clone (ví dụ Amazon Linux):

```bash
sudo dnf install -y git
```

### 3.1 Trên máy chủ Database (chỉ cần repo `database_service`)

```bash
cd ~
git clone https://github.com/khanhpn308/database_service.git
cd database_service
```

### 3.2 Trên máy chỉnh App (chỉ cần repo `app_service`)

```bash
cd ~
git clone https://github.com/khanhpn308/app_service.git
cd app_service
```

### 3.3 Cùng một máy (thử nghiệm / môi trường nhỏ)

```bash
cd ~
git clone https://github.com/khanhpn308/database_service.git
git clone https://github.com/khanhpn308/app_service.git
```

Sau khi clone, tiếp tục cài Docker và triển khai theo các mục bên dưới.

## 4. Cài đặt Security Group (AWS EC2)

Áp dụng khi hai máy chủ **Application** và **Database** là **hai instance EC2** (hoặc tương đương) trong **cùng VPC** (khuyến nghị để giao tiếp qua **Private IP**). Điều chỉnh cổng theo đúng `docker-compose.yml` và `.env` thực tế.

### 4.1 Inbound và Outbound — ý nghĩa

Trên AWS, mỗi Security Group có hai nhóm quy tắc độc lập:

| Khái niệm (AWS)    | Tiếng Việt thường dùng    | Ý nghĩa                                                                                                                                                                                                                    |
| ------------------ | ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Inbound rules**  | Quy tắc **vào** (ingress) | Kiểm soát **lưu lượng đi vào** instance: **ai** được phép **kết nối tới** máy này, trên **cổng / giao thức** nào. Ví dụ: mở `443` để người dùng truy cập HTTPS; mở `3306` trên máy DB **chỉ** từ SG (hoặc IP) của máy App. |
| **Outbound rules** | Quy tắc **ra** (egress)   | Kiểm soát **lưu lượng đi ra** khỏi instance: máy này được phép **gửi** kết nối tới đâu (internet, dịch vụ AWS, máy khác trong VPC). Ví dụ: `git pull`, `docker pull`, truy vấn DNS, gọi API bên ngoài.                     |

**Gợi ý đọc bảng dưới:** các bảng cổng (SSH, HTTP, MySQL…) ở mục **4.2** và **4.3** là **Inbound**, trừ khi ghi rõ Outbound.

**Lưu ý (AWS):** Security Group là **stateful** — khi một luồng Inbound được phép (ví dụ client vào `443`), **phản hồi** của cùng kết nối đó thường **không** cần thêm rule Outbound “đối xứng” từng gói. Outbound vẫn quan trọng khi **chính server chủ động** mở kết nối mới ra ngoài (pull image, cập nhật OS).

**Ví dụ luồng App → DB:** máy App **Outbound** tới `DB_HOST:3306`; máy DB phải có **Inbound** cho `3306` từ nguồn là SG (hoặc IP private) của App. Thiếu một trong hai phía, kết nối sẽ thất bại.

### 4.2 Security Group gắn máy chủ Application

**Quy tắc Inbound (ingress):**

| Loại           | Cổng / Giao thức | Nguồn (Source)                            | Ghi chú                                                                                |
| -------------- | ---------------- | ----------------------------------------- | -------------------------------------------------------------------------------------- |
| SSH            | TCP `22`         | IP quản trị cố định, VPN, hoặc bastion    | Tránh mở `0.0.0.0/0` cho SSH trên môi trường production nếu có thể.                    |
| HTTP           | TCP `80`         | `0.0.0.0/0` hoặc mạng khách hàng nội bộ   | Phục vụ Nginx (frontend) theo `FRONTEND_HTTP_PORT` (thường map `80`).                  |
| HTTPS          | TCP `443`        | `0.0.0.0/0` hoặc hạn chế theo nhu cầu     | Khi bật TLS trên app server.                                                           |
| API (tùy chọn) | TCP `8000`       | Chỉ khi publish trực tiếp backend ra host | Mặc định nhiều stack chỉ expose qua Nginx; khi đó **không** cần mở `8000` ra internet. |

**Quy tắc Outbound (egress):** thường giữ mặc định _Allow all_ (để `git pull`, `docker pull`, cập nhật gói, DNS). Có thể thu hẹp theo chính sách doanh nghiệp.

### 4.3 Security Group gắn máy chủ Database

**Quy tắc Inbound (ingress):**

| Loại  | Cổng / Giao thức | Nguồn (Source)                                                                     | Ghi chú                                                                      |
| ----- | ---------------- | ---------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| SSH   | TCP `22`         | IP quản trị / bastion                                                              | Giống nguyên tắc máy App.                                                    |
| MySQL | TCP `3306`       | **Chỉ** Security Group của máy Application **hoặc** Private IP của máy App (`/32`) | **Không** mở `3306` ra `0.0.0.0/0`. Cho phép đúng nguồn từ máy chạy backend. |

**Quy tắc Outbound (egress):** mặc định cho phép (hoặc chỉ mở những đích cần cho backup/monitoring nếu tách rule).

### 4.4 Nguyên tắc tóm tắt

- DB chỉ lộ cổng MySQL **nội bộ VPC** tới App (hoặc qua SG reference).
- App cần HTTP/HTTPS phía user; backend `8000` chỉ mở public khi thiết kế có expose trực tiếp.
- Sau khi sửa Security Group, kiểm tra lại từ máy App: `telnet`/`nc` tới `DB_HOST:DB_PORT` hoặc dùng client MySQL.

### 4.5 Liên hệ với biến `.env` trên App

- `DB_HOST` trên máy App nên là **Private IP** của máy DB (cùng VPC) hoặc endpoint nội bộ — khớp với rule cho phép `3306` từ SG/IP App.
- Nếu buộc dùng Public IP cho DB (không khuyến nghị), vẫn phải siết chặt nguồn trên SG DB, không để MySQL mở toàn cầu.

## 5. Chuẩn bị máy chủ Linux (Docker)

Mục tiêu: Docker Engine + Compose plugin + **Buildx đủ mới**, tránh lỗi khi chạy `docker compose up --build` trên `app_service`:

`compose build requires buildx 0.17.0 or later`

Nguyên nhân thường gặp: gói Docker từ distro kèm **Buildx cũ** (ví dụ `0.12.x`) trong khi Docker Compose mới yêu cầu **Buildx ≥ 0.17.0**.

### 5.1 Yêu cầu phiên bản tối thiểu (nên kiểm tra trước khi triển khai)

| Thành phần              | Ghi chú                                                      |
| ----------------------- | ------------------------------------------------------------ |
| Docker Engine           | Khuyến nghị **23.x trở lên** (kiểm tra: `docker --version`). |
| Docker Compose (plugin) | Khuyến nghị **v2.20+** (kiểm tra: `docker compose version`). |
| **Docker Buildx**       | **Bắt buộc ≥ 0.17.0** (kiểm tra: `docker buildx version`).   |

### 5.2 Cài Docker (ví dụ Amazon Linux 2023)

```bash
sudo dnf update -y
sudo dnf install -y docker
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ec2-user
```

Nếu dùng **Amazon Linux 2** hoặc policy nội bộ yêu cầu Docker CE từ kho chính thức Docker, làm theo tài liệu Docker Engine cho từng distro: [Install Docker Engine](https://docs.docker.com/engine/install/).

### 5.3 Cài Docker Compose plugin (V2)

Dùng bản binary ổn định từ GitHub (tránh Compose quá cũ):

```bash
sudo mkdir -p /usr/libexec/docker/cli-plugins
sudo curl -SL "https://github.com/docker/compose/releases/latest/download/docker-compose-linux-$(uname -m)" -o /usr/libexec/docker/cli-plugins/docker-compose
sudo chmod +x /usr/libexec/docker/cli-plugins/docker-compose
docker compose version
```

### 5.4 Cài hoặc nâng cấp Docker Buildx (≥ 0.17.0)

**Luôn kiểm tra sau bước trên:**

```bash
docker buildx version
```

Nếu số phiên bản **< 0.17.0**, cài bản Buildx mới vào thư mục plugin. **Chỉ dùng tag có thật** trên [releases buildx](https://github.com/docker/buildx/releases) — ví dụ `v0.17.0` đáp ứng ≥ 0.17.0; không phải mọi bản `0.17.x` đều được phát hành (tag sai sẽ gây **404** khi `curl`).

**Kiến trúc x86_64 (hầu hết EC2 Intel/AMD):**

```bash
sudo mkdir -p /usr/libexec/docker/cli-plugins
sudo curl -fSL "https://github.com/docker/buildx/releases/download/v0.17.0/buildx-v0.17.0.linux-amd64" -o /usr/libexec/docker/cli-plugins/docker-buildx
sudo chmod +x /usr/libexec/docker/cli-plugins/docker-buildx
```

**Kiến trúc ARM64 (ví dụ Graviton):** đổi tên file thành `buildx-v0.17.0.linux-arm64` trong URL trên.

Trên **máy chủ production**, ưu tiên cài Buildx vào **`/usr/libexec/docker/cli-plugins/`** (mục **Kiến trúc x86_64** ở trên). Cài trong `~/.docker/cli-plugins` chỉ nên dùng khi chắc chắn Docker nhận plugin user; nếu file tải sai, Docker có thể báo `docker: 'buildx' is not a docker command` (xem mục 5.6).

**Lưu ý:** `docker buildx install` chỉ gắn builder mặc định, **không** tự nâng phiên bản Buildx.

### 5.5 Xác nhận trước khi chạy `docker compose up --build`

```bash
docker --version
docker compose version
docker buildx version
```

Đảm bảo dòng Buildx hiển thị **0.17.0 trở lên**, sau đó mới triển khai `app_service`.

Đăng xuất và đăng nhập lại (hoặc `newgrp docker`) sau khi thêm user vào nhóm `docker` nếu cần quyền không dùng `sudo`.

### 5.6 Xử lý sự cố: `docker: 'buildx' is not a docker command`

Sau khi tải Buildx tay, nếu `docker buildx version` báo **không có lệnh `buildx`**, thường do một trong các nguyên nhân sau:

1. **File tải về không phải binary** (ví dụ GitHub trả HTML do lỗi mạng, rate limit, **tag/release không tồn tại → 404**, URL sai) — file rất nhỏ (vài byte hoặc vài KB) hoặc `file docker-buildx` không báo `ELF`.
2. **Plugin user (`~/.docker/cli-plugins/docker-buildx`) bị lỗi** — Docker có thể ưu tiên file này và **không** dùng bản hệ thống, dẫn tới mất hẳn `buildx` hợp lệ.

**Cách xử lý gợi ý:**

```bash
# Kiểm tra file cũ (nếu có)
ls -la ~/.docker/cli-plugins/docker-buildx 2>/dev/null
file ~/.docker/cli-plugins/docker-buildx 2>/dev/null

# Xóa plugin user bị lỗi (nếu tồn tại)
rm -f ~/.docker/cli-plugins/docker-buildx

# Cài lại Buildx vào thư mục hệ thống (x86_64 EC2)
sudo mkdir -p /usr/libexec/docker/cli-plugins
sudo curl -fSL "https://github.com/docker/buildx/releases/download/v0.17.0/buildx-v0.17.0.linux-amd64" \
  -o /usr/libexec/docker/cli-plugins/docker-buildx
sudo chmod +x /usr/libexec/docker/cli-plugins/docker-buildx

docker buildx version
```

Tham số **`-f`** (`--fail`) giúp `curl` báo lỗi nếu tải không thành công, tránh lưu nhầm trang HTML.

Nếu vẫn lỗi, kiểm tra phiên bản Compose: `docker compose version` — bản **quá cũ** (ví dụ v2.1.x) nên **nâng** theo mục **5.3**, rồi chạy lại `docker compose up -d --build`.

## 6. Triển khai `database_service`

```bash
cd ~/database_service
cp .env.example .env
```

Cập nhật tối thiểu trong `.env`:

- `MYSQL_ROOT_PASSWORD`
- `MYSQL_DATABASE`
- `MYSQL_USER`
- `MYSQL_PASSWORD`
- `MYSQL_PORT` (mặc định `3306`)

Khởi chạy:

```bash
docker compose up -d
docker compose ps
docker compose logs -f db
```

## 7. Triển khai `app_service`

```bash
cd ~/app_service
cp .env.example .env
```

**Lưu ý:** Stack `app_service` đã bao gồm **Mosquitto broker** — cấu hình nằm ở `app_service/mosquitto.conf`. Backend sẽ chạy `depends_on` Mosquitto với điều kiện `service_healthy`.

### 7.1 Giải thích log Mosquitto khi khởi chạy

Khi chạy `docker compose up -d --build`, bạn sẽ thấy log từ Mosquitto có nhiều dòng like:

```
1775703804: New connection from 127.0.0.1:54722 on port 1883.
1775703804: New client connected from 127.0.0.1:54722 as auto-... (p2, c1, k60).
1775703805: Client auto-... disconnected.
```

Đây là **hành vi bình thường**: Docker healthcheck kết nối tới broker mỗi 10 giây để kiểm tra sức khỏe (`netstat -tln | grep -q 1883`). Không phải lỗi.

### 7.2 Cấu hình Mosquitto (mosquitto.conf)

Cấu hình Mosquitto nằm ở `app_service/mosquitto.conf`. Nội dung mặc định:

```conf
# MQTT qua TCP (Cho ESP32, FastAPI)
listener 1883 0.0.0.0
allow_anonymous true

# MQTT qua WebSockets (Bắt buộc cho React Frontend)
listener 9001 0.0.0.0
protocol websockets
```

**Giải thích:**

- **`listener 1883 0.0.0.0`**: Broker lắng nghe MQTT TCP trên cổng 1883 (chuẩn MQTT), mở cho toàn bộ IP (`0.0.0.0`).
- **`allow_anonymous true`**: cho phép kết nối không xác thực. **Trên production**, nên đổi thành `false` và cấu hình username/password.
- **`listener 9001 ... websockets`**: Cổng 9001 cho WebSocket (React frontend dùng khi kết nối MQTT qua JS).

**Nếu muốn bật xác thực (khuyến nghị production):**

Thêm vào file `app_service/mosquitto.conf`:

```conf
allow_anonymous false
password_file /mosquitto/config/mosquitto.passwd
```

Tạo file `mosquitto.passwd` trên host bằng lệnh:

```bash
# Tạo file với user "iot_user"
docker run --rm -v $(pwd)/app_service:/mosquitto/config eclipse-mosquitto \
  mosquitto_passwd -c /mosquitto/config/mosquitto.passwd iot_user
# (Nhập mật khẩu khi được hỏi)
```

Gắn volume vào `docker-compose.yml`:

```yaml
mosquitto:
  ...
  volumes:
    - ./mosquitto.conf:/mosquitto/config/mosquitto.conf:ro
    - ./mosquitto.passwd:/mosquitto/config/mosquitto.passwd:ro
    ...
```

Cập nhật backend `.env`:

```dotenv
MQTT_USERNAME=iot_user
MQTT_PASSWORD=<password-được-set>
```

**Cấu hình nghe trên IP cụ thể (thay vì 0.0.0.0):**

Nếu muốn Mosquitto chỉ nghe trên interface riêng:

```conf
listener 1883 10.0.1.5
allow_anonymous true
```

Cập nhật backend `.env`:

```dotenv
MQTT_HOST=10.0.1.5
```

**Thêm logging chi tiết:**

```conf
log_dest file /mosquitto/log/mosquitto.log
log_dest stdout
log_timestamp true
log_type all
```

Kiểm tra log:

```bash
docker compose logs -f mosquitto
```

Cập nhật trong `.env` — bảng chú thích các thông số (theo `app_service/.env.example`):

| Biến                 | Ý nghĩa / Ghi chú                                                                                                                                                                                                                                                                                                                             |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `APP_NAME`           | Tên hiển thị/log cho backend (chuỗi mô tả ứng dụng API). Không ảnh hưởng URL; có thể giữ mặc định hoặc đổi cho đúng tên sản phẩm.                                                                                                                                                                                                             |
| `ENVIRONMENT`        | Nhãn môi trường chạy API (ví dụ `prod`, `staging`, `dev`). Dùng để phân biệt cấu hình và log; nên khớp thực tế triển khai.                                                                                                                                                                                                                    |
| `CORS_ORIGINS`       | **Origin** được phép trình duyệt gọi API (scheme + host + port). Phải trùng URL mà user mở app (ví dụ `http://47.129.182.197` hoặc `https://ten-mien.com`). `http://localhost` chỉ đúng khi test từ máy local; trên server production cần đổi sang URL public hoặc domain thật. Nhiều origin: cách nhau bằng dấu phẩy (theo quy ước backend). |
| `DB_HOST`            | Địa chỉ máy chủ MySQL. **`127.0.0.1` chỉ dùng khi MySQL chạy trên cùng máy với App.** Nếu DB là **máy riêng** (EC2 khác): dùng **Private IP** (cùng VPC, an toàn hơn) hoặc **Public IP**/DNS của server DB — và mở security group/firewall cho cổng `DB_PORT` từ IP máy App.                                                                  |
| `DB_PORT`            | Cổng MySQL (thường `3306`). Khớp với cổng MySQL thực tế trên server DB và với `MYSQL_PORT` ở `database_service` nếu map port ra host.                                                                                                                                                                                                         |
| `DB_USER`            | User MySQL backend dùng để đăng nhập (thường trùng `MYSQL_USER` trên DB server; tránh dùng `root` trên production nếu có thể).                                                                                                                                                                                                                |
| `DB_PASSWORD`        | Mật khẩu của `DB_USER`, trùng khớp cấu hình trên server database.                                                                                                                                                                                                                                                                             |
| `DB_NAME`            | Tên schema/database (ví dụ `iot`), khớp `MYSQL_DATABASE` ở `database_service`.                                                                                                                                                                                                                                                                |
| `API_URL`            | URL gốc mà **client/frontend** hoặc tài liệu tham chiếu dùng để gọi API (ví dụ `http://47.129.182.197:8000` hoặc domain qua reverse proxy). `http://localhost:8000` chỉ đúng khi dev local; trên EC2 nên dùng **public IP + cổng backend** hoặc URL HTTPS sau khi có domain.                                                                  |
| `FRONTEND_HTTP_PORT` | Cổng trên **máy chủ App** mà container Nginx (SPA) publish ra ngoài (mặc định `80`). User truy cập web qua `http://<public-ip>:<port>` nếu không dùng 443.                                                                                                                                                                                    |
| `JWT_SECRET`         | Chuỗi bí mật ký JWT đăng nhập. **Bắt buộc đổi** sang chuỗi dài, ngẫu nhiên, không commit lên Git; nếu lộ, token có thể bị giả mạo.                                                                                                                                                                                                            |
| `JWT_EXPIRE_MINUTES` | Thời gian hiệu lực token (phút). Ví dụ `10080` ≈ 7 ngày. Giảm nếu cần bảo mật cao hơn; tăng nếu chấp nhận phiên dài.                                                                                                                                                                                                                          |

**Lưu ý nhanh:** `DB_HOST=127.0.0.1` trên app server chỉ hợp lệ khi container DB hoặc MySQL cài **cùng host** với backend. Tách DB sang EC2 khác thì phải điền IP/hostname của **máy database**.

Với kiến trúc tách stack, `INFLUX_URL` nên trỏ tới service name `influxdb` trên mạng dùng chung `iot-net` (mặc định `http://influxdb:8086`).

Khởi chạy:

```bash
docker compose up -d --build
docker compose ps
docker compose logs -f backend
docker compose logs -f frontend
```

## 8. Triển khai `influxdb_service`

```bash
cd ~/influxdb_service
cp .env.example .env
```

Cập nhật tối thiểu trong `.env`:

- `INFLUX_INIT_USERNAME`
- `INFLUX_INIT_PASSWORD`
- `INFLUX_ORG`
- `INFLUX_BUCKET`
- `INFLUX_TOKEN`

Khởi chạy:

```bash
docker compose up -d
docker compose ps
docker compose logs -f influxdb
```

Kiểm tra nhanh API InfluxDB:

```bash
curl -i http://127.0.0.1:8086/health
```

## 9. Giai đoạn B — Server đang hoạt động: cập nhật mã từ Git

Khi có bản cập nhật mới trên GitHub, trên **từng máy chủ** tương ứng:

### 9.1 Máy chủ Database — kéo mã về (`git pull`)

```bash
cd ~/database_service
git fetch origin
git status
git pull origin main
```

Nếu `docker-compose.yml` thay đổi (image, port, biến môi trường), khởi động lại dịch vụ:

```bash
docker compose up -d
docker compose ps
```

### 9.2 Cập nhật schema MySQL sau khi có file `sql/` mới trên Git

**Quan trọng:** `git pull` chỉ cập nhật file trên đĩa máy chủ. MySQL trong Docker **không tự** chạy lại các script trong `sql/` khi volume dữ liệu (`mysql_data`) **đã tồn tại** từ trước.

- **Máy DB mới, volume trống (lần đầu `docker compose up`):** entrypoint MySQL chạy một lần các file trong `/docker-entrypoint-initdb.d` (thư mục `sql/` được mount vào đó) → bảng/schema theo repo được tạo **tự động**.
- **Máy DB đã chạy lâu, đã có dữ liệu:** kéo code mới có thêm/sửa `schema.sql` hoặc file migration (`004_*.sql`, `005_*.sql`, …) → bạn phải **áp dụng thủ công** lên database đang chạy (hoặc dựa vào migration trong backend `app_service` khi process backend **được khởi động lại** và kết nối được MySQL — xem mục 9.4).

Sau khi `git pull`, xem trong `sql/` có file migration mới nào (đọc comment đầu file để biết thứ tự và điều kiện). Ví dụ áp dụng migration (điều chỉnh user/mật khẩu/database cho đúng `.env`):

```bash
cd ~/database_service
# Nạp biến từ .env (nếu shell hỗ trợ)
set -a && source .env && set +a

docker compose exec -T db mysql -u"$MYSQL_USER" -p"$MYSQL_PASSWORD" "$MYSQL_DATABASE" < sql/004_device_ui_columns.sql
# Nếu tài liệu migration yêu cầu chạy thêm (ví dụ gỡ cột cũ):
# docker compose exec -T db mysql -u"$MYSQL_USER" -p"$MYSQL_PASSWORD" "$MYSQL_DATABASE" < sql/005_drop_last_reading.sql
# Migration topic MQTT theo từng thiết bị:
# docker compose exec -T db mysql -u"$MYSQL_USER" -p"$MYSQL_PASSWORD" "$MYSQL_DATABASE" < sql/006_add_device_topic.sql
```

Kiểm tra nhanh:

```bash
docker compose exec db mysql -u"$MYSQL_USER" -p"$MYSQL_PASSWORD" "$MYSQL_DATABASE" -e "SHOW COLUMNS FROM device;"
```

**Lưu ý:** `docker compose up -d` **không** thay thế bước trên — nó chỉ đảm bảo container MySQL chạy; không import lại toàn bộ `sql/` vào DB cũ.

### 9.3 Máy chủ App — kéo mã và build frontend

```bash
cd ~/app_service
git fetch origin
git status
git pull origin main
docker compose up -d --build
docker compose ps
```

Trong `docker-compose.yml` mặc định của `app_service`, **chỉ service `frontend`** có chỉ thị `build:` (Dockerfile). Lệnh `docker compose up -d --build` vì vậy **build lại image SPA/Nginx** khi `Dockerfile`, `package.json` hoặc mã React thay đổi. Service **`backend`** dùng image `python:3.12-slim` và **mount** thư mục `./backend` vào container — **không** có bước build image cho từng lần sửa file Python.

### 9.4 Bắt buộc: reset / khởi động lại backend sau khi cập nhật mã Python

**Vì sao cần:** Thư mục `backend/` trên máy chủ đã được `git pull` mới, nhưng process **uvicorn** trong container (thường **không** bật `--reload` trên production) **không tự nạp lại** module đã import. Nếu chỉ chạy `docker compose up -d --build`, Compose có thể **không** tạo lại container `backend` (image và cấu hình service không đổi) → backend vẫn hiển thị trạng thái **Up nhiều giờ**, vẫn chạy **mã cũ trong bộ nhớ**, dù file trên đĩa đã mới. Điều này dễ gây lỗi “đã pull code nhưng API/ghi DB vẫn sai”.

**Cách làm** (chọn một trong các cách; thực hiện **sau** `git pull` và tùy chọn `docker compose up -d --build`):

```bash
cd ~/app_service
docker compose restart backend
```

Hoặc ép tạo lại container backend:

```bash
docker compose up -d --force-recreate backend
```

Có thể kết hợp build frontend và recreate backend một lần:

```bash
docker compose up -d --build --force-recreate backend
```

Kiểm tra:

```bash
docker compose ps
docker compose logs backend --tail 80
```

Bạn nên thấy backend **vừa khởi động lại** (thời gian “Up” ngắn sau lệnh), và log không báo lỗi import/kết nối DB.

**Gợi ý:** Sau khi backend restart, migration trong code (ví dụ `app/core/db_migrate.py`) chạy lại **mỗi khi** process khởi động — hữu ích khi `DB_HOST` trỏ đúng máy database; vẫn nên kiểm tra schema bằng `SHOW COLUMNS` khi cần.

### 9.5 Thứ tự khuyến nghị khi có bản cập nhật đồng thời

1. Trên **máy Database:** `git pull` trong `database_service`, rồi **chạy các file migration SQL** cần thiết (mục 9.2) nếu volume MySQL đã có dữ liệu.
2. Trên **máy App:** `git pull` trong `app_service`, `docker compose up -d --build`, sau đó **`docker compose restart backend`** hoặc **`--force-recreate backend`** (mục 9.4), rồi xem log backend (`docker compose logs backend --tail 100`) để xác nhận kết nối DB và không lỗi migration.

### 9.6 Lưu ý

- Không ghi đè file `.env` khi merge/pull; nên backup `.env` trước khi `git pull` nếu repo có thay đổi mẫu env.
- Nếu nhánh mặc định trên GitHub không phải `main`, thay `main` bằng tên nhánh đúng (ví dụ `master`).
- Sao lưu (`mysqldump`) trước khi chạy migration trên production nếu thay đổi schema có rủi ro.

### 9.7 Checklist deploy nhanh trên EC2 (chạy một lần)

Mục tiêu: cập nhật `app_service`, restart backend để nạp mã Python mới, và verify nhanh các endpoint quan trọng.

```bash
# 0) SSH vào EC2 App
ssh -i <key.pem> ec2-user@<PUBLIC_IP_APP>

# 1) Cập nhật mã
cd ~/app_service
git fetch origin
git pull origin main

# 2) Build frontend + bảo đảm service chạy
docker compose up -d --build

# 3) Bắt buộc restart backend để nạp code Python mới
docker compose restart backend

# 4) Kiểm tra nhanh container
docker compose ps
docker compose logs backend --tail 80
```

Verify endpoint (ngay trên máy App):

```bash
# Health
curl -i http://127.0.0.1:8000/api/health

# DB health
curl -i http://127.0.0.1:8000/api/health/db
```

Verify endpoint mới liên quan quyền/xóa thiết bị:

```bash
# Lấy token admin (điền đúng username/password)
TOKEN=$(curl -s -X POST http://127.0.0.1:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"AD00000","password":"<ADMIN_PASSWORD>"}' | python3 -c "import sys, json; print(json.load(sys.stdin).get('access_token',''))")

# Kiểm tra danh sách user có field authorized_devices
curl -s http://127.0.0.1:8000/api/users \
  -H "Authorization: Bearer $TOKEN"

# Kiểm tra route xóa device đã có (không còn 405 Method Not Allowed).
# Dùng ID test/sandbox để tránh xóa nhầm dữ liệu production.
curl -i -X DELETE "http://127.0.0.1:8000/api/devices/<DEVICE_ID_TEST>" \
  -H "Authorization: Bearer $TOKEN"
```

Kỳ vọng:

- `/api/health` trả `200`.
- `/api/users` trả JSON có `authorized_devices` trên từng user (hoặc frontend đã fallback nếu backend cũ).
- `DELETE /api/devices/{id}` không trả `405`; kết quả hợp lệ thường là `204` (xóa thành công), `404` (không tồn tại), hoặc `403` (không đủ quyền).

### 9.8 Restart container khi có cập nhật

**Viết tắt:**

- **RCU** = **Restart Containers on Update**.

**Công dụng:**

- Đảm bảo process trong container nạp mã/cấu hình mới nhất sau `git pull` hoặc chỉnh `.env`.
- Tránh tình trạng container vẫn `Up` nhưng đang chạy code cũ trong bộ nhớ.

**Nguyên tắc khuyến nghị:**

- Chỉ restart service bị ảnh hưởng để giảm downtime.
- Khi đổi biến môi trường, ưu tiên `--force-recreate` thay vì chỉ `restart`.

#### 9.8.1 `app_service`

```bash
cd ~/app_service

# Cập nhật mã
git fetch origin
git pull origin main

# Build lại frontend nếu có thay đổi FE/Dockerfile
docker compose up -d --build frontend

# Backend: nạp mã Python mới
docker compose restart backend

# Nếu có đổi .env hoặc compose cho backend
docker compose up -d --force-recreate backend

docker compose ps
docker compose logs backend --tail 80
```

#### 9.8.2 `database_service`

```bash
cd ~/database_service
git fetch origin
git pull origin main

# Nếu chỉ cập nhật script vận hành: đảm bảo service chạy
docker compose up -d

# Nếu đổi biến môi trường/cấu hình container DB
docker compose up -d --force-recreate db

docker compose ps
docker compose logs db --tail 80
```

#### 9.8.3 `influxdb_service`

```bash
cd ~/influxdb_service
git fetch origin
git pull origin main

# Đảm bảo service chạy
docker compose up -d

# Nếu đổi .env hoặc compose cho InfluxDB
docker compose up -d --force-recreate influxdb

docker compose ps
docker compose logs influxdb --tail 80
```

**Lưu ý quan trọng:**

- Không dùng `docker compose down -v` trên production trừ khi chủ động xoá dữ liệu volume.
- Với MySQL/InfluxDB, thay đổi file SQL hoặc bootstrap env không tự áp lại vào volume cũ.

## 10. Kiểm tra sau triển khai

- API health: `http://<server-ip>:8000/api/health` (nếu backend được publish port)
- Frontend: `http://<server-ip>:<FRONTEND_HTTP_PORT>`
- DB kết nối nội bộ: kiểm tra qua log backend và health endpoint DB.

## 11. HTTPS (khuyến nghị)

Hướng dẫn từng bước (DNS, Security Group, Certbot, `docker-compose.override.yml`, CORS, gia hạn chứng chỉ): **`docs/deployment/https-setup.md`**.

Tóm tắt: dùng Let’s Encrypt (Certbot) và file mẫu `app_service/nginx/prod.https.conf`. Có thể tham khảo thêm ghi chú vận hành ở `deloy.md` (nếu có trong repo).

## 12. Lưu ý vận hành

- Không commit file `.env`.
- Dùng mật khẩu mạnh cho MySQL, JWT secret và Influx token.
- Bật backup định kỳ cho dữ liệu DB và InfluxDB.
- Giám sát `docker compose logs` và tài nguyên hệ thống khi build image frontend; sau khi sửa mã Python trong `backend/`, nhớ **restart backend** (mục 9.4).

## 13. Tài liệu bổ sung

- Thao tác Docker chi tiết trên máy chủ Linux (build, up/down, log, exec, prune): `docs/deployment/docker-linux-tutorial.md`.
- Thao tác MySQL trên máy chủ (Docker `database_service`, root/user, truy vấn, backup): `docs/deployment/mysql-linux-operations.md`.
- Cấu hình HTTPS (TLS) cho `app_service`: `docs/deployment/https-setup.md`.
