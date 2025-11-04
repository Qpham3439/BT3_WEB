# BT3_WEB
Bài tập 03 của sinh viên: K225480106057 - Phạm Mạnh Quỳnh - môn Phát Triển Ứng Dụng Trên Nền WEB
Giảng viên: Đỗ Duy Cốp

Lớp học phần: 58KTP

Sinh viên thực hiện: Phạm Mạnh Quỳnh

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
-----------------------------------------------------------------------------------
3. CÀI ĐẶT MÔI TRƯỜNG:

Bước 1: Cài đặt Ubuntu
- Chạy PowerShell với quyền admin (Run as Administrator)
- Gõ lệnh:
```wsl --install```
-> Hệ thống sẽ tự cài WSL
<img width="1507" height="782" alt="Screenshot 2025-11-04 184910" src="https://github.com/user-attachments/assets/6c336fe3-7f63-499a-a29d-d8ca7919b759" />
- Khởi động lại máy.
- Gõ lệnh:
```wsl --install -d Ubuntu```
-> Hệ thống sẽ tự cài Ubuntu:
  <img width="1509" height="777" alt="Screenshot 2025-11-04 190220" src="https://github.com/user-attachments/assets/4e91171e-d120-4856-88df-93bbfe971c41" />
- Chạy lsb_release -a để xác nhận Ubuntu chạy.
<img width="690" height="176" alt="Screenshot 2025-11-04 191107" src="https://github.com/user-attachments/assets/0a890555-2db9-4ffa-af09-adc8b3a5fc8e" />
- chuyển đến thư mục chính:
<img width="723" height="137" alt="Screenshot 2025-11-04 191319" src="https://github.com/user-attachments/assets/9c5b97d0-7839-4c3c-ad49-81ffb61643b1" />

Bước 2: Cài đặt Docker và Docker Compose

- Cài Docker engine và thêm user vào group docker (để chạy docker không cần sudo):
<img width="1508" height="662" alt="Screenshot 2025-11-04 192439" src="https://github.com/user-attachments/assets/29ef206e-eef0-4b11-8042-de929cad091f" />
- Áp dụng thay đổi: Gõ exit để logout, rồi login lại (gõ username/password như trước). Hoặc reboot nhanh: sudo reboot.
<img width="1018" height="119" alt="Screenshot 2025-11-04 192920" src="https://github.com/user-attachments/assets/7c4c7f46-1468-47fb-bb95-87fa72e7929d" />
- Test không sudo:
```docker run hello-world```

Nếu thấy "Hello from Docker!", là thành công (không cần sudo nữa).

<img width="1279" height="661" alt="Screenshot 2025-11-04 193135" src="https://github.com/user-attachments/assets/cdd7a140-967a-49eb-af9c-4d07a20ce81d" />

4. CẤU HÌNH DOCKER-COMPOSE

Tạo file docker-compose.yml:
```
 version: '3.9'

services:
  mariadb:
    image: mariadb:latest
    container_name: mariadb
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: ecommerce
      MYSQL_USER: quynh
      MYSQL_PASSWORD: 1234
    ports:
      - "3306:3306"
    volumes:
      - ./mariadb/data:/var/lib/mysql

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: phpmyadmin
    restart: always
    environment:
      PMA_HOST: mariadb
      MYSQL_ROOT_PASSWORD: root
    ports:
      - "8080:80"
   depends_on:
      - mariadb

  nodered:
    image: nodered/node-red:latest
    container_name: nodered
    restart: always
    ports:
      - "1880:1880"
    volumes:
      - ./node-red/data:/data
    depends_on:
      - mariadb

  influxdb:
    image: influxdb:latest
    container_name: influxdb
    restart: always
    ports:
      - "8086:8086"
    volumes:
      - ./influxdb/data:/var/lib/influxdb
 grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SERVER_ROOT_URL=%(protocol)s://%(domain)s/grafana
    volumes:
      - ./grafana/data:/var/lib/grafana
    restart: always
 nginx:
    image: nginx:latest
    container_name: nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
      - ./web:/usr/share/nginx/html
    depends_on:
      - phpmyadmin
      - nodered
      - grafana
```

Chạy toàn bộ các container:```docker compose up -d```
<img width="1491" height="384" alt="Screenshot 2025-11-04 194458" src="https://github.com/user-attachments/assets/66a2e89a-5021-4db8-917b-f713b24e1b26" />

5. CẤU HÌNH NGINX

File nginx/default.conf:

```
   server {
    listen 80;
    server_name phammanhquynh.com localhost;

    # Trang chính (web tĩnh)
    location / {
        root /usr/share/nginx/html;
        index index.html;
        try_files $uri $uri/ =404;
    }

    # Node-RED
    location /nodered/ {
        proxy_pass http://nodered:1880/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }

    # Grafana
    location /grafana/ {
        proxy_pass http://grafana:3000/;
        proxy_set_header Host $host;
    }
    # phpMyAdmin
    location /phpmyadmin/ {
        proxy_pass http://phpmyadmin:80/;
        proxy_set_header Host $host;
    }
}
```
6. 
