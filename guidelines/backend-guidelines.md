# Quy Định Cấu Trúc Mã Nguồn Backend

- **Mã tài liệu**: BE-GUIDE-001
- **Phiên bản**: 1.0.0
- **Ngày cập nhật**: 2026-04-04

## 1. Kiến trúc phân lớp backend

Hiện trạng mã nguồn:

```text
app_service/backend/app/
  api/       # routers
  core/      # config, deps, security, db, mqtt, migrate
  models/    # SQLAlchemy models
  schemas/   # Pydantic schemas
```

Chuẩn mở rộng khuyến nghị:

```text
app_service/backend/app/
  api/
    routers/
  services/
  repositories/
  core/
  models/
  schemas/
```

## 2. Quy định lớp (layer)

### 2.1 Router layer (`api`)

- Nhận request, validate schema đầu vào.
- Gọi service nghiệp vụ.
- Trả response model.
- Không chứa truy vấn DB phức tạp kéo dài.

### 2.2 Service layer (`services`) - áp dụng khi tái cấu trúc

- Chứa logic nghiệp vụ và rule.
- Không phụ thuộc FastAPI `Request` hoặc `Response`.
- Điều phối repository và transaction.

### 2.3 Repository layer (`repositories`) - áp dụng khi tái cấu trúc

- Chứa truy vấn SQLAlchemy.
- Đóng gói truy xuất dữ liệu theo domain.
- Không chứa nghiệp vụ vượt khỏi phạm vi dữ liệu.

## 3. Middleware và dependency

- Auth dependency chuẩn:
  - `get_current_user`
  - `require_admin`
- CORS cấu hình tại app startup.
- Mọi endpoint cần auth phải khai báo dependency rõ ràng.

## 4. Quy định xác thực và phân quyền

- Dùng JWT Bearer.
- Payload tối thiểu gồm:
  - `sub` (username)
  - `uid`
  - `role`
  - `exp`
- Mật khẩu phải hash bằng bcrypt, không lưu plaintext.
- Admin-only endpoint bắt buộc dùng `require_admin`.

## 5. Quy định xử lý lỗi (exception handling)

- Dùng `HTTPException` với mã lỗi đúng ngữ nghĩa.
- Chuẩn status code:
  - `400`: dữ liệu/logic nghiệp vụ sai.
  - `401`: chưa xác thực, token sai/hết hạn.
  - `403`: không đủ quyền.
  - `404`: không tìm thấy.
  - `409`: trùng dữ liệu.
  - `503`: phụ thuộc ngoài chưa sẵn sàng.
- Thông điệp lỗi ngắn gọn, rõ nghĩa nghiệp vụ.

## 6. Quy định dữ liệu và migration

- Migration nhẹ hiện tại chạy khi startup (`db_migrate`).
- Không chỉnh tay schema production khi chưa có kế hoạch rollback.
- Khuyến nghị chuẩn hóa công cụ migration (Alembic) khi bước vào quy mô production.

## 7. Quy định bảo mật

- Không commit `.env`.
- Không hard-code secret production.
- Không log token và thông tin mật khẩu.
- Định kỳ rotate JWT secret theo chính sách vận hành.

## 8. Quy định kiểm thử

- Ưu tiên test theo cấp:
  - unit test cho service/repository.
  - integration test cho endpoint auth, users, devices.
- Mọi bug nghiêm trọng phải có test hồi quy.
- Với luồng realtime/telemetry, bắt buộc có test profiling độ trễ theo từng chặng (Node -> Gateway -> Server) và lưu được các trường timestamp/delay để phân tích hậu kiểm.
- Chuẩn báo cáo latency regression nên bao gồm ít nhất `mean`, `p95`, `p99`, `max` và tỉ lệ mẫu lỗi, tham chiếu quy trình tại `docs/testing/testcases.md` (Test Case 02).
