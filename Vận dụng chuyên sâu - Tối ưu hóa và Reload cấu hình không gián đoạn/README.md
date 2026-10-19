### 1. Cấu hình Nginx tối ưu Upload và Gzip
Để khắc phục lỗi `413 Request Entity Too Large` và tăng tốc độ tải trang, bạn cần bổ sung chỉ thị `client_max_body_size` và block cấu hình `gzip`. Các cấu hình này có thể đặt trong block `http {}` (của file `/etc/nginx/nginx.conf` để áp dụng toàn cục) hoặc đặt trong block `server {}` (của file site cụ thể) như sau:

```nginx
server {
    listen 80;
    server_name my-ecommerce.com;

    # 1. Tối ưu hóa Upload: Cho phép gửi request body (upload file) lên tới 10MB
    client_max_body_size 10M;

    # 2. Tối ưu hóa tốc độ tải trang: Bật nén Gzip
    gzip on;
    gzip_disable "msie6"; # Tắt nén cho các trình duyệt IE quá cũ
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6; # Mức độ nén từ 1-9 (6 là cân bằng tốt nhất giữa CPU và dung lượng)
    gzip_buffers 16 8k;
    gzip_http_version 1.1;
    # Chỉ định các loại file text sẽ được nén (Không nén file ảnh, video vì đã được nén sẵn)
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}