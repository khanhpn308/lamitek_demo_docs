# Changelog (docs)

## 2026-08-02

- **Cải thiện nhận diện thiết bị trên GPS Dashboard**
  - Danh sách bên phải dùng tên thiết bị từ database theo định dạng `Tên(ID)` và
    fallback thành ID khi `devicename` null/rỗng.
  - Marker luôn hiển thị tên phía trên chấm vị trí; nhãn chỉ còn chữ cùng màu với chấm,
    không nền/viền/padding/đổ bóng; tên dài vẫn có ellipsis và title chứa giá trị đầy đủ.
  - Bỏ tọa độ X/Y và timestamp riêng khỏi UI thiết bị nhưng tiếp tục dùng X/Y nội bộ
    để định vị realtime.
  - Mở rộng tìm kiếm theo cả tên và ID; thay badge `Live Tracking` bằng một đồng hồ
    giờ địa phương chung `HH:mm:ss` được cô lập để tránh re-render toàn dashboard.
  - **Verification**:
    - Targeted GPS: `13 passed` trên 4 test files.
    - Toàn bộ frontend: `41 passed` trên 12 test files; Vite production build pass.
    - Chrome smoke test pass ở 1518×737 và 768×800; console và request-failure sạch.

## 2026-07-30

- **Fix màu chữ ô chọn nhóm trong modal “Thêm ảnh bản đồ”**
  - Khắc phục trường hợp ô `Nhóm bản đồ` có nền trắng nhưng chữ cũng màu trắng do
    kế thừa màu chữ từ dark theme.
  - Đặt màu chữ đen và nền trắng rõ ràng cho cả thẻ `select` lẫn từng `option`,
    giúp tên nhóm dễ đọc khi đóng hoặc mở danh sách.
  - Thêm regression test mở modal và xác nhận ô chọn cùng lựa chọn nhóm đều có
    class `text-black`.
  - **Verification**:
    - Frontend `38 passed` trên 12 test files; Vite production build pass.
    - Frontend container được dựng lại và ở trạng thái healthy.
    - Code fix: commit `84e9b67`.

## 2026-07-27

- **Mở rộng định dạng và kích thước ảnh bản đồ**
  - Cho phép upload ảnh tĩnh WebP, PNG và JPG/JPEG; backend đối chiếu đuôi file,
    MIME và định dạng giải mã thực tế trước khi lưu.
  - Bỏ giới hạn width 800 px và height 1–8000 px; mọi kích thước ảnh dương được
    chấp nhận, trong khi Pillow vẫn giữ cơ chế bảo vệ ảnh giải nén bất thường.
  - Đổi giới hạn dung lượng thành **nhỏ hơn 10 MiB**; file đúng 10 MiB trở lên
    bị từ chối ở cả frontend và backend.
  - Nâng giới hạn request Nginx lên 11 MiB để đủ phần multipart overhead nhưng
    backend vẫn là nơi áp dụng biên ảnh nghiêm ngặt dưới 10 MiB.
  - Cập nhật CHECK constraint cho `locations_using` và `locations_deleted`,
    kèm runtime migration cho MySQL volume đã tồn tại và init SQL cho clean install.
  - Giao diện kéo-thả/chọn file, hướng dẫn, preview và cảnh báo đã đồng bộ với
    contract mới.
  - **Verification**:
    - Backend `69 passed`; frontend `37 passed` trên 12 test files; Vite build pass.
    - Backend container healthy; MySQL hiện tại đã nhận đủ CHECK constraint mới.

## 2026-07-24

- **Fix đăng nhập production — `Failed to fetch`**
  - **Nguyên nhân**:
    - Docker frontend dùng `COPY . .`, làm `.env.local` của Vite dev lọt vào
      production build và đóng cứng API/WebSocket thành `localhost:8001`.
    - Trang được phục vụ tại `http://localhost` nên request khác origin bị CSP chặn
      trước khi tới backend.
  - **Thay đổi**:
    - Loại `.env.local` và `.env.*.local` khỏi Docker build context; production trở
      lại dùng proxy Nginx cùng origin `/api` và `/api/ws`.
    - Thêm regression test bảo đảm các Vite local override không bị đóng gói vào
      production image.
  - **Verification**:
    - Frontend `33 passed` trên 12 test files; Docker frontend build thành công và
      container healthy.
    - Kiểm tra tại `http://localhost/login`: request `POST /api/auth/login` đến backend;
      thông tin thử sai trả `401` cùng thông báo nghiệp vụ, không còn `Failed to fetch`.
    - Code fix: commit `5f9cacf`.

## 2026-07-23

- **Map WebP theo nhóm — Phase 5**
  - **Security/WebSocket**:
    - Chuyển JWT frontend khỏi query string sang `Sec-WebSocket-Protocol`
      `iot-jwt`; backend chỉ negotiate tên protocol công khai và từ chối query credential.
    - Thiết bị xác thực bằng protocol `iot-device` hoặc header
      `x-device-password`; mật khẩu không còn nằm trong URL/access log.
    - Bổ sung security header cho document/static asset ở cả Nginx HTTP và HTTPS config.
  - **Regression/fix**:
    - Bổ sung integration test cho file upload giả định dạng, WebP động và lỗi duplicate
      trả từ backend.
    - Sửa GPS user thường tải `/api/devices/my`; admin tiếp tục tải `/api/devices`,
      loại request `403` thừa phát hiện khi kiểm thử trình duyệt.
    - Sửa Compose frontend healthcheck dùng `127.0.0.1`.
  - **Dependency**:
    - Loại dependency `webflow-api` không được sử dụng và toàn bộ legacy subtree;
      `npm audit --omit=dev --audit-level=low` còn **0 vulnerabilities**.
  - **Verification**:
    - Backend `61 passed`; frontend `32 passed` trên 11 test files; Vite build pass.
    - Docker backend/frontend/Mosquitto healthy; frontend/backend HTTP `200`.
    - Kiểm thử owner/member/admin, nhiều group và payload `location` thực tế pass;
      marker xuất hiện đúng map, console sạch, access log không chứa query token.
  - **Tài liệu**:
    - Thêm `ADR-0005` và `architecture/map-webp-phase5.md`; cập nhật API, SRS,
      kiến trúc, changelog và vòng đời Task 10.

- **Map WebP theo nhóm — Phase 4**
  - **Backend/lifecycle**:
    - Xóa group archive toàn bộ map active với lý do `group_deleted`, dọn membership/invitation
      và hard-delete group trong cùng transaction; lỗi giữa chừng trả `409` và rollback.
    - Xóa tài khoản owner dùng cùng lifecycle với lý do `owner_deleted`; membership trong group
      của owner khác chỉ bị dọn, không xóa nhầm group/map.
    - Giữ policy owner inactive/hết hạn: non-admin tạm không thấy group/map, admin vẫn quản trị;
      active lại tự khôi phục khả năng xem.
  - **Seed/Docker**:
    - Seed idempotent `Floor_1`…`Floor_4` vào `System Debug Maps` của `AD00000` qua cùng validator WebP.
    - Kiểm tra location ở cả active/archive, không phân biệt hoa/thường; map đã archive không xuất hiện lại.
    - Đóng gói floorplan seed read-only trong backend image; compatibility endpoint tiếp tục đọc MySQL.
  - **Frontend**:
    - Cập nhật xác nhận xóa group để nêu rõ mọi map đang dùng sẽ được chuyển vào lịch sử đã xóa.
  - **Quality**:
    - Backend `57 passed`; frontend `27 passed`; Vite production build pass.
    - Docker/MySQL existing deployment seed đủ bốn map width 800 px và giữ nguyên ID sau restart.
  - **Tài liệu**:
    - Thêm `architecture/map-webp-phase4.md`; cập nhật API, SRS, kiến trúc và vòng đời task.

- **Map WebP theo nhóm — Phase 3**
  - **Backend/API**:
    - Thêm upload multipart WebP với validation nội dung thật, cảnh báo tiếng Việt, chống duplicate/race và rate limit 30 lần/giờ/user.
    - Thêm API metadata, phân phối ảnh theo quyền với `nosniff` + `private, no-store`, archive map và lịch sử admin có phân trang.
    - Chuyển compatibility API `/locations` và `/floorplans/{location}.webp` sang đọc MySQL.
  - **Frontend**:
    - Thêm nút **Thêm bản đồ**, modal kéo-thả/chọn file, preview, chọn group và nhập location gateway.
    - GPS dùng dropdown `Nhóm bản đồ → Khu vực (Map)` và chỉ tải BLOB đang chọn.
    - Thêm xóa map cho owner/admin và tab **Lịch sử map đã xóa** cho admin.
    - Đồng bộ ngay dropdown GPS sau khi tạo/xóa group hoặc accept lời mời, không cần reload.
  - **Quality/security**:
    - Backend `48 passed`; frontend `26 passed`; Vite production build pass.
    - Docker build và browser smoke test end-to-end pass; console không có warning/error.
    - BLOB không có trong API danh sách/lịch sử; object URL được thu hồi khi đổi map/đóng preview.
  - **Tài liệu**:
    - Thêm `architecture/map-webp-phase3.md`; cập nhật API, SRS, kiến trúc và Task 5–7.

- **Map WebP theo nhóm — Phase 2**
  - **Backend/API**:
    - Thêm Group CRUD với quyền admin/owner, tên trim và unique không phân biệt hoa/thường theo owner.
    - Thêm invitation/membership: mời username chính xác, accept/reject, hủy pending, gỡ member và re-invite sau reject.
    - Chặn self-invite, duplicate, inactive account, owner inactive khi accept và IDOR; gửi lời mời giới hạn 100 lần/giờ theo JWT user.
    - Tại thời điểm Phase 2, group có map active chưa được xóa; giới hạn này đã được thay thế ở Phase 4 bằng cascade archive atomically.
  - **Frontend**:
    - Thêm nút và dialog **Quản lý nhóm** trên GPS Dashboard.
    - Hỗ trợ tạo/list/rename/delete group, admin tạo hộ owner, quản lý thành viên và xử lý lời mời.
    - Thêm loading, retry lỗi, trạng thái chỉ xem và toolbar responsive.
  - **Quality/security**:
    - Chuẩn hóa frontend test về Vitest; `14 passed`, Vite production build pass.
    - Backend `41 passed`; Docker smoke test create/list/rename pass và console trình duyệt sạch.
    - Loại toàn bộ advisory critical/high/moderate bằng bản vá dependency tương thích;
      5 low trong chuỗi legacy `webflow-api` tại thời điểm này đã được xử lý ở Phase 5
      bằng cách loại dependency không sử dụng.
  - **Tài liệu**:
    - Thêm `architecture/map-webp-phase2.md`.
    - Cập nhật hợp đồng API, SRS và trạng thái Task 3–4 trong `alltasks.md`.

## 2026-07-22

- **Map WebP theo nhóm — Phase 1**
  - **Database/ORM**:
    - Thêm `map_group`, `map_group_membership`, `locations_using` và `locations_deleted`.
    - Map active dùng `MEDIUMBLOB`, unique location toàn hệ thống, CHECK kích thước/dung lượng và FK `RESTRICT` để bắt buộc archive trước khi xóa group/owner.
    - Archive giữ nguyên BLOB/metadata cùng snapshot group, owner, người upload, người xóa và lý do; không có foreign key để không mất lịch sử.
    - Hỗ trợ cả clean install qua `database_service/sql/schema.sql` và existing volume qua `Base.metadata.create_all()`.
  - **Backend foundation**:
    - Thêm policy dùng chung cho admin, owner và accepted member; user thường không thấy nhóm khi bản thân hoặc owner inactive/hết hạn.
    - Thêm validator giải mã nội dung WebP thật bằng Pillow: file tĩnh, tối đa 5 MiB, width 800 px, height 1–8000 px và SHA-256 tin cậy.
    - Thêm archive helper dùng row lock, copy-before-delete và để caller quản lý commit/rollback.
    - Thêm `Pillow`, `python-multipart` và dependency test backend.
  - **Verification**:
    - `26 passed` cho toàn bộ backend tests.
    - Clean bootstrap MySQL 8.4, existing-volume startup, schema/index/FK/CHECK và backend import đều pass.
  - **Tài liệu**:
    - Thêm tài liệu kiến trúc `architecture/map-webp-phase1.md`.
    - Thêm `ADR-0004` cho quyết định lưu BLOB và tách bảng active/archive.
  - **Chưa bao gồm**:
    - Chưa thêm REST API, giao diện upload/quản lý nhóm hoặc chuyển GPS sang đọc map từ MySQL; các phần này thuộc Phase 2–4.

## 2026-05-17

- **GPS Dashboard & InfluxDB Realtime Integration**
  - **Backend**:
    - Triển khai endpoint `GET /api/locations`: Tự động quét thư mục `assets/floorplans/` và trả về danh sách bản đồ SVG khả dụng.
    - Cấu hình `locations_routes.py` và đăng ký vào `api_router`.
  - **Frontend**:
    - Xây dựng hệ thống GPS Tracking hoàn chỉnh:
      - `MapViewer.jsx`: Xử lý hiển thị bản đồ SVG và ánh xạ tọa độ `x`, `y` động theo tỷ lệ phần trăm (0-100%).
      - `GPSDashboard.jsx`: Giao diện điều khiển với dropdown chọn khu vực (location) tự động đồng bộ từ API, thanh tìm kiếm thiết bị và danh sách thiết bị realtime.
      - `GPSPage.jsx`: Trang tích hợp dữ liệu thực tế, thực hiện gộp dữ liệu từ MySQL (danh sách thiết bị) và InfluxDB (tọa độ mới nhất qua `/api/mqtt/history`).
    - Cơ chế cập nhật: Thiết lập polling 15 giây để đồng bộ vị trí thiết bị liên tục từ InfluxDB.
    - UI/UX: Bổ sung nút Hamburger Menu tại `Layout.jsx` hỗ trợ chuyển đổi nhanh giữa các Dashboard (Telemetry vs GPS).
  - **Fixes**: Khắc phục triệt để lỗi encoding ký tự lạ và lỗi render màn hình trắng (White Screen) do truy cập thuộc tính undefined.

- **Frontend - Thiết kế & Triển khai Dashboard Thiết bị**
  - Thiết kế JSON Schema chuẩn cho endpoint REST `/api/devices` và kênh WebSocket `/ws/devices/{device_id}`, thống nhất kiểu dữ liệu `device_id` là `integer` xuyên suốt hệ thống.
  - Xây dựng Component Tree Blueprint theo mô hình Smart/Dumb components để tối ưu hiệu năng và khả năng bảo trì.
  - Triển khai mã nguồn thực tế:
    - `src/types/device.ts`: Định nghĩa TypeScript Interfaces nghiêm ngặt cho Device và Authorization.
    - `src/lib/axios.ts`: Cấu hình API client hỗ trợ Bearer Token và tiền tố `/api`.
    - `src/hooks/useDevices.ts`: Custom hook xử lý fetch dữ liệu, trạng thái loading và error handling.
    - `src/components/devices/`: Bộ UI components hoàn chỉnh gồm `DeviceFilters`, `DeviceTable`, `DeviceTableRow`, và `DeviceTableSkeleton` (skeleton loading).
    - `src/pages/Devices.tsx`: Trang danh sách thiết bị tích hợp logic lọc tìm kiếm client-side và điều hướng.
  - Công nghệ sử dụng: React 19 (Functional Components + Hooks), Tailwind CSS, shadcn/ui, Axios, Lucide React.

## 2026-05-07

- **Backend - Refactoring**
  - Cấu trúc WebSocket routes: chuyển 2 endpoint `/ws/global` và `/ws/devices/{device_id}` từ inline `@app.websocket` trong `app/main.py` sang `app/api/websocket_routes.py` để nhất quán với chuẩn architecture (tất cả routes trong `app/api/**_routes.py`).
  - Cập nhật `app/api/websocket_routes.py`:
    - Thêm endpoint `/ws/global` (broadcast realtime data tới tất cả GlobalDashboard clients).
    - Thêm endpoint `/ws/devices/{device_id}` (broadcast realtime data tới dashboard per-device).
    - Cập nhật endpoint `/ws/esp32/{device_id}` (bi-directional device uplink):
      - Hỗ trợ text JSON, binary frames, ping/pong.
      - Thêm extensive docstring giải thích flow, payload schema, error handling.
      - TODO comment: future downlink support qua `hub.send_to_esp32()` từ REST API.
      - TODO comment: thêm device authentication (API key, JWT token, device_secret).
    - Cập nhật docstring module với data flow diagram: MQTT → decoder → InfluxDB/RealtimeHub/TestService.
    - Thêm hàm `_get_realtime_hub()` helper lấy shared RealtimeHub từ app state.
  - Cập nhật `app/main.py`:
    - Xóa 2 endpoint `/ws/global` và `/ws/devices/{device_id}` (chuyển sang websocket_routes.py).
    - Thêm inline comment rõ ràng giải thích WebSocket routes được define trong websocket_routes.py.
  - Cập nhật `docs/api/api-documentation.md`:
    - Thêm section mới "## 4. WebSocket (Realtime Data Streaming)" với chi tiết:
      - WS `/ws/global`: mô tả, cách kết nối, payload example, ghi chú.
      - WS `/ws/devices/{device_id}`: mô tả, khi nào dùng vs `/ws/global`.
      - WS `/ws/esp32/{device_id}`: uplink/downlink mô tả, binary support, Arduino/ESP32 code example.

- **Backend - Test mode (WebSocket + đo độ trễ)**
  - Bổ sung luồng ghi log uplink qua WebSocket để phục vụ đo độ trễ end-to-end (tính `delay_ms` dựa trên `timestamp_ms`/`ts` và `server_receive_ms`).
  - Cập nhật API test:
    - `GET /api/test/config` (admin): xem cấu hình test mode.
    - `PUT /api/test/config` (admin): cập nhật cấu hình, hỗ trợ `protocol: mqtt | websocket` và `device_id`.
    - `GET /api/test/logs` (admin): bổ sung query params `protocol` và `device_id` để auto-filter log theo cấu hình đang test.

- **Backend - WebSocket compatibility & robustness**
  - Mở đồng thời 2 base path cho WebSocket để tương thích nhiều client:
    - Có prefix `/api`: `/api/ws/global`, `/api/ws/devices/{device_id}`, `/api/ws/esp32/{device_id}`
    - Không prefix `/api`: `/ws/global`, `/ws/devices/{device_id}`, `/ws/esp32/{device_id}`
  - Tăng độ bền handler khi client disconnect (handle `websocket.disconnect` frame) và log rõ connect attempt để debug mạng.
  - Hỗ trợ uplink binary protobuf (GPS `coordinates_data`) trên `/ws/devices/{device_id}` và `/ws/esp32/{device_id}`.

- **Nginx - WebSocket proxy hardening (prod)**
  - Cập nhật cấu hình proxy cho `/ws/` và `/api/ws/` để hỗ trợ Upgrade ổn định: `Connection upgrade`, timeout dài, tắt buffering cho WS.

- **Docs - Troubleshooting**
  - Thêm tài liệu xử lý lỗi “kẹt cổng” (localhost OK nhưng IP LAN fail) và các câu lệnh kiểm tra/xoá/tạo `portproxy` + firewall:
    - `docs/troubleshooting/bugs_ket_port_local.md`

## 2026-04-24

- **Docs**
  - Cập nhật `docs/testing/testcases.md`:
    - Mở rộng chi tiết **Test Case 02 (Latency Profiling)** từ đo đơn điểm thành đo theo từng chặng `Node -> Gateway -> Server -> Frontend`.
    - Bổ sung điều kiện tiên quyết (NTP sync, payload timestamp bắt buộc), công thức đo chi tiết, và bộ chỉ số thống kê (`mean`, `median`, `p95`, `p99`, `max`, `std`).
    - Bổ sung quy trình thu mẫu chuẩn (>= 1000 mẫu, bỏ warm-up), đo theo 3 bối cảnh tải (Baseline/Normal/Stress), và tiêu chí chấp nhận cụ thể cho `delay_node_to_server_ms`.
    - Bổ sung yêu cầu đầu ra kiểm thử (artifact CSV/JSON + báo cáo thống kê).
  - Cập nhật `docs/adr/SRS.md`:
    - Làm rõ NFR độ trễ bằng ngưỡng đo định lượng (`mean` và `p95`) và liên kết phương pháp đo với `docs/testing/testcases.md` (Test Case 02).
  - Cập nhật `docs/guidelines/backend-guidelines.md`:
    - Bổ sung guideline kiểm thử backend cho luồng latency profiling: bắt buộc lưu timestamp/delay theo từng chặng và chuẩn hóa artifact phục vụ phân tích hồi quy hiệu năng.

## 2026-04-13

- **Docs**
  - Thêm `docs/overvew.md`: tài liệu tổng quan chuẩn Markdown cho phạm vi `app_service`, gồm:
    - kiến trúc tổng thể FE/BE/MQTT/Influx/Nginx,
    - cây thư mục chính,
    - chức năng từng thư mục,
    - chức năng từng file trọng yếu (root config, backend modules, frontend pages/components/lib, deployment).

## 2026-04-10

- **Backend**
  - Cập nhật `app_service/backend/app/core/payload_decoder.py`:
    - Bổ sung nhánh decode Protobuf cho schema `SimpleSensor` mới của node ESP32:
      - `string device_id = 1`
      - `float temperature = 2`
      - `bool is_active = 3`
      - `uint32 sequence = 4`
      - `uint64 timestamp_ms = 5`
    - Cập nhật thứ tự fallback decode: JSON UTF-8 -> Protobuf `SimpleSensor` -> template binary NanoPB -> `raw_hex` khi không parse được.
    - Bổ sung map trường chuẩn đầu ra gồm `is_active`, `sequence`, `timestamp_ms`.
    - Chuẩn hóa `ts` ưu tiên từ `timestamp_ms` (nếu có) để đồng bộ timeline dữ liệu giữa publisher và backend.
  - Cập nhật `app_service/backend/app/core/payload_decoder.py`:
    - Bổ sung chuẩn hóa timestamp epoch với guard hợp lệ (tránh ghi điểm về năm `1970` khi node gửi uptime),
    - Parse chuỗi ISO không timezone theo chuẩn UTC (không cộng/bù giờ theo timezone máy chủ, ví dụ `+7`),
    - Chuẩn hóa alias payload cho template mới (`device_id`/`deviceId`/`id`, `sensor_type`/`sensorType`/`type`, `temperature`/`temp`/`temp_c`, ...),
    - Hỗ trợ map payload dạng generic `value`/`reading`/`measurement` theo `sensor_type`.
  - Cập nhật `app_service/backend/app/core/influx_service.py`:
    - Chuẩn hóa ghi field metric theo alias (`temperature`, `vibration`, `voltage`, `current`),
    - Bổ sung guard timestamp khi ghi point vào InfluxDB để fallback về thời gian hiện tại nếu timestamp không hợp lệ.
  - Cập nhật `app_service/backend/app/api/mqtt_routes.py`:
    - Thêm endpoint debug `GET /api/mqtt/influx/status` để kiểm tra trạng thái kết nối Influx (`enabled`, `started`, `bucket`, `measurement`, `last_error`).
- **Frontend**
  - Cập nhật `app_service/src/pages/GlobalDashboard.jsx`:
    - Chuẩn hóa `device_type` có dấu tiếng Việt bằng hàm normalize bỏ dấu,
    - Cho phép suy luận type từ payload realtime (`sensor_type`) và từ history,
    - Bổ sung hiển thị thiết bị phát hiện từ dữ liệu history (trường hợp `device_id` là chuỗi từ node ESP32).
  - Cập nhật `app_service/src/pages/DeviceDetail.jsx`:
    - Fallback truy vấn history theo `topic` khi `device_id` DB không khớp `device_id` payload,
    - Fallback realtime qua `ws/global` + filter theo `topic`/`device_id` để không mất dữ liệu chart khi định danh thiết bị lệch giữa DB và node,
    - Chuẩn hóa timestamp event realtime về milliseconds trước khi render biểu đồ.
- **Docs**
  - Thêm `.gitattributes` trong repo `docs` để chuẩn hóa line ending (DLE = Docs Line Endings), tránh cảnh báo `LF will be replaced by CRLF` khi `git add` trên Windows.
  - Cập nhật `docs/changelogs.md` để phản ánh luồng fix Influx + dashboard mapping cho payload template mới từ ESP32.
  - Thêm `.gitattributes` tại root repo để chuẩn hóa line ending (GLE = Git Line Endings), giảm cảnh báo `LF will be replaced by CRLF` và tránh diff không cần thiết giữa Windows/Linux.
  - Cập nhật `docs/deployment/docker-linux-deployment.md`: bổ sung mục `9.8 Restart container khi có cập nhật` (RCU), kèm quy trình restart/recreate cho `app_service`, `database_service`, `influxdb_service`.
  - Cập nhật `deloy.md`: bổ sung mục `7) Restart container khi có cập nhật` (RCU) cho cả ba stack dịch vụ.
  - Thêm `docs/bugs/2026-04-10-mosquitto-esp32-intermittent-disconnect-clientid.md`: ghi nhận sự cố MQTT kết nối dao động do trùng Client ID ESP32 và quy trình xử lý từng bước.
  - Thêm `docs/bugs/2026-04-09-mysql-auth-docker-env-override.md`: ghi nhận sự cố xác thực MySQL trong Docker do biến môi trường CMD ghi đè `.env`, lỗi `1045` cho `root` và `iot_user`, cùng quy trình xử lý step by step (chuẩn hóa `--env-file`, đồng bộ password/quyền user ứng dụng, và kiểm tra lệch schema `iot`/`demo_iot`).
  - Cập nhật nội dung `docs/bugs/2026-04-10-mosquitto-esp32-intermittent-disconnect-clientid.md`: làm rõ triệu chứng ESP32 vẫn publish trong lúc broker đóng phiên cũ và reconnect.

## 2026-04-08

- **Docker**
  - Tách InfluxDB thành stack riêng `influxdb_service` với `docker-compose.yml` độc lập, volume `influxdb-data`, healthcheck và kết nối external network `iot-net`.
  - Cập nhật `app_service/docker-compose.yml`: gỡ service `influxdb` nội bộ khỏi app stack để `backend` kết nối sang InfluxDB qua tên dịch vụ `influxdb` trên mạng chung `iot-net`.
  - Cập nhật environment cho `backend` trong compose để truyền đầy đủ cấu hình MQTT + Influx (`MQTT_*`, `INFLUX_*`).
- **Backend**
  - Thêm `app_service/backend/app/core/influx_service.py`: service ghi dữ liệu cảm biến vào InfluxDB và truy vấn lịch sử theo cửa sổ thời gian (mặc định 30 phút).
  - Thêm `app_service/backend/app/core/payload_decoder.py`: khung mẫu giải mã payload nhị phân NanoPB (template) + chuẩn hóa dữ liệu cho các loại cảm biến `temperature`, `vibration`, `power`.
  - Thêm `app_service/backend/app/core/realtime_hub.py`: broadcast realtime qua WebSocket cho global dashboard và dashboard theo từng thiết bị.
  - Cập nhật `app_service/backend/app/core/mqtt_subscriber.py`:
    - decode payload nhận từ MQTT bằng decoder mới,
    - callback để ghi Influx + phát realtime,
    - hỗ trợ subscribe/unsubscribe topic động khi runtime (`list_topics`, `subscribe_topic`, `unsubscribe_topic`).
  - Cập nhật `app_service/backend/app/main.py`: khởi tạo `InfluxService`, `RealtimeHub`, wiring callback MQTT, và mở WebSocket endpoint `/ws/global`, `/ws/devices/{device_id}`.
  - Cập nhật `app_service/backend/app/api/mqtt_routes.py`: thêm API quản lý topic động cho admin (`/mqtt/topics`, `/mqtt/topics/subscribe`, `/mqtt/topics/unsubscribe`) và API lịch sử InfluxDB (`/mqtt/history?minutes=30&device_id=...`).
  - Cập nhật mô hình thiết bị: thêm trường `topic` trên bảng `device` (ORM + schema + API), hỗ trợ lưu topic theo từng thiết bị và tự đồng bộ subscribe/unsubscribe runtime khi admin cập nhật topic.
  - Bổ sung API quản lý topic theo thiết bị:
    - `GET /api/devices/topics` (admin): danh sách topic đã lưu theo thiết bị,
    - `PUT /api/devices/{device_id}/topic` (admin): cập nhật riêng topic và đồng bộ runtime.
  - Cập nhật startup `main.lifespan`: khôi phục danh sách subscribe MQTT từ dữ liệu `device.topic` đã lưu trong DB.
  - Cập nhật `app_service/backend/requirements.txt`: thêm dependency `influxdb-client`.
  - Cập nhật file môi trường `app_service/backend/.env.example` và `app_service/.env.example` để bổ sung cấu hình MQTT/Influx/WebSocket.
- **Frontend**
  - Cập nhật `app_service/src/pages/DeviceDetail.jsx`:
    - lấy dữ liệu lịch sử 30 phút từ API `/api/mqtt/history` cho bảng History và chart Dashboard,
    - giữ luồng realtime qua WebSocket nối thêm vào chart,
    - thêm UI cho admin subscribe/unsubscribe topic trực tiếp từ web.
  - Cập nhật `app_service/src/pages/GlobalDashboard.jsx`: preload số liệu gần nhất từ lịch sử 30 phút trước khi nhận stream realtime WebSocket.
  - Thêm trang admin `app_service/src/pages/TopicManagement.jsx` và route `/topic-management`: giao diện quản lý topic MQTT theo từng thiết bị (lưu vào DB + đồng bộ runtime).
- **Docs**
  - Cập nhật `docs/deployment/docker-linux-deployment.md`: bổ sung luồng triển khai `influxdb_service` riêng và thứ tự vận hành nhiều stack.
  - Cập nhật `docs/api/api-documentation.md`, `docs/api/openapi-like.yaml`: bổ sung trường `topic` trong hợp đồng thiết bị và mô tả UI admin quản lý topic.
  - Cập nhật `app_service/backend/README.md`: bổ sung hướng dẫn InfluxDB, payload decode template NanoPB, endpoint topic động, endpoint history và websocket realtime.

## 2026-04-06

- **Docs**
  - Thêm `docs/architecture/codebase-walkthrough.md`: thuật ngữ/viết tắt (JWT, RBAC, MQTT, CCCD, …), bản đồ thư mục `app_service/`, thứ tự đọc code, bảng biến môi trường, file “mỏ neo” khi debug.
  - Cập nhật `docs/architecture/system-architecture.md` (mục 7): liên kết walkthrough và mô tả docstring/JSDoc trong repo.
  - Thêm `docs/app_service-functions.md`: liệt kê toàn bộ function trong phạm vi source `app_service/backend/app` và `app_service/src` (kèm line và loại function).
- **Docker**
  - Chuẩn hóa kết nối `app_service` ↔ `database_service` khi chạy bằng 2 compose riêng: dùng external network chung `iot-net` và `DB_HOST=db` (service name) thay vì `127.0.0.1`.
- **Mã nguồn (comment / docstring)**
  - Backend (`app_service/backend/app/`): module docstring + docstring hàm/lớp cho `main`, `core` (config, db, deps, security, db*wait, mqtt_subscriber, user_expiry), `api/`*, `models/_`, `schemas/\*` (bổ sung đầu file / class nơi cần).
  - Frontend: JSDoc/ghi chú đầu file cho `main.jsx`, `App.jsx`, `IoTApp.jsx`, `AuthContext.jsx`, `lib/api.js`, `lib/base-url.ts`, `ProtectedRoute.jsx`, `AdminRoute.jsx`, `Layout.jsx`.
  - Thêm `app_service/src/components/ui/README.md` (giải thích thư mục shadcn/ui, không doc từng file primitive).
  - Ghi chú đầu file `app_service/vite.config.js` (proxy API khi dev).
- **Frontend (Device types & dashboard)**
  - Cập nhật `AddDeviceModal`: `deviceTypes` còn 3 loại chuẩn: `Nhiệt độ (Temperature)`, `Công suất (Power)`, `Độ rung (Vibration)`.
  - Refactor `DeviceDetail` tab `Dashboard` thành biểu đồ theo loại cảm biến:
    - `Temperature`: chỉ biểu đồ nhiệt độ `°C` theo thời gian.
    - `Power`: biểu đồ `Voltage (V)` và `Current (A)` theo thời gian.
    - `Vibration`: biểu đồ `mm/s` theo thời gian (line chart).
  - Chuẩn hóa hiển thị `device_type` trên trang `Devices` theo 3 loại trên.
  - Cập nhật `GlobalDashboard`:
    - Hiển thị đủ 4 biểu đồ tổng quan `Current`, `Voltage`, `Temperature`, `Vibration` (trục X theo thiết bị, trục Y theo giá trị).
    - Thiết bị chỉ đi vào đúng biểu đồ theo `device_type`: `Temperature` -> biểu đồ nhiệt độ; `Power` -> biểu đồ voltage/current; `Vibration` -> biểu đồ rung.
    - Phạm vi dữ liệu theo quyền: admin xem toàn bộ, user xem thiết bị được phân quyền.
    - Thêm cơ chế auto scale trục Y và tự giãn cột/nhãn theo số lượng thiết bị; thiết bị mới tự xuất hiện khi có dữ liệu realtime.
    - Tối giản giao diện chart: ẩn nhãn thiết bị trên trục X, chỉ hiển thị tên thiết bị khi hover cột (tooltip).
    - Bổ sung nút phóng to/thu nhỏ biểu đồ cho cả `GlobalDashboard` và tab `Dashboard` trong `DeviceDetail` (hỗ trợ thoát bằng `Esc` hoặc click nền).

## 2026-04-05

- **Backend**
  - `DELETE /api/devices/{device_id}` (admin): xóa thiết bị và bản ghi `device_authorization` liên quan.
  - `GET /api/devices/{device_id}`: thêm `authorized_users` (RBAC); `user_device_asignment_id` chỉ trả cho admin.
  - `PATCH /api/devices/{device_id}`: hỗ trợ cập nhật `user_device_asignment_id`.
  - `GET /api/users`: mỗi user có `authorized_devices` (thiết bị đã phân quyền).
  - `GET /api/authorizations`: tham số `user_id` hoặc `device_id` (đúng một trong hai).
- **Frontend**
  - Trang Devices: admin có nút xóa thiết bị (xác nhận bằng `OK`).
  - Quản lý người dùng: mỗi thẻ user hiển thị danh sách thiết bị được phân quyền.
  - Chi tiết thiết bị: danh sách user được phân quyền; khối chỉnh sửa `user_device_asignment_id` chỉ admin.
  - Hotfix tương thích backend cũ: nếu `GET /users` chưa có `authorized_devices`, frontend tự backfill từ `GET /authorizations?user_id=...`.
  - Hiển thị lỗi rõ ràng khi backend chưa deploy endpoint `DELETE /api/devices/{id}` (405 Method Not Allowed).
- **Docs**
  - Thêm `docs/guidelines/git-github-teamwork.md`: hướng dẫn branch, fetch/pull, làm việc nhóm trên GitHub, đồng bộ với remote không mất code local, conflict, rebase/merge, stash, force-with-lease.
  - Bổ sung checklist deploy cực nhanh EC2 (pull, build, restart backend, verify endpoint) tại `docs/deployment/docker-linux-deployment.md` mục `8.7`.
  - Hotfix runtime backend sau deploy: thêm import `date` trong `app_service/backend/app/schemas/devices.py` để tránh lỗi `NameError: name 'date' is not defined` khi khởi động.
- **Frontend hotfix**
  - Trang `Devices`: mỗi thẻ thiết bị hiển thị user đang được phân quyền quản lý; ưu tiên dữ liệu từ `GET /users` và có fallback sang `GET /authorizations?device_id=...` cho backend cũ.

Tài liệu API: `docs/api/openapi-like.yaml`, `docs/api/api-documentation.md`.
