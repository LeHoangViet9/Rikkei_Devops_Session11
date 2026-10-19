Nếu ứng dụng React ở port 3000 bị sập (tiến trình bị tắt, crash, hoặc chưa được bật), Nginx sẽ trả về mã lỗi 502 Bad Gateway.

Giải thích:
Khi Nginx (đóng vai trò là Gateway/Proxy) cố gắng kết nối tới backend (localhost:3000) thông qua chỉ thị proxy_pass nhưng backend này từ chối kết nối hoặc không phản hồi (do đã sập), Nginx hiểu rằng server phía sau đang gặp sự cố. Do đó, nó trả về lỗi 502 để báo cho client biết rằng nó không thể lấy được dữ liệu từ máy chủ gốc.