# Quy Định Cấu Trúc Mã Nguồn Frontend

- **Mã tài liệu**: FE-GUIDE-001
- **Phiên bản**: 1.1.0
- **Ngày cập nhật**: 2026-06-25

## 1. Kiến trúc thư mục chuẩn

```text
app_service/
  src/components/          # reusable components và layout
  src/components/ui/       # primitive UI components (shadcn/radix)
  src/components/common/   # component dùng chung theo token (PageHeader, Panel, StatCard, StatusBadge)
  src/components/Dashboard/GPS/  # GPS tracking (MapViewer, GPSDashboard)
  src/pages/               # page-level components
  src/contexts/            # app-wide state (AuthContext)
  src/lib/                 # api client, utils, helper (deviceStatus, wsUrl)
  src/hooks/               # custom hooks
  src/data/                # mock data và helper demo
  src/styles/              # css global + semantic tokens mapping
  generated/               # webflow generated design tokens
```

### Nguyên tắc
- Page chỉ orchestration, không chứa quá nhiều business logic.
- Component domain không gọi API trực tiếp nếu có thể chuyển lên page/hook.
- Primitive UI trong `components/ui` không chứa logic nghiệp vụ.

## 2. Luồng quản lý trạng thái

### 2.1 Phân loại state
- **Global auth state**: quản lý qua `AuthContext`.
- **Page state**: quản lý bằng `useState`/`useMemo` trong trang tương ứng.
- **Form state**: trạng thái cục bộ trong form/modal.
- **Server state**: lấy qua `apiFetch`, cập nhật theo vòng đời trang.

### 2.2 Quy tắc bắt buộc
- Mọi gọi API phải có trạng thái:
  - loading
  - error
  - success/empty
- Không mutate object/array state trực tiếp.
- Không để token nằm rải rác; dùng thống nhất key `iot_token`.

## 3. Tiêu chuẩn tái sử dụng component

### 3.1 Danh sách component domain và props

| Component | Props chính | Vai trò |
|---|---|---|
| `IoTApp` | Không | Root router toàn app |
| `Layout` | Không | Khung điều hướng chính |
| `ProtectedRoute` | Không | Chặn route chưa đăng nhập |
| `AdminRoute` | Không | Chặn route không phải admin |
| `AddDeviceModal` | `onClose`, `onAdd` | Tạo thiết bị mới |
| `AssignDeviceModal` | `user`, `currentAdmin`, `onClose`, `onSuccess` | Cấp quyền thiết bị cho user |
| `ChangePasswordModal` | `deviceId`, `onClose` | Đổi mật khẩu thiết bị — dựng trên `<Dialog>` shadcn (focus-trap, ESC) |

### 3.1.1 Component dùng chung (`src/components/common/`)

Trích các pattern lặp lại, dùng design token thay vì hard-code màu. Ưu tiên dùng khi dựng trang mới.

| Component | Props chính | Vai trò |
|---|---|---|
| `PageHeader` | `title`, `description`, `actions` | Tiêu đề trang `<h1>` + mô tả + slot action bên phải |
| `Panel` | `className`, `children` | Khung card/section chuẩn (`bg-card border-border rounded-xl shadow-lg`) |
| `StatCard` | `title`, `value`, `icon`, `color`, `subtitle` | Thẻ KPI (nền token; `color` là màu semantic do caller truyền) |
| `StatusBadge` | `status` | Badge ONLINE/OFFLINE, tự chuẩn hoá status qua `isOnline()` |

### 3.2 Quy ước props
- Callback dùng tiền tố `on*`.
- Prop boolean dùng tiền tố `is/has/show`.
- Tránh truyền props dư thừa; ưu tiên object nhỏ và rõ nghĩa.

## 4. Quy tắc định tuyến (routing)

- Route public: `/login`, `/forgot-password`.
- Route private: các route nghiệp vụ phải bọc `ProtectedRoute`.
- Route admin: phải bọc thêm `AdminRoute`.
- Không chỉ ẩn menu; backend phải kiểm tra quyền tương ứng.

## 5. Chuẩn gọi API

- Dùng duy nhất `apiFetch(path, options)`.
- Mọi request private phải đính kèm bearer token.
- Thông điệp lỗi UI lấy ưu tiên từ `detail` backend.

## 6. Quy tắc UI nhất quán

- Action chính: dùng token `primary` (`bg-primary text-primary-foreground`).
- Action phá hủy dữ liệu: dùng `destructive` + xác nhận qua `<AlertDialog>` (vd modal xoá thiết bị).
- Trạng thái thiết bị: chuẩn hoá qua `src/lib/deviceStatus.js` (`toUiStatus`/`isOnline`) —
  backend trả `active`/`deactive`, mock trả `online`/`offline`; UI luôn quy về `online`/`offline`.
  Hiển thị qua `<StatusBadge>` (online=xanh, offline=đỏ).
- Trạng thái dữ liệu: khi rỗng phải có **empty state** rõ ràng (vd 4 biểu đồ telemetry hiện
  "Chưa có dữ liệu" thay vì trục trống).
- Màu nền/chữ dùng token (`bg-card`, `text-foreground`, `text-muted-foreground`...) — KHÔNG
  hard-code `slate-*`. Ngoại lệ: màu semantic trạng thái, màu chart, gradient nền Login.
- Không dùng text mơ hồ; thông báo lỗi phải cụ thể. UI dùng tiếng Việt CÓ DẤU đầy đủ.

## 7. Tiêu chuẩn typography và color token (enterprise)

### 7.1 Nguồn chuẩn (source of truth)

- `app_service/generated/webflow.css`: token gốc từ design system.
- `app_service/generated/fonts.css`: danh sách font import do Webflow sinh.
- `app_service/src/styles/global.css`: ánh xạ token sang theme runtime.

### 7.2 Font policy (chuẩn dùng trong ứng dụng)

- Font chuẩn áp dụng UI: `Roboto, sans-serif`.
- Khai báo thống nhất:
  - `--body-font`
  - `--heading-font`
  - `--button-font`
- Không hard-code `font-family` tại component nếu không có phê duyệt từ Design.
- Không chỉnh sửa trực tiếp file generated; override thông qua `global.css`.

### 7.3 Color policy (semantic token)

- Chỉ dùng semantic tokens (`--primary`, `--background`, `--destructive`, ...).
- Không dùng mã HEX trực tiếp trong component trừ khi có design exception.
- Light/Dark phải có cặp token tương ứng.
- Mọi thay đổi token phải cập nhật tài liệu `docs/design/frontend-design-tokens.md`.

## 8. Định hướng cải tiến

- Tách custom hooks cho các luồng lớn:
  - `useUsersManagement`
  - `useDevices`
  - `useDeviceDetail`
- Chuẩn hóa dần component sang TypeScript để tăng type safety.
