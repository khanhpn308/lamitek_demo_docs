# Changelog (docs)

## 2026-08-27

- **JSON Ping — PING-17/P6 final verification complete**
  - Xác minh migration 015 trên cả MySQL hiện có và database tạm sạch; schema, constraint và index
    khớp contract, database thử nghiệm được drop sau kiểm tra.
  - Full-stack Docker/MySQL smoke pass exact text/binary echo, forward gap, late recovery, live admin
    UI và delete/reset; dữ liệu Gateway/Node tạm được dọn sạch và browser console có 0 issue.
  - Smoke phát hiện Google Fonts bị production CSP chặn; đã bỏ external font links, chuyển sang system
    font stack và thêm regression test cho Docker build context.
  - **Verification**: backend `200 passed, 1 skipped`; frontend `92 passed` trên 25 files; Vite và
    Docker production build pass; chi tiết tại `docs/testing/ping-feature-test-report.md`.

## 2026-08-26

- **JSON Ping — PING-16 contract/deployment documentation**
  - Đồng bộ OpenAPI-like với strict text/binary Ping, redacted error, admin summary/delete và
    `ping_stats_updated` WebSocket event.
  - Bổ sung schema/index migration 015, quy trình backup/apply/verify và cảnh báo destructive rollback.
  - Làm rõ URL ID xác thực Gateway khác Node ID trong payload; JSON ping bypass telemetry, Influx,
    presence/`last_seen_at`. MQTT không đổi; WebSocket legacy `PING|` raw echo đã bị bỏ.

- **JSON Ping — Phase P5 frontend admin complete**
  - Thêm admin route/navigation `/ping`, catalog filter và đúng ba summary cards; current payload chỉ
    hiển thị order cùng Node uptime milliseconds, không render raw payload.
  - Admin WebSocket `/ws/pings` refresh đúng selected device, coalesce event burst, chặn stale response
    và reconnect sau 1200 ms; destructive dialog hỗ trợ atomic clear và zero-state refetch.
  - Mobile nav dùng horizontal scrolling có kiểm soát; non-admin bị chặn bởi `AdminRoute`.
  - **Verification**: toàn frontend `91 passed`; production build pass; real-browser mocked-API smoke
    pass desktop/mobile/admin/non-admin, không có console issue. Docker daemon tắt nên chưa chạy
    full-stack browser smoke với MySQL thật.

- **JSON Ping — Phase P4 backend complete**
  - Thêm dedicated admin group và WebSocket `/api/ws/pings` + `/ws/pings`, xác thực JWT
    subprotocol `iot-jwt` và từ chối non-admin bằng close `1008`.
  - Ping commit/raw echo phát redacted `received`; delete commit phát `cleared` thread-safe từ
    worker thread. Events không chứa payload/credential và không đi telemetry groups.
  - RealtimeHub dọn failed socket, đóng/clear admin group khi stop; REST summary/delete, RBAC,
    atomic rollback và realtime flow đã hoàn tất end-to-end.
  - **Verification**: toàn nhóm ping `74 passed`; toàn backend `200 passed, 1 skipped`; MQTT
    implementation không thay đổi.

- **JSON Ping — PING-09 admin summary/delete REST APIs**
  - Thêm admin-only `GET /api/pings/{device_id}/summary` với aggregate totals và current
    payload theo database ID mới nhất.
  - Thêm atomic `DELETE /api/pings/{device_id}`: khóa Device row, xóa missing trước payload,
    trả exact deleted counts và reset inferred `predicted_order` về 1; zero state idempotent.
  - Typed response schemas, RBAC `403`, unknown device `404` và rollback khi commit lỗi đều
    được khóa bằng integration tests. Realtime event được giữ cho PING-10.
  - **Verification**: Ping APIs `8 passed`; toàn nhóm ping `68 passed`; toàn backend
    `194 passed, 1 skipped`.

- **JSON Ping — Phase P3 WebSocket integration**
  - Tích hợp text và binary UTF-8 JSON ping vào `/api/ws/esp32/{device_id}` với shared strict
    validation/persistence path; raw frame chỉ được echo sau DB commit và giữ nguyên frame type.
  - Ping hợp lệ/lỗi bypass ACK, presence, Influx/telemetry và không cập nhật `last_seen_at`;
    lỗi trả `ping_error` redacted nhưng giữ WebSocket connection sống.
  - Xóa WebSocket special-case `PING|`; MQTT legacy echo không thay đổi. Binary không phải
    JSON ping tiếp tục dùng binary/protobuf pipeline hiện có.
  - Bổ sung full validation/error integration matrix gồm UTF-8 size, ranges, strict types,
    extra field, unknown device, malformed JSON và case-sensitive sensor type.
  - **Verification**: schema + WebSocket `48 passed`; MQTT `2 passed, 1 skipped`; toàn backend
    `186 passed, 1 skipped`.

- **JSON Ping — Phase P2 strict validation và sequence persistence**
  - Thêm Pydantic v2 `PingMessage` strict, required toàn bộ field, cấm extra, kiểm tra range,
    device ID decimal dương và `size` theo UTF-8 byte length.
  - Error formatter chỉ trả field/reason, không lặp raw payload hoặc input lớn.
  - Thêm transaction service khóa Device row bằng `FOR UPDATE`, suy ra cycle/predicted order
    hoàn toàn từ DB, bulk insert tối đa 10.000 missing và rollback gap 10.001.
  - Duplicate/late order vẫn được lưu; late arrival xóa missing của current cycle; order 1
    sau lịch sử mở cycle mới. Lỗi DB trả message ổn định và rollback partial rows.
  - Phase này chưa nối service vào WebSocket và không thay đổi MQTT, telemetry, InfluxDB,
    presence, REST hoặc frontend.
  - **Verification**: schema `24 passed`; service `6 passed`; toàn backend
    `162 passed, 1 skipped`.

- **JSON Ping — PING-01 database foundation**
  - Thêm ORM `PingPayload` và `MissingPingPayload`, đăng ký trước startup
    `Base.metadata.create_all()` và giữ SQLite-compatible primary-key variant cho tests.
  - Thêm hai bảng với FK cascade, CHECK constraints, unique missing order và các composite
    index đã khóa trong `alltasks.md`.
  - Thêm clean-install DDL, migration additive `015_ping_payload_tracking.sql` và rollback
    chỉ xóa hai bảng mới theo thứ tự an toàn. `payload_id` là missing order, không phải FK.
  - Thêm ADR-0008 và contract tests chống drift giữa ORM/migration/clean schema.
  - PING-01 chưa thay đổi WebSocket, MQTT, telemetry, REST hoặc frontend.
  - **Verification**: contract `6 passed`; backend `132 passed, 1 skipped`; SQLite metadata
    creation pass. MySQL disposable dry-run chưa chạy vì máy hiện không có container/image
    MySQL.

## 2026-08-09

- **Anchor MQTT — delta coalesce riêng theo từng Gateway**
  - Mutation create/update/delete chỉ tạo event cho Anchor thay đổi; phần tử có
    `action=upsert|delete`. PATCH no-op trả `unchanged` và không tăng revision.
  - Dispatcher compose payload riêng từ `applied_revision` của từng Gateway, gộp theo
    Anchor ID với action cuối thắng; retry dùng payload delivery bất biến.
  - Full `replace` chỉ dùng cho bootstrap Gateway, resync có đích hoặc clear map. UI chuyển
    nút resync vào từng Gateway; endpoint resync toàn location cũ trả `410 Gone`.
  - Thêm `target_gateway_id` cho outbox và `payload` cho delivery qua migration additive
    `014_anchor_delta_delivery.sql`; startup repair hỗ trợ volume MySQL hiện có.
  - Chặn hai Gateway active dùng chung downlink topic; dữ liệu legacy trùng topic được đánh
    dấu `misconfigured` để ACK không bị nhập nhằng.
  - Sửa lỗi production `autoflush=False` khiến delivery đã ACK `applied` nhưng outbox cha
    còn `pending`; ACK idempotent giờ cũng tự sửa aggregate cũ.
  - **Verification**: backend `126 passed, 1 skipped`; frontend `79 passed` trên 23 test files;
    Vite/Docker build pass; migration 014 đã backfill không còn payload null; retained baseline
    revision 47 và ACK thực tế chuyển delivery/outbox sang `applied/completed`.

- **Anchor MQTT — loại bỏ Hardware ID khỏi downlink**
  - Mỗi phần tử trong `anchor_config.v1.anchors` chỉ còn `id`, `mac_address`, `name`, `x`, `y`, `z`;
    không còn gửi field `hardware_id` trùng lặp xuống Gateway.
  - Database và REST vẫn giữ `hardware_id` legacy để không phá dữ liệu/client cũ.
  - Bổ sung regression test xác nhận snapshot MQTT không chứa `hardware_id`.
  - **Verification**: backend `113 passed, 1 skipped`; Docker backend healthy; revision `r46`
    đã thay thế payload cũ và retained MQTT tại `gateway/136024/backend_send` không còn `hardware_id`.

- **Devices — hiển thị đúng loại Gateway**
  - Bổ sung mapping `device_type='gateway' → Gateway` cho card danh sách và trang chi tiết.
  - Gateway không còn rơi vào fallback Temperature hoặc sinh biểu đồ nhiệt độ trên trang chi tiết.
  - Thêm regression test từ dữ liệu API thực tế; frontend `79 passed`, Vite build pass.

- **Anchor editor — khôi phục tương phản nút Hủy**
  - Khai báo rõ nền xám nhạt và chữ xám đậm cho secondary action, tránh bị CSS toàn
    cục biến thành chữ trắng trên nền trắng.
  - Bổ sung trạng thái hover, focus-visible và disabled; thêm regression test cho style.
  - **Verification**: frontend `78 passed`; Vite production build pass.

- **Anchor — chuyển định danh phần cứng sang MAC Address**
  - Đổi UI tạo/sửa và danh sách quản lý từ **Hardware ID** sang **MAC Address**; input
    nhận sáu octet hexadecimal, chuẩn hóa chữ thường thành chữ hoa và báo lỗi định dạng sai.
  - REST, ORM, snapshot MQTT và search bổ sung `mac_address`; MAC trở thành bất biến sau
    khi gán. Anchor legacy có ID số được phép gán MAC một lần qua editor.
  - Thêm migration additive `013_anchor_mac_address.sql`: cột `VARCHAR(17)` nullable,
    unique index và backfill có điều kiện; giữ `hardware_id` để rollout không phá dữ liệu cũ.
  - Cập nhật clean schema, runtime migration, ERD tool, SRS và API/MQTT contract.
  - **Verification**: backend `113 passed, 1 skipped`; frontend `77 passed`; Vite và
    Docker backend/frontend build pass; MySQL existing volume và ERD Workbench đã xác minh.

- **Anchor — chuẩn hóa tọa độ 2 chữ số thập phân**
  - Thu gọn cụm input X/Y/Z còn nửa chiều rộng modal trên màn hình `sm` trở lên;
    màn hình nhỏ vẫn dùng toàn bộ chiều rộng để nhập liệu thuận tiện.
  - Input dùng bước `0.01`, không nhận chữ số thập phân thứ ba và chuẩn hóa dữ liệu cũ
    khi mở editor.
  - Marker có dữ liệu cũ và thao tác kéo trên map đều snap theo bước `0.01`; API làm
    tròn half-up trước khi lưu DB và tạo payload Gateway, ví dụ `8.325 → 8.33`.
  - **Verification**: backend `107 passed, 1 skipped`; frontend `74 passed` trên 22 test
    files; Vite production build pass; Chrome smoke xác nhận layout, nhập liệu, kéo-thả
    và console sạch.

## 2026-08-08

- **Đăng ký Gateway trực tiếp trên web**
  - Thêm loại **Gateway** vào form **Add New Device**, hiển thị Device ID cùng topic uplink/downlink.
  - Topic mặc định là `gateway/{device_id}/backend_receive` và
    `gateway/{device_id}/backend_send`; admin vẫn có thể nhập topic riêng.
  - Backend áp dụng cùng default contract kể cả client không gửi topic và subscribe uplink
    ngay sau khi tạo, không cần restart để nhận ACK.
  - **Verification**: backend `106 passed, 1 skipped`; frontend `73 passed` trên 22 test files;
    Vite production build pass; Chrome smoke xác nhận topic/payload và console sạch.

- **GPS Dashboard — bố cục full-bleed ba vùng**
  - Chuyển dashboard submenu về hàng ngang và bỏ `max-width`, margin/card wrapper của route GPS.
  - Tách thao tác Hệ thống sang sidebar trái 240px, map co giãn ở giữa và danh sách thiết bị
    sang panel phải 300px; giữ nguyên luồng upload của nút **Thêm bản đồ**.
  - Ảnh mặt bằng tự fit trọn vẹn theo cả chiều rộng và chiều cao vùng map bằng
    `ResizeObserver`, giữ tỷ lệ và không cần thanh cuộn; bỏ nền lưới quanh map.
  - Dưới 1024px, Hệ thống và Thiết bị dùng drawer với target thao tác tối thiểu 44px.
  - Chưa triển khai tầng, database tầng, đa-map hoặc kéo-thả/sắp xếp map.

- **Cấu hình Anchor — Phase 6: integration, security và rollout readiness**
  - Thêm Gateway simulator và test Mosquitto thật cho retained QoS 1 cùng ACK applied/rejected.
  - Xác minh migration `012` chạy lặp trên MySQL 8.4 disposable database; cập nhật ERD
    và giữ backup `D:\iot\mysql\erd.before-phase5-20260808.mwb`.
  - Chrome 375×800 phát hiện panel bị navbar chặn; đổi panel mobile thành fixed-centered
    z-index 60 rồi smoke lại với console sạch và đúng một resync request.
  - Vá package lock loại `nanoid` high/PostCSS moderate; Mosquitto bắt buộc password auth.
  - Thêm runbook, test report và cập nhật OpenAPI-like/SRS/ADR/architecture/inventory.
  - Hardening vòng review cuối: khóa trạng thái ACK terminal trước ACK mâu thuẫn đến trễ,
    audit stale ACK, loại delivery khi Gateway rời location, claim song song bằng
    `FOR UPDATE SKIP LOCKED` và không đưa ACK vào pipeline telemetry.
  - **Verification**: backend `104 passed, 1 skipped` + MQTT opt-in `1 passed`;
    frontend `65 passed`; Vite và Docker backend/frontend build pass; không còn high/critical.

- **Cấu hình Anchor — Phase 5: lifecycle và Gateway reconciliation**
  - Map/group/user deletion soft-delete Anchor và tạo empty snapshot trong cùng transaction
    trước khi archive active map.
  - Chuyển `anchor.location_id` thành snapshot ID, thêm migration `012` và startup repair
    để inactive Anchor audit không bị FK chặn hoặc cascade mất khi map bị archive.
  - Device admin create/patch/topic/delete reconcile revision mới nhất; thay publish topic
    tự enqueue lại, inactive/chuyển location loại khỏi aggregate cũ.
  - Dispatcher định kỳ reconcile cả snapshot đã completed để Gateway thêm sau vẫn nhận latest.
  - **Verification**: full backend `104 passed`.

- **Cấu hình Anchor — Phase 4: MQTT delivery, liveness, ACK và sync UI**
  - Thêm dispatcher transactional outbox với lease, retry, restart recovery và publish
    MQTT QoS 1 retained một lần cho mỗi revision/topic.
  - Reconcile Gateway active theo location; Gateway thiếu publish topic xuất hiện
    `misconfigured` và broker offline không làm mất delivery.
  - Theo dõi liveness từ MQTT/WS đã xác thực, throttle ghi `last_seen_at` 5 giây và
    tính online bằng server time với timeout mặc định 30 giây.
  - Nhận ACK MQTT/WS, validate schema/gateway/topic/location/revision, cập nhật idempotent;
    thêm status/resync API và aggregate `synced|partial|pending|error|no_gateway`.
  - GPS toolbar có badge, chi tiết từng Gateway, polling 5 giây và nút gửi lại cấu hình;
    member chỉ-view không gọi endpoint quản trị.
  - **Verification**: backend `101 passed`; frontend Phase 4 và toàn bộ GPS regression
    tests pass.

## 2026-08-07

- **Cấu hình Anchor — Phase 3: danh sách quản lý và bulk invitations**
  - Thêm dialog quản lý Anchor với search debounce 300 ms, filter group/map, page size
    25 và đầy đủ DB ID/hardware/group/map/x/y/z/updated time.
  - Row trong danh sách chuyển đúng group/map, chờ ảnh và Anchor fresh trước khi mở
    editor; hủy an toàn nếu resource hoặc quyền thay đổi giữa lúc tải.
  - Thêm bulk invitation tối đa 50 username, trim/dedupe xử lý, exact-case lookup,
    partial success và mã kết quả từng user; UI nhận multiline/comma input.
  - **Verification**:
    - Backend `94 passed`; frontend `62 passed` trên 18 test files; Vite build pass.
    - Chrome smoke test xác nhận debounce/search, list-to-map editor, PATCH/bulk payload,
      partial result, dialog 375 px và console sạch.
    - `npm audit --omit=dev`: không có critical/high; còn 3 moderate hiện hữu từ
      React Router/PostCSS, không thay dependency ngoài phạm vi Phase 3.

- **Cấu hình Anchor — Phase 2: CRUD và editor trên map**
  - Thêm REST list/create/get/patch/soft-delete và management search có filter/pagination,
    với policy viewer/member và mutation admin hoặc owner có cờ `yes`.
  - Mỗi mutation tạo một full snapshot `anchor_config.v1` theo Anchor active, cùng
    transaction với dữ liệu nghiệp vụ; revision mới supersede delivery cũ chưa applied.
  - GPS Dashboard tải/cancel Anchor theo map, render marker riêng và giữ nguyên marker
    thiết bị realtime; viewer marker không tương tác.
  - Thêm editor tạo/sửa/xóa, draft `(50,50,0)`, numeric input, kéo-thả có clamp và
    xử lý thu hồi quyền 403 bằng refresh session + ẩn control.
  - **Verification**:
    - Backend `91 passed`; frontend `55 passed` trên 16 test files; Vite build pass.
    - Chrome smoke test xác nhận marker, draft mặc định, đúng một POST khi Save, marker
      mới và console/page error sạch.

- **Cấu hình Anchor — Phase 1: phân quyền end-to-end**
  - Mở rộng contract register/user/login/me với `can_config_anchor=yes|no`, mặc định
    `no`; cờ không được đưa vào JWT.
  - Thêm endpoint admin-only `PATCH /api/users/{user_id}/anchor-permission` và helper
    quyền: admin bypass; user thường cần active, chưa hết hạn, là owner và có cờ `yes`.
  - Trang quản lý user có switch khi tạo user và trên card user thường; admin hiển thị
    quyền cố định. Request lỗi rollback switch và hiển thị alert.
  - Bổ sung semantics dialog/switch và accessible name; cập nhật profile JSDoc trong
    `AuthContext`.
  - **Verification**:
    - Backend `85 passed`; frontend `44 passed` trên 13 test files; Vite build pass.
    - Chrome smoke test xác nhận switch mặc định tắt, admin copy, PATCH payload,
      accessibility tree và console sạch.
    - `npm audit --omit=dev`: không có critical/high; còn 3 moderate hiện hữu từ
      React Router/PostCSS, chưa thay dependency ngoài phạm vi Phase 1.

- **Cấu hình Anchor — Phase 0: baseline, schema và runtime foundation**
  - Đồng bộ clean-install schema với bốn bảng map và bổ sung migration map image
    contract còn thiếu; giữ nguyên migration xóa test log hiện hữu.
  - Thêm migration `011_anchor_configuration.sql` cùng các bảng `anchor`,
    `anchor_config_outbox`, `anchor_config_delivery`, cột `user.can_config_anchor`
    mặc định `no` và `device.last_seen_at`.
  - Thêm ORM tương ứng, startup patch idempotent cho existing MySQL volume và settings
    liveness/dispatcher/retry; chưa khởi chạy dispatcher và chưa mở API/UI Anchor.
  - Cập nhật `D:\iot\mysql\erd.mwb` bằng MySQL Workbench 8.0.47, loại
    `anchor_group_member`; giữ backup timestamp độc lập với `.bak` của user.
  - **Verification**:
    - Backend `81 passed`; frontend `41 passed` trên 12 test files; Vite build pass.
    - Clean schema và migration `011` đều chạy thành công hai lần liên tiếp trên MySQL 8.4.
    - ERD mở lại và forward-engineer đúng contract cho ba bảng Anchor chính.

- **Đặc tả và kế hoạch feature cấu hình Anchor (ghi nhận trước triển khai)**
  - Chốt ma trận quyền: admin luôn cấu hình được; user thường cần
    `can_config_anchor = 'yes'` đồng thời là owner group; accepted member chỉ xem.
  - Chốt UX tạo/sửa/kéo/xóa Anchor, tọa độ phần trăm `(x, y, z)`, danh sách tìm kiếm
    phân trang và lời mời nhiều thành viên.
  - Chốt cơ chế đồng bộ Gateway bằng full snapshot versioned, MQTT QoS 1 retained,
    transactional outbox, retry/supersede, ACK và liveness.
  - Ghi toàn bộ phase/task/dependency/DoD vào `alltasks.md`.
  - Thêm SRS riêng `srs/anchor-configuration-spec.md`, hợp đồng REST/MQTT
    `api/anchor-configuration-api.md` và ADR-0007; cập nhật SRS tổng, kiến trúc và mục lục docs.
  - Riêng mục lập kế hoạch này không thay đổi runtime/database; phần Phase 0 ở trên là
    implementation foundation được thực hiện sau khi spec được chốt.

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
