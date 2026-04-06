# Bộ Tài Liệu Dự Án

Tài liệu được chuẩn hóa theo 3 nhóm định dạng:

- **IEEE SRS**: đặc tả yêu cầu phần mềm chi tiết.
- **OpenAPI-like**: mô tả hợp đồng API ở dạng YAML.
- **ADR (Architecture Decision Record)**: ghi nhận quyết định kiến trúc.

## Cấu trúc tài liệu

- `docs/srs/IEEE-SRS.md`: Đặc tả yêu cầu phần mềm (chuẩn IEEE).
- `docs/architecture/system-architecture.md`: Tài liệu kiến trúc hệ thống.
- `docs/design/ui-ux-design.md`: Tài liệu UI/UX.
- `docs/design/frontend-design-tokens.md`: Chuẩn typography và color token cho frontend.
- `docs/deployment/docker-linux-deployment.md`: Hướng dẫn deploy Docker trên Linux cho `app_service` và `database_service`.
- `docs/deployment/https-setup.md`: Hướng dẫn bật HTTPS (Let’s Encrypt, Nginx trong Docker, CORS, gia hạn chứng chỉ).
- `docs/deployment/docker-linux-tutorial.md`: Hướng dẫn thao tác Docker/Compose trên Linux (build, up/down, log, dọn dẹp).
- `docs/deployment/mysql-linux-operations.md`: Vào MySQL trên Linux (root/user Docker), lọc dữ liệu, cập nhật và sao lưu an toàn.
- `docs/api/openapi-like.yaml`: Đặc tả API theo kiểu OpenAPI.
- `docs/api/api-documentation.md`: Diễn giải API cho đội phát triển và kiểm thử.
- `docs/app_service-functions.md`: Danh sách function trong source `app_service/backend/app` và `app_service/src`.
- `docs/changelogs.md`: Nhật ký thay đổi tính năng / API liên quan tài liệu.
- `docs/guidelines/frontend-guidelines.md`: Quy định kiến trúc và coding guideline cho Frontend.
- `docs/guidelines/backend-guidelines.md`: Quy định kiến trúc và coding guideline cho Backend.
- `docs/guidelines/git-github-teamwork.md`: Hướng dẫn Git/GitHub khi làm việc nhóm (branch, fetch/pull, đồng bộ remote, xử lý conflict và các tình huống thường gặp).
- `docs/adr/ADR-0001-kien-truc-tong-the.md`: ADR kiến trúc tổng thể.
- `docs/adr/ADR-0002-chien-luoc-xac-thuc-phan-quyen.md`: ADR xác thực và phân quyền.
- `docs/adr/ADR-0003-chien-luoc-du-lieu-thiet-bi-realtime.md`: ADR dữ liệu realtime.

## Nguyên tắc cập nhật

- Cấu trúc repository chính thức (2026-04):
  - `app_service`: frontend + backend
  - `database_service`: MySQL + init script
- Mọi thay đổi endpoint phải cập nhật đồng thời:
  - `docs/api/openapi-like.yaml`
  - `docs/api/api-documentation.md`
- Mọi thay đổi quyết định kiến trúc bắt buộc tạo ADR mới trong `docs/adr`.
- Mọi thay đổi yêu cầu nghiệp vụ lớn phải phản ánh vào `docs/srs/IEEE-SRS.md`.
