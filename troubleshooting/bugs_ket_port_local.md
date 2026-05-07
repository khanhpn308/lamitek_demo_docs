
# Bug: Bị “kẹt cổng” (port) khiến LAN không vào được

Tài liệu này tổng hợp các lỗi đã gặp và đã xử lý thành công khi triển khai stack Docker + WSL2 trên Windows, trong đó **cổng API (ví dụ 8000)** bị một cơ chế chuyển tiếp/tiến trình Windows “giữ” sai cách.

## 1) Dấu hiệu nhận biết

### A. Truy cập bằng `localhost` OK nhưng bằng IP LAN thì FAIL

- Truy cập từ máy Windows host:
	- `http://localhost:8000/api/health` chạy được
	- `ws://localhost:8000/api/ws/esp32/<device_id>` chạy được
- Nhưng từ máy khác trong LAN hoặc từ Postman dùng IP LAN của máy host:
	- `http://192.168.x.x:8000/api/health` không vào được
	- `ws://192.168.x.x:8000/api/ws/esp32/<device_id>` báo lỗi kiểu **“socket hang up”**, **Upgrade header error**, hoặc timeout
- Backend gần như **không log** được attempt connect khi bạn gọi qua IP LAN (vì request không tới được Uvicorn/FastAPI).

### B. `netstat` cho thấy port chỉ listen ở loopback IPv6

- Chạy:
	- `netstat -ano | findstr :8000`
- Nếu thấy dạng:
	- `TCP    [::1]:8000    ...    LISTENING    <PID>`
	- và **không** thấy `0.0.0.0:8000 LISTENING`

=> Đây là dấu hiệu Windows chỉ mở cổng trên loopback (localhost), LAN sẽ không vào được.

### C. `portproxy` tồn tại và trỏ sai (tự loop hoặc chặn Docker)

- Chạy:
	- `netsh interface portproxy show all`
- Nếu thấy rule kiểu:
	- `0.0.0.0 8000  ->  127.0.0.1 8000`

=> Rule này rất dễ gây hiện tượng “kẹt port”/đi vòng vòng, hoặc **svchost/iphlpsvc** đứng ra listen thay vì Docker.

## 2) Cách kiểm tra nhanh (chẩn đoán)

Chạy CMD (khuyến nghị **Run as Administrator** để xem đầy đủ):

1) Xem port đang do tiến trình nào giữ:
- `netstat -ano | findstr :8000`

2) Nếu có PID, xem PID thuộc service nào:
- `tasklist /svc /fi "PID eq <PID>"`

3) Kiểm tra có portproxy không:
- `netsh interface portproxy show all`

4) Xác định IP LAN của máy host (ESP32/Postman sẽ dùng IP này):
- `ipconfig`

5) (Tuỳ chọn) Xác định IP/route bên WSL:
- `wsl hostname -I`

## 3) Cách khắc phục (đã dùng và thành công)

### Case 1: Có rule portproxy v4→v4 sai (0.0.0.0:8000 → 127.0.0.1:8000)

**Dấu hiệu:** `portproxy show all` thấy rule như trên; `netstat` có thể cho thấy `0.0.0.0:8000` do `svchost.exe` listen.

**Cách xử lý:**

1) Xoá rule cụ thể:
- `netsh interface portproxy delete v4tov4 listenport=8000 listenaddress=0.0.0.0`

2) Kiểm tra lại:
- `netsh interface portproxy show all`

3) Kiểm tra listener:
- `netstat -ano | findstr :8000`

Nếu sau khi xoá rule mà port lại chỉ còn `[::1]:8000`, xem Case 2.

### Case 2: Port chỉ listen `[::1]:8000` (localhost OK, LAN FAIL)

**Dấu hiệu:**
- `netstat -ano | findstr :8000` chỉ ra `TCP [::1]:8000 LISTENING ...`

**Mục tiêu:** làm cho Windows **listen IPv4 0.0.0.0:8000** và forward vào loopback IPv6 `::1:8000`.

**Cách xử lý (khuyến nghị): dùng portproxy v4→v6**

1) Reset toàn bộ portproxy (cẩn thận: xoá tất cả rule portproxy hiện có):
- `netsh interface portproxy reset`

2) Tạo rule mới IPv4 → IPv6 loopback:
- `netsh interface portproxy add v4tov6 listenaddress=0.0.0.0 listenport=8000 connectaddress=::1 connectport=8000`

3) Mở firewall inbound cho port 8000:
- `netsh advfirewall firewall add rule name="IoT Backend 8000" dir=in action=allow protocol=TCP localport=8000`

4) Kiểm tra lại:
- `netsh interface portproxy show all`
- `netstat -ano | findstr :8000`

**Kỳ vọng:** thấy `0.0.0.0:8000 LISTENING` (thường do `svchost.exe`/`iphlpsvc` listen là bình thường trong trường hợp dùng portproxy).

5) Test từ LAN:
- HTTP: `http://192.168.x.x:8000/api/health`
- WebSocket: `ws://192.168.x.x:8000/api/ws/esp32/<device_id>`

### Case 3 (thay thế): Dùng port 80 qua Nginx thay vì expose 8000

Nếu bạn đã publish frontend/nginx ra port 80 và cấu hình proxy WebSocket đúng, có thể cho ESP32/Postman kết nối qua:

- `ws://192.168.x.x/ws/esp32/<device_id>`
hoặc
- `ws://192.168.x.x/api/ws/esp32/<device_id>`

Khi đó bạn chỉ cần đảm bảo:
- `netstat -ano | findstr :80` có `0.0.0.0:80 LISTENING`
- Firewall cho port 80

## 4) Ghi chú quan trọng

- Nếu **localhost OK nhưng IP LAN FAIL** thì gần như chắc chắn là vấn đề **binding/port forwarding của Windows/WSL/Docker**, không phải route FastAPI.
- Portproxy là “dao hai lưỡi”: rule sai (đặc biệt `0.0.0.0:8000 -> 127.0.0.1:8000`) dễ gây kẹt cổng.
- Sau khi đổi portproxy/firewall, nên test lại theo thứ tự: `netstat` → `health` HTTP → WebSocket.

