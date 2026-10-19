Tại sao phải truyền Header X-Forwarded-For?
Khi client (người dùng) gửi request đến hệ thống, request đó đi qua Nginx trước rồi mới được chuyển tiếp (proxy) tới Backend (API Gateway). Nginx đóng vai trò là một "người trung gian". Header X-Forwarded-For được sinh ra để Nginx có thể "báo cáo" lại cho Backend biết địa chỉ IP thực sự của người dùng ban đầu là gì.

Nếu không truyền, Backend sẽ thấy IP của ai?
Nếu bạn không truyền header này (và X-Real-IP), Backend sẽ coi Nginx chính là client đang gọi trực tiếp tới nó. Do Nginx và Backend đang chạy trên cùng một máy chủ, Backend sẽ chỉ nhìn thấy địa chỉ IP là 127.0.0.1 (localhost) đối với mọi request.