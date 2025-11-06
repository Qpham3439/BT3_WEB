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
- Ghi chú: trong bài dùng các cổng có khác so với yêu cầu.
- lý do: do dùng chung máy để chạy cho nên các cổng không thể trùng nhau, nên phải thay đổi cổng để chương trình có thể chạy.
-----------------------------------------------------------------------------------
2. CÀI ĐẶT MÔI TRƯỜNG:

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

3. CẤU HÌNH DOCKER-COMPOSE

Tạo file docker-compose.yml:
```
version: '3.8'

services:
  mariadb:
    image: mariadb:latest
    container_name: mariadb_1
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: appdb
      MYSQL_USER: user
      MYSQL_PASSWORD: user123
    volumes:
      - ./data/mariadb_1:/var/lib/mysql   
    ports:
      - "3310:3306"
  
  phpmyadmin:
    image: phpmyadmin:latest
    container_name: phpmyadmin_1
    restart: always
    environment:
      PMA_HOST: mariadb_1  
      PMA_USER: root
      PMA_PASSWORD: root
    ports:
      - "8082:80"
    depends_on:
      - mariadb

  nodered:
    image: nodered/node-red:latest
    container_name: nodered_1
    restart: always
    ports:
      - "1882:1880"
    volumes:
      - ./data/nodered_1:/data

  influxdb:
    image: influxdb:latest
    container_name: influxdb_1
    restart: always
    ports:
      - "8088:8086"
    volumes:
      - ./data/influxdb_1:/var/lib/influxdb

  grafana:
    image: grafana/grafana:latest
    container_name: grafana_1
    restart: always
    ports:
      - "3002:3000"
    depends_on:
      - influxdb
    volumes:
      - ./data/grafana_1:/var/lib/grafana

  nginx:
    image: nginx:latest
    container_name: nginx_1
    restart: always
    ports:
      - "82:80"
      - "446:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./web:/usr/share/nginx/html:ro
    depends_on:
      - nodered
      - grafana

```

Chạy toàn bộ các container:```docker compose up -d```
<img width="1491" height="384" alt="Screenshot 2025-11-04 194458" src="https://github.com/user-attachments/assets/66a2e89a-5021-4db8-917b-f713b24e1b26" />

4. CẤU HÌNH NGINX

File nginx/default.conf:

```
   events {}

http {
  server {
    listen 80;
    server_name phammanhquynh.com;

    # Website chính (index.html)
    location / {
      root /usr/share/nginx/html;
      index index.html;
      try_files $uri $uri/ =404;
    }

    # --- Proxy tới Node-RED ---
    location /nodered/ {
      proxy_pass http://nodered:1880/;
      proxy_http_version 1.1;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_cache_bypass $http_upgrade;

      # Xử lý redirect nội bộ để không mất /nodered/
      proxy_redirect / /nodered/;
    }

    # --- Proxy tới Grafana ---
    location /grafana/ {
      proxy_pass http://grafana:3000/;
      proxy_http_version 1.1;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_cache_bypass $http_upgrade;

      # Giữ prefix /grafana/
      proxy_redirect / /grafana/;
    }
  }
}

```
- website: phammanhquynh.com
- nodered: phammanhquynh.com/nodered
- grafana: phammanhquynh.com/grafana
- phpMyadmin: phammanhquynh.com/phpmyadmin
5. WEB FRONTEND + BACKEND:

6. 
