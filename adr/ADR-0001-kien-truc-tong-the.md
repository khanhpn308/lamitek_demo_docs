# ADR-0001: Chọn kiến trúc tổng thể React + FastAPI + MySQL

- **Trạng thái**: Draff
- **Ngày**: 2026-04-04
- **Người quyết định**: Nhóm kỹ thuật dự án

## Bối cảnh

Hệ thống cần:
- UI web cho quản trị và giám sát.
- API backend dễ phát triển nhanh.
- CSDL quan hệ cho dữ liệu user/device/authorization.
- Khả năng tích hợp MQTT.

## Quyết định

Chọn kiến trúc:
- Frontend: React (Vite).
- Backend: FastAPI.
- CSDL: MySQL.
- Giao tiếp FE-BE: REST JSON.

## Lý do

- FastAPI phù hợp phát triển API nhanh, rõ schema.
- React phù hợp UI dashboard và phân trang nghiệp vụ.
- MySQL phù hợp quan hệ dữ liệu và tính ổn định vận hành.

## Hệ quả

### Tích cực
- Tách lớp rõ frontend/backend.
- Dễ mở rộng API.
- Dễ tuyển dụng nhân sự phù hợp stack phổ biến.

### Tiêu cực
- Cần quản lý đồng bộ contract API giữa hai phía.
- Cần chiến lược realtime riêng cho dashboard (WS/SSE) khi mở rộng.
