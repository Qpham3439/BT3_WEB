# BT3_WEB
Bài tập 03 của sinh viên: K225480106057 - Phạm Mạnh Quỳnh - môn Phát Triển Ứng Dụng Trên Nền WEB
Giảng viên: Đỗ Duy Cốp

Lớp học phần: 58KTP

Sinh viên thực hiện: Nguyễn Như Khiêm

Chủ đề: Lập trình ứng dụng web thương mại điện tử trên nền Linux (Docker + Hyper-V + Ubuntu)

-----------------------------------------------------------------------------------------------------
1. GIỚI THIỆU CHUNG:
Bài tập yêu cầu xây dựng một ứng dụng web thương mại điện tử dạng Single Page Application (SPA), triển khai trên Linux (Ubuntu) chạy trong Hyper-V.

Sử dụng Docker Compose để quản lý các container:
- mariadb – cơ sở dữ liệu lưu user, sản phẩm, đơn hàng
- phpmyadmin – giao diện quản trị DB
- nodered – backend xử lý request, trả JSON
- grafana – hiển thị thống kê sản phẩm bán chạy
- influxdb – lưu lịch sử thống kê (nếu cần)
- nginx – web server reverse proxy
----------------------------------------------------------
2. CẤU TRÚC DỰ ÁN
```
/home/quynh/web-ecommerce/  
│
├── docker-compose.yml             # File chính khai báo toàn bộ container
│
├── nginx/
│   ├── default.conf               # File cấu hình nginx (reverse proxy, domain)
│   └── certs/                     # Nếu sau này thêm SSL
│
├── node-red/
│   ├── data/                      # Lưu flow.json, settings.js, node_modules...
│
├── mariadb/
│   ├── data/                      # Lưu database của MariaDB
│
├── influxdb/
│   ├── data/                      # Dữ liệu time-series cho Grafana
│
├── grafana/
│   ├── data/                      # Lưu config, dashboards, users...
│
├── phpmyadmin/                    # (tuỳ chọn, không cần data riêng)
│
└── web/
    ├── index.html                 # Single Page Application chính
    ├── js/
    │   ├── app.js                 # Logic xử lý giao diện + gọi API nodered
    │   ├── login.js               # Xử lý đăng nhập
    │   └── cart.js                # Giỏ hàng, đặt hàng
    ├── css/
    │   └── style.css
    └── assets/
        └── images/  
```
