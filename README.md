# BT5 - Docker Compose: App Monitor + Alert Data Realtime
## Họ & tên : Lăng Nguyễn Minh Lượng
## MSSV: K225480106044
## 1. Thông tin bài tập lớn

**Môn học:** Phát triển ứng dụng với mã nguồn mở - TEE0421

**Các bài trong bài tập lớn:**

- **BT1:** Ubuntu + Docker: Dùng Docker để build `myapi`
- **BT2:** Django Python: Web quản lý tiệm cầm đồ
- **BT3:** WordPress + MariaDB + phpMyAdmin
- **BT4:** WordPress + n8n + Bot Telegram + Gemini: Auto đăng bài bằng cách chat
- **BT5:** Docker Compose: App Monitor + Alert Data Realtime

---

# PHẦN I. LÝ THUYẾT

## 2. Docker là gì?

Docker là nền tảng dùng để đóng gói, triển khai và chạy ứng dụng trong các **container**.

Container là một môi trường chạy độc lập, bên trong có đầy đủ:

- Mã nguồn ứng dụng
- Thư viện cần thiết
- Runtime
- File cấu hình
- Các dependency liên quan

Nhờ Docker, ứng dụng có thể chạy ổn định trên nhiều môi trường khác nhau như:

- Laptop cá nhân
- Máy ảo Ubuntu
- Máy chủ thật
- Cloud server

Ví dụ: một ứng dụng Flask cần Python, Flask và thư viện kết nối MariaDB. Thay vì cài thủ công các thành phần này trên từng máy, ta đóng gói toàn bộ vào Docker image. Sau đó chỉ cần chạy:

```bash
docker run ten-image
```

Ứng dụng sẽ chạy với môi trường đã được đóng gói sẵn.

---

## 3. Docker Compose là gì?

Docker Compose là công cụ dùng để quản lý nhiều container cùng lúc bằng một file cấu hình:

```text
docker-compose.yml
```

Ví dụ một hệ thống có nhiều thành phần:

- Flask API
- MariaDB
- InfluxDB
- Grafana
- Node-RED
- Nginx

Nếu chạy từng container bằng `docker run` thì câu lệnh rất dài và khó quản lý.

Với Docker Compose, ta chỉ cần khai báo toàn bộ service trong file `docker-compose.yml`, sau đó chạy:

```bash
docker compose up -d
```

Docker sẽ tự tạo network, volume và chạy toàn bộ container theo cấu hình.

---

## 4. Các keyword thường dùng trong docker-compose.yml

### 4.1. `services`

Dùng để khai báo danh sách các service/container trong hệ thống.

```yaml
services:
  api:
    build: ./api
    ports:
      - "5000:5000"
```

Ý nghĩa:

- Tạo service tên là `api`
- Build image từ thư mục `./api`
- Mở port `5000`

### 4.2. `image`

Dùng để chỉ định image có sẵn từ Docker Hub hoặc image đã build local.

```yaml
mariadb:
  image: mariadb:11
```

Ý nghĩa: service `mariadb` sử dụng image `mariadb:11`.

### 4.3. `build`

Dùng để build image từ Dockerfile.

```yaml
api:
  build: ./api
```

Ý nghĩa: Docker sẽ tìm file `Dockerfile` trong thư mục `./api` và build image cho service `api`.

### 4.4. `container_name`

Dùng để đặt tên cụ thể cho container.

```yaml
container_name: bt5_api
```

Ý nghĩa: container được tạo ra có tên là `bt5_api`, giúp dễ quản lý khi dùng:

```bash
docker ps
docker logs bt5_api
```

### 4.5. `ports`

Dùng để ánh xạ cổng từ máy thật vào container.

```yaml
ports:
  - "8080:80"
```

Ý nghĩa:

```text
Port 8080 của máy thật -> Port 80 trong container
```

Khi truy cập `http://localhost:8080` thì thực chất đang truy cập vào port `80` bên trong container.

### 4.6. `environment`

Dùng để khai báo biến môi trường cho container.

```yaml
environment:
  MYSQL_ROOT_PASSWORD: root123
  MYSQL_DATABASE: monitor_db
```

Ý nghĩa:

- Đặt mật khẩu root cho MariaDB
- Tạo database tên `monitor_db`

### 4.7. `volumes`

Dùng để lưu dữ liệu bền vững hoặc mount thư mục từ máy thật vào container.

```yaml
volumes:
  mariadb_data:

services:
  mariadb:
    volumes:
      - mariadb_data:/var/lib/mysql
```

Ý nghĩa: dữ liệu MariaDB được lưu trong volume `mariadb_data`. Khi container bị xóa, dữ liệu vẫn còn.

### 4.8. `networks`

Dùng để tạo mạng nội bộ cho các container giao tiếp với nhau.

```yaml
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

Ý nghĩa: `api` và `mariadb` cùng nằm trong network `monitor_net`, service `api` có thể gọi database bằng hostname `mariadb`.

### 4.9. `depends_on`

Dùng để quy định thứ tự khởi động service.

```yaml
api:
  depends_on:
    - mariadb
```

Ý nghĩa: service `mariadb` được khởi động trước `api`.

### 4.10. `restart`

Dùng để cấu hình tự khởi động lại container.

```yaml
restart: always
```

Ý nghĩa: container sẽ tự khởi động lại nếu bị lỗi hoặc khi máy chủ restart.

### 4.11. `command`

Dùng để ghi đè lệnh chạy mặc định trong container.

```yaml
command: python app.py
```

Ý nghĩa: khi container chạy, nó sẽ thực thi lệnh `python app.py`.

### 4.12. `healthcheck`

Dùng để kiểm tra container có hoạt động bình thường hay không.

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:5000/api/health"]
  interval: 10s
  timeout: 5s
  retries: 5
```

Ý nghĩa: Docker sẽ kiểm tra API định kỳ. Nếu API lỗi nhiều lần, container được đánh dấu là `unhealthy`.

---

## 5. Ưu điểm khi triển khai ứng dụng bằng Docker

### 5.1. Đồng nhất môi trường chạy

Ứng dụng chạy được trên laptop thì khi đưa lên server cũng có thể chạy giống vậy, hạn chế lỗi:

```text
Máy em chạy được, máy thầy không chạy được
```

### 5.2. Triển khai nhanh

Chỉ cần có Dockerfile, docker-compose.yml và source code, sau đó chạy:

```bash
docker compose up -d
```

### 5.3. Dễ quản lý nhiều service

Một hệ thống có nhiều thành phần như webserver, API, database, dashboard, công cụ xử lý dữ liệu có thể được quản lý chung trong một file `docker-compose.yml`.

### 5.4. Dễ backup và restore

Có thể backup Docker image, source code, volume và database, sau đó chuyển sang máy khác và khôi phục lại.

### 5.5. Cô lập ứng dụng

Mỗi service chạy trong một container riêng như Flask API, MariaDB, Grafana, Node-RED. Nhờ vậy hạn chế xung đột thư viện và cấu hình.

---

## 6. Triển khai Docker app lên máy chủ không có Internet

Khi app đã được build và test OK trên laptop cá nhân hoặc máy ảo Ubuntu, nếu muốn triển khai lên máy chủ thật không có Internet, cần thực hiện các bước sau.

### 6.1. Trên máy đang test OK

Kiểm tra các image đang có:

```bash
docker images
```
<img width="1135" height="646" alt="image" src="https://github.com/user-attachments/assets/b5722523-2b9f-4e2d-8f4b-08a18639ffa7" />

Đóng gói image ra file `.tar`:

```bash
docker save -o monitor-alert-images.tar \
grafana/grafana:latest \
influxdb:2.7 \
mariadb:11 \
monitor-flask-api:1.0 \
nginx:latest \
nodered/node-red:latest
```

Kiểm tra file image:

```bash
ls -lh monitor-alert-images.tar
```

Kết quả thực tế:

```text
monitor-alert-images.tar   908M
```

### 6.2. Backup volume dữ liệu

Các volume quan trọng gồm:

```text
monitor-alert-app_nodered_data
monitor-alert-app_grafana_data
monitor-alert-app_influxdb_data
monitor-alert-app_mariadb_data
```

Dừng app tạm thời:

```bash
cd ~/monitor-alert-app
docker compose stop
```

Backup Node-RED:

```bash
docker run --rm \
--entrypoint tar \
-v monitor-alert-app_nodered_data:/data \
-v /home/admin1/monitor-alert-app/backups:/backup \
nodered/node-red:latest \
czf /backup/nodered_data.tar.gz -C /data .
```

Backup Grafana:

```bash
docker run --rm \
--user root \
--entrypoint tar \
-v monitor-alert-app_grafana_data:/data \
-v /home/admin1/monitor-alert-app/backups:/backup \
grafana/grafana:latest \
czf /backup/grafana_data.tar.gz -C /data .
```

Backup InfluxDB:

```bash
docker run --rm \
--entrypoint tar \
-v monitor-alert-app_influxdb_data:/data \
-v /home/admin1/monitor-alert-app/backups:/backup \
influxdb:2.7 \
czf /backup/influxdb_data.tar.gz -C /data .
```

Backup MariaDB:

```bash
docker run --rm \
--entrypoint tar \
-v monitor-alert-app_mariadb_data:/data \
-v /home/admin1/monitor-alert-app/backups:/backup \
mariadb:11 \
czf /backup/mariadb_data.tar.gz -C /data .
```

Chạy lại app:

```bash
docker compose up -d
```

Kiểm tra backup:

```bash
ls -lh ~/monitor-alert-app/backups
```

Kết quả gồm:

<img width="1118" height="640" alt="image" src="https://github.com/user-attachments/assets/5da23457-6c6e-42f0-a2a6-9c710e5b9c6f" />
### 6.3. Đóng gói source code

```bash
cd ~

tar --exclude='monitor-alert-app/monitor-alert-images.tar' \
-czvf monitor-alert-app-source.tar.gz monitor-alert-app
```

Kiểm tra:

```bash
ls -lh monitor-alert-app-source.tar.gz
ls -lh ~/monitor-alert-app/monitor-alert-images.tar
```

Kết quả thực tế:

<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/d9758ba3-744a-4ca5-93fb-a8bd9e60668b" />

### 6.4. Copy file sang máy chính hoặc server offline

Có thể copy bằng USB, WinSCP, SCP hoặc Shared Folder máy ảo.

Ví dụ copy từ Ubuntu máy ảo ra Windows bằng SCP:

```powershell
mkdir D:\DockerBackup

scp admin1@192.168.220.129:/home/admin1/monitor-alert-app/monitor-alert-images.tar D:\DockerBackup\
scp admin1@192.168.220.129:/home/admin1/monitor-alert-app-source.tar.gz D:\DockerBackup\
```

Kiểm tra trên Windows:

```powershell
dir D:\DockerBackup
```

Kết quả:

```text
monitor-alert-app-source.tar.gz
monitor-alert-images.tar
```

<img width="1177" height="679" alt="image" src="https://github.com/user-attachments/assets/77fb201f-d8db-48ed-b246-b5c3fdf479ff" />


<img width="1366" height="767" alt="image" src="https://github.com/user-attachments/assets/3a2a445f-85ab-4531-b726-497f49bfd05f" />


### 6.5. Trên server offline

Copy 2 file vào server:

```text
monitor-alert-images.tar
monitor-alert-app-source.tar.gz
```

Giải nén source:

```bash
cd ~
tar -xzvf monitor-alert-app-source.tar.gz
cd monitor-alert-app
```

Load image:

```bash
docker load -i ~/monitor-alert-images.tar
```

Chạy app:

```bash
docker compose up -d
docker ps
```

Nếu cần restore volume đã backup, dừng app:

```bash
docker compose stop
```

Restore Node-RED:

```bash
docker run --rm --entrypoint tar \
-v monitor-alert-app_nodered_data:/data \
-v /home/admin1/monitor-alert-app/backups:/backup \
nodered/node-red:latest \
xzf /backup/nodered_data.tar.gz -C /data
```

Restore Grafana:

```bash
docker run --rm --user root --entrypoint tar \
-v monitor-alert-app_grafana_data:/data \
-v /home/admin1/monitor-alert-app/backups:/backup \
grafana/grafana:latest \
xzf /backup/grafana_data.tar.gz -C /data
```

Restore InfluxDB:

```bash
docker run --rm --entrypoint tar \
-v monitor-alert-app_influxdb_data:/data \
-v /home/admin1/monitor-alert-app/backups:/backup \
influxdb:2.7 \
xzf /backup/influxdb_data.tar.gz -C /data
```

Restore MariaDB:

```bash
docker run --rm --entrypoint tar \
-v monitor-alert-app_mariadb_data:/data \
-v /home/admin1/monitor-alert-app/backups:/backup \
mariadb:11 \
xzf /backup/mariadb_data.tar.gz -C /data
```

Chạy lại app:

```bash
docker compose up -d
```

---


<img width="1121" height="229" alt="image" src="https://github.com/user-attachments/assets/3781d4c7-712f-44e7-afd3-6ae96b7aaa3a" />


vẽ biểu đồ
<img width="1360" height="751" alt="image" src="https://github.com/user-attachments/assets/62a8318e-ca00-49ea-89a3-722a72199f0b" />




<img width="1366" height="753" alt="image" src="https://github.com/user-attachments/assets/3b3fd770-e6e4-4d96-ad0a-1debd7ed54e8" />


chuyển file từ máy ảo ra máy thật
<img width="1118" height="634" alt="image" src="https://github.com/user-attachments/assets/2038897e-9d00-47c1-b898-6b8721ee1dce" />
