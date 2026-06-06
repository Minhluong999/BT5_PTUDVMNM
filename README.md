# BT5 - Docker Compose: App Monitor + Alert Data Realtime

## 1. Phần lý thuyết

### 1.1. Docker là gì?

Docker là nền tảng dùng để đóng gói, triển khai và chạy ứng dụng trong các **container**.

Container là môi trường chạy độc lập, bên trong có đầy đủ:

- Mã nguồn ứng dụng
- Thư viện cần thiết
- Runtime
- Cấu hình môi trường
- Các dependency liên quan

Nhờ Docker, ứng dụng có thể chạy ổn định trên nhiều môi trường khác nhau như:

- Laptop cá nhân
- Máy chủ thật
- Máy ảo
- Cloud server

Ví dụ:

Một ứng dụng Flask cần Python, Flask và MySQL connector.  
Thay vì cài thủ công trên từng máy, ta đóng gói toàn bộ vào Docker image.

Sau đó chỉ cần chạy:

```bash
docker run ten-image
```
### 1.2. Docker Compose là gì?

Docker Compose là công cụ dùng để quản lý và chạy nhiều container cùng lúc bằng file cấu hình:
```
docker-compose.yml
```
Ví dụ một hệ thống có nhiều thành phần:

Flask API
MariaDB
InfluxDB
Grafana
Node-RED
Nginx

Nếu chạy từng container bằng lệnh docker run sẽ rất dài và khó quản lý.

Với Docker Compose, ta chỉ cần chạy:
```
docker compose up -d
```
Docker sẽ tự tạo và chạy toàn bộ các service đã khai báo trong file docker-compose.yml.
## 2. Các keyword thường dùng trong docker-compose.yml
### 2.1. services
Dùng để khai báo danh sách các service/container trong hệ thống.
Ví dụ:
```
services:
  api:
    build: ./api
    ports:
      - "5000:5000"
```
Ý nghĩa:

Tạo service tên là api
Build image từ thư mục ./api
Mở port 5000
### 2.2. image

Dùng để chỉ định image có sẵn.

Ví dụ:
```
mariadb:
  image: mariadb:11
```
Ý nghĩa:

Service mariadb sử dụng image mariadb:11
### 2.3. build

Dùng để build image từ Dockerfile.

Ví dụ:
```
api:
  build: ./api
```
Ý nghĩa:

Docker sẽ tìm file Dockerfile trong thư mục api
Sau đó build image cho service api
### 2.4. container_name

Dùng để đặt tên cụ thể cho container.

Ví dụ:
```
container_name: bt5_api
```
Ý nghĩa:

Container được tạo ra có tên là bt5_api
Giúp dễ quản lý khi dùng lệnh docker ps
### 2.5. ports

Dùng để ánh xạ cổng từ máy thật vào container.

Ví dụ:
```
ports:
  - "8080:80"
```
Ý nghĩa:

Máy thật port 8080 -> Container port 80

Khi truy cập:
```
http://localhost:8080
```
thì thực chất đang truy cập vào port 80 trong container.
### 2.6. environment

Dùng để khai báo biến môi trường cho container.

Ví dụ:
```
environment:
  MARIADB_ROOT_PASSWORD: root123
  MARIADB_DATABASE: monitor_db
```
Ý nghĩa:

Đặt mật khẩu root cho MariaDB
Tạo database tên `monitor_db`
### 2.7. volumes

Dùng để lưu dữ liệu bền vững hoặc mount thư mục từ máy thật vào container.

Ví dụ:
```
volumes:
  mariadb_data:

services:
  mariadb:
    volumes:
      - mariadb_data:/var/lib/mysql
```
Ý nghĩa:

Dữ liệu MariaDB được lưu trong volume mariadb_data
Khi container bị xóa, dữ liệu vẫn còn
### 2.8. networks

Dùng để tạo mạng nội bộ cho các container giao tiếp với nhau.

Ví dụ:
```
networks:
  monitor_net:

services:
  api:
    networks:
      - monitor_net

  mariadb:
    networks:
      - monitor_net
```
Ý nghĩa:

api và mariadb cùng nằm trong network monitor_net
Service api có thể gọi database bằng hostname:
```
mariadb
```
### 2.9. depends_on

Dùng để quy định thứ tự khởi động service.

Ví dụ:
```
api:
  depends_on:
    - mariadb
```
Ý nghĩa:

Service mariadb được khởi động trước api
### 2.10. restart

Dùng để cấu hình tự khởi động lại container.

Ví dụ:
```
restart: unless-stopped
```
Ý nghĩa:

Container sẽ tự chạy lại khi bị lỗi hoặc khi máy chủ restart
Trừ khi người dùng chủ động stop container
### 2.11. command

Dùng để ghi đè lệnh chạy mặc định trong container.

Ví dụ:
```
command: python app.py
```
Ý nghĩa:

Khi container chạy, nó sẽ thực thi lệnh `python app.py`
### 2.12. healthcheck

Dùng để kiểm tra container có hoạt động bình thường hay không.

Ví dụ:
```
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:5000/api/latest"]
  interval: 10s
  timeout: 5s
  retries: 5
```
Ý nghĩa:

Cứ 10 giây Docker kiểm tra API một lần
Nếu API lỗi nhiều lần, container được đánh dấu là unhealthy
## 3. Ưu điểm khi triển khai app bằng Docker

Docker có nhiều ưu điểm khi triển khai ứng dụng:

### 3.1. Đồng nhất môi trường chạy

Ứng dụng chạy được trên laptop thì khi đưa lên server cũng có thể chạy giống vậy.

Điều này giúp tránh lỗi kiểu:
```
Máy em chạy được, máy thầy không chạy được
```
### 3.2. Triển khai nhanh

Chỉ cần có:

Dockerfile
docker-compose.yml
Source code

Sau đó chạy:
```
docker compose up -d
```
### 3.3. Dễ quản lý nhiều service

Một hệ thống có nhiều thành phần như:

Web server
API
Database
Dashboard
Tool xử lý dữ liệu

có thể quản lý chung trong một file docker-compose.yml.
### 3.4. Dễ backup và restore

Có thể backup:

Docker image
Source code
Volume
Database

Sau đó chuyển sang máy khác và khôi phục lại.
### 3.5. Cô lập ứng dụng

Mỗi service chạy trong một container riêng.

Ví dụ:

Flask API chạy trong container riêng
MariaDB chạy trong container riêng
Grafana chạy trong container riêng

Nhờ vậy hạn chế xung đột thư viện và cấu hình.



<img width="1121" height="229" alt="image" src="https://github.com/user-attachments/assets/3781d4c7-712f-44e7-afd3-6ae96b7aaa3a" />


vẽ biểu đồ
<img width="1360" height="751" alt="image" src="https://github.com/user-attachments/assets/62a8318e-ca00-49ea-89a3-722a72199f0b" />
