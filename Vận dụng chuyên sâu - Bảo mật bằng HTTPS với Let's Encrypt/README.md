### 1. Chuỗi lệnh cài đặt Certbot trên Ubuntu
```bash
# Cập nhật danh sách các package
sudo apt update

# Cài đặt certbot và plugin dành cho nginx
sudo apt install certbot python3-certbot-nginx -y
2. Lệnh cấp phát SSL tự động
Để Certbot tự động xác thực tên miền và sửa file cấu hình Nginx, sử dụng lệnh sau:

Bash
sudo certbot --nginx -d api.my-ecommerce.com
3. Cấu hình Nginx viết tay cho HTTPS (Port 443)
Nếu không dùng Certbot tự động cấu hình mà tự viết tay, file cấu hình sẽ yêu cầu trỏ đường dẫn tới ssl_certificate và ssl_certificate_key như sau:

Nginx
# 1. Block tự động chuyển hướng HTTP (80) sang HTTPS (443)
server {
    listen 80;
    server_name api.my-ecommerce.com;
    return 301 https://$host$request_uri;
}

# 2. Block xử lý HTTPS (443)
server {
    listen 443 ssl;
    server_name api.my-ecommerce.com;

    # Trỏ tới file Public Key / Certificate
    ssl_certificate /etc/letsencrypt/live/[api.my-ecommerce.com/fullchain.pem](https://api.my-ecommerce.com/fullchain.pem);
    
    # Trỏ tới file Private Key
    ssl_certificate_key /etc/letsencrypt/live/[api.my-ecommerce.com/privkey.pem](https://api.my-ecommerce.com/privkey.pem);

    location / {
        # Proxy tới Backend API Gateway
        proxy_pass http://localhost:8080;
        
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
4. Bẫy: Thiết lập tự động gia hạn (renew) chứng chỉ
Trả lời:
Khi cài đặt certbot qua package manager (apt) trên Ubuntu, hệ thống đã tự động sinh ra một systemd timer để chạy ngầm kiểm tra và gia hạn chứng chỉ 2 lần mỗi ngày.

Lệnh kiểm tra tiến trình tự động gia hạn của hệ thống: sudo systemctl status certbot.timer

Nếu thiết lập thủ công bằng Cronjob:
Mở crontab bằng lệnh sudo crontab -e và thêm cấu hình sau để tự động gia hạn và reload lại Nginx:

Bash
0 0,12 * * * /usr/bin/certbot renew --quiet --post-hook "systemctl reload nginx"