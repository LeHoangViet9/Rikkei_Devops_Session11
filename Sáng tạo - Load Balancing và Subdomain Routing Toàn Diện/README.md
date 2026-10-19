**Tình huống:** *Một developer đề xuất: "Chúng ta đã có Spring Cloud Gateway đảm nhiệm việc gom API rồi, tại sao vẫn phải tốn tài nguyên chạy thêm Nginx ở phía trước nó nữa? Bỏ Nginx đi, mở cổng 80 cho Spring Gateway luôn!".*

Dưới góc độ của một **Software Architect**, tôi phản biện và đưa ra quyết định kiến trúc như sau: **Chúng ta KHÔNG THỂ VÀ KHÔNG NÊN bỏ Nginx.** 

Mặc dù cả Nginx và Spring Cloud Gateway đều có khả năng "định tuyến" (routing), nhưng chúng được thiết kế để giải quyết các bài toán ở hai tầng (layer) hoàn toàn khác nhau trong hệ thống. Việc gom chung mọi thứ vào Spring Gateway là một "Anti-pattern" (thiết kế lỗi) trong Microservices. Dưới đây là phân tích chi tiết:

### 1. Thế mạnh của Nginx (Hoạt động ở Edge Network - Rìa mạng)
Nginx đóng vai trò là "Vỏ bọc thép", là điểm chạm đầu tiên chặn mọi sóng gió từ Internet trước khi request lọt vào ứng dụng Java.
* **SSL/TLS Termination:** Quá trình mã hóa/giải mã HTTPS tiêu tốn rất nhiều CPU. Nginx (được viết bằng C) tối ưu hóa việc này cực kỳ tốt ở tầng lõi HĐH. Thay vì bắt máy ảo Java (JVM) của Spring phải gánh vác việc giải mã SSL, ta giao việc "tay chân" này cho Nginx, sau đó chuyển request dạng HTTP thuần (nhẹ nhàng) vào bên trong.
* **Phục vụ file tĩnh (Static Assets):** Hệ thống E-commerce có rất nhiều ảnh, file JS, CSS của React/Vue. Nginx là "ông hoàng" trong việc phân phối file tĩnh và Caching. Spring Cloud Gateway không sinh ra để phục vụ file tĩnh, nếu dùng nó sẽ rất lãng phí tài nguyên RAM và CPU của JVM.
* **Bảo mật tầng mạng (Network-level Security):** Nginx cực kỳ hiệu quả trong việc chặn các đợt tấn công DDoS (tầng 4 và tầng 7), chặn các IP độc hại, hoặc giới hạn request (Rate Limiting) dựa trên IP. Việc chặn này xảy ra ở rìa mạng, giúp bảo vệ toàn bộ cụm Microservices bên trong không bị quá tải.

### 2. Thế mạnh của Spring Cloud Gateway (Hoạt động ở Application Layer - Tầng ứng dụng)
Spring Cloud Gateway là "Bộ não định tuyến", nó không rành về mạng lưới (network) nhưng cực kỳ hiểu về "nghiệp vụ" (business logic).
* **Bảo mật tầng ứng dụng:** Thay vì chặn theo IP như Nginx, Spring Gateway đọc nội dung Request để Validate JWT Token, xác thực người dùng (Authentication) và kiểm tra quyền (Authorization) trước khi cho phép đi vào các Service con.
* **Tích hợp Service Discovery (Eureka/Consul):** Spring Gateway nằm trong hệ sinh thái Java/Spring. Nó có thể kết nối với Eureka để biết chính xác hiện tại có bao nhiêu instance của `Order-Service` đang sống, port động của chúng là gì để tự động Load Balancing, điều mà Nginx (bản miễn phí) rất khó làm linh hoạt.
* **Filter nghiệp vụ phức tạp:** Gateway có thể can thiệp sâu vào luồng dữ liệu như: thay đổi Request/Response Body, gắn thêm Header cho nội bộ, Rate Limit theo UserID (chứ không phải theo IP), và áp dụng Circuit Breaker (Resilience4j) để ngắt mạch nếu một Service bị lỗi.

---

### 3. Sơ đồ luồng dữ liệu chuẩn (Data Flow) trong Microservices

Để thấy rõ sự phân chia trách nhiệm, đây là luồng dữ liệu chuẩn từ Client đến hệ thống:

```text
[ Client (Browser/Mobile App) ]
         |
         | (HTTPS - Port 443) 
         v
+---------------------------------------------------+
|                   NGINX (Rìa mạng)                |
| - SSL Termination (Giải mã HTTPS -> HTTP)         |
| - Phục vụ file tĩnh của Frontend (ReactJS)        |
| - Chặn DDoS IP, Caching ảnh/video                 |
+---------------------------------------------------+
         |
         | (HTTP - Chỉ định tuyến các request /api/)
         v
+---------------------------------------------------+
|            SPRING CLOUD GATEWAY (Port 8080)       |
| - Xác thực JWT Token                              |
| - Rate limit theo UserID, Routing nghiệp vụ       |
| - Giao tiếp với Eureka Server để tìm Service      |
+---------------------------------------------------+
         |
         | (Định tuyến nội bộ - Load Balancing)
         v
+---------------------------------------------------+
|               BACKEND MICROSERVICES               |
|  [ User Service ]  [ Product Service ]  [ Order ] |
+---------------------------------------------------+