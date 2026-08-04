# Nhóm bản đồ và lời mời — Phase 2

- **Trạng thái**: Hoàn thành
- **Ngày hoàn thành**: 2026-07-23
- **Phạm vi**: Group CRUD, invitation/membership API, dialog quản lý nhóm tại GPS Dashboard
- **Ngoài phạm vi**: Upload WebP, chọn map theo nhóm, archive map và cascade khi xóa group/owner

## 1. Kết quả

Phase 2 cung cấp lớp quản trị nhóm dùng chung cho các map sẽ được upload ở Phase 3:

- User active tạo và quản lý nhiều nhóm của chính mình.
- Admin xem mọi nhóm và có thể tạo nhóm cho owner khác bằng username khớp chính xác.
- Owner/admin đổi tên, xóa nhóm chưa có map active, mời người dùng, hủy lời mời và gỡ thành viên.
- Người được mời phải accept trước khi nhóm xuất hiện trong danh sách có thể truy cập.
- Member chỉ có quyền xem; không có endpoint để upload, xóa, quản lý hoặc tự rời nhóm.

## 2. REST API

Các endpoint đều yêu cầu JWT Bearer:

| Method | Endpoint | Quyền và mục đích |
|---|---|---|
| `GET` | `/api/map-groups` | Admin xem tất cả; user xem nhóm sở hữu hoặc đã accept |
| `POST` | `/api/map-groups` | Tạo nhóm; admin có thể truyền `owner_username` |
| `PATCH` | `/api/map-groups/{group_id}` | Owner/admin đổi tên |
| `DELETE` | `/api/map-groups/{group_id}` | Owner/admin xóa nhóm chưa có map active |
| `GET` | `/api/map-groups/{group_id}/members` | Owner/admin xem pending/accepted/rejected |
| `POST` | `/api/map-groups/{group_id}/invitations` | Owner/admin mời username chính xác |
| `DELETE` | `/api/map-groups/{group_id}/members/{user_id}` | Hủy pending hoặc gỡ membership |
| `GET` | `/api/map-group-invitations` | User xem lời mời pending của chính mình |
| `PATCH` | `/api/map-group-invitations/{group_id}` | User accept hoặc reject lời mời của chính mình |

Các lookup username kiểm tra lại bằng phép so sánh Python để giữ semantics phân biệt
hoa/thường ngay cả khi MySQL dùng collation không phân biệt hoa/thường.

## 3. Quy tắc nghiệp vụ và bảo mật

- Tên group được trim, dài 1–100 ký tự và duy nhất không phân biệt hoa/thường theo từng owner.
- Username được trim, dài tối đa 45 ký tự và phải khớp chính xác.
- Không cho self-invite, duplicate pending/accepted hoặc mời tài khoản inactive/hết hạn.
- Invitation bị reject có thể chuyển lại `pending` khi owner/admin mời lại.
- Chỉ chính user được mời mới được accept/reject; phản hồi lần hai trả `409`.
- Accept bị chặn nếu owner không còn active.
- Endpoint quản trị trả `404` cho group không thuộc quyền để hạn chế IDOR enumeration.
- Gửi lời mời giới hạn `100/hour` theo danh tính JWT; IP chỉ là fallback cho request không giải mã được token.
- Race condition tạo membership trùng được ánh xạ thành `409`, không làm lộ lỗi database.
- Phase 4 đã thay thế giới hạn này: xóa group sẽ archive map active và dọn membership
  trong cùng transaction; lỗi giữa chừng trả `409` và rollback.

## 4. Frontend

`GPSDashboard` có nút **Quản lý nhóm** mở dialog gồm:

- Tab **Nhóm của tôi**: danh sách group, badge `Chỉ xem` cho member, form tạo group.
- Admin có thêm ô `Username owner`.
- Màn quản trị owner/admin: đổi tên, mời username, xem trạng thái membership, hủy/gỡ và xóa group.
- Tab **Lời mời**: accept/reject invitation pending.
- Trạng thái loading, lỗi có thể retry và các nút disabled trong lúc mutation.
- Toolbar GPS cho phép wrap để nút mới không làm vỡ layout hẹp.

Client API nằm tại `app_service/src/lib/mapGroupsApi.js`; dialog được tách thành
orchestrator và ba presentational component để kiểm thử hành vi độc lập.

## 5. Kiểm thử và xác minh

- Backend: `41 passed`, gồm authorization matrix, IDOR, duplicate/self/inactive,
  re-invite, pending → accepted/rejected, owner inactive và rate limit theo JWT.
- Frontend: `14 passed` trên Vitest, gồm API client, dialog, admin create,
  invitation, lỗi có thể retry và regression GPS hiện có.
- Production build: Vite `7.3.6` build thành công.
- Docker: backend healthy; frontend image build và khởi động thành công.
- Browser smoke test: admin tạo, list và rename group trên GPS Dashboard thành công;
  console không có warning/error; group tạm đã được xóa sau kiểm tra.
- Tại thời điểm Phase 2, `npm audit fix` đã loại toàn bộ advisory
  critical/high/moderate; còn 5 low trong dependency tree legacy của `webflow-api`.
  Phase 5 xác nhận package này không được sử dụng, loại dependency và đưa audit về
  0 advisory mà không cần `--force`.

## 6. Công việc tiếp theo

- Phase 3 đã hoàn thành: upload/validate WebP, dropdown `Nhóm → Location`, endpoint ảnh và archive map.
- Phase 4 đã hoàn thành: archive toàn bộ map trước khi xóa group/owner và seed map hệ thống.

Xem thêm [Phase 1](map-webp-phase1.md), [Phase 4](map-webp-phase4.md),
[ADR-0004](../adr/ADR-0004-luu-tru-map-webp-va-archive.md)
và [task đã hoàn thành](../../alltasks-done.md).
