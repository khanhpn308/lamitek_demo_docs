# ADR-0003: Chiến lược dữ liệu realtime cho giám sát thiết bị

- **Trạng thái**: Draff (giai đoạn hiện tại)
- **Ngày**: 2026-04-04
- **Người quyết định**: Nhóm kỹ thuật dự án

## Bối cảnh

Dashboard yêu cầu dữ liệu gần realtime.
Hiện tại backend đã có MQTT subscriber để nhận dữ liệu từ broker.
Frontend có phần chuẩn bị luồng WebSocket tiêu thụ dữ liệu realtime.

## Quyết định

- Tiếp tục dùng MQTT subscriber ở backend để lấy dữ liệu đầu vào.
- Giai đoạn hiện tại:
  - Cung cấp API giám sát MQTT (`/api/mqtt/status`, `/api/mqtt/messages`).
  - Cho phép frontend fallback mock data khi chưa có luồng WS backend hoàn chỉnh.
- Giai đoạn tiếp theo:
  - Bổ sung endpoint WS/SSE chính thức từ backend để push dữ liệu realtime cho dashboard.

## Lý do

- Đảm bảo hệ thống có thể vận hành từng bước, không chặn release.
- Tách vấn đề ingest MQTT và phân phối realtime đến frontend thành hai lớp rõ ràng.

## Hệ quả

### Tích cực
- Có khả năng quan sát trạng thái ingest dữ liệu sớm.
- Giảm rủi ro khi triển khai dần các phần realtime.

### Tiêu cực
- Tạm thời có chênh lệch giữa kỳ vọng realtime của frontend và năng lực backend.
- Cần roadmap cụ thể để đóng gap WS/SSE.
