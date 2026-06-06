# BT5 - Docker Compose: App Monitor + Alert Data Realtime
## Họ & tên : Lăng Nguyễn Minh Lượng
## MSSV: K225480106044
## 1. Thông tin bài tập lớn

**Môn học:** Phát triển ứng dụng với mã nguồn mở - TEE0421

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
# PHẦN II. THỰC HÀNH APP MONITOR + ALERT DATA REALTIME

## 7. Mô tả ứng dụng

Ứng dụng gồm các service:

| Service | Vai trò |
|---|---|
| Node-RED | Lấy dữ liệu realtime, xử lý cảnh báo |
| MariaDB | Lưu giá trị tức thời |
| InfluxDB | Lưu dữ liệu lịch sử |
| Grafana | Vẽ biểu đồ lịch sử |
| Flask API | Đọc dữ liệu tức thời từ MariaDB |
| Nginx | Webserver chạy frontend |
| Telegram Bot | Gửi alert vào group |

Nguồn dữ liệu thực tế sử dụng:

```text
Open-Meteo API
```

Dữ liệu lấy về là nhiệt độ thực tế khu vực Thái Nguyên.

---
## 8. Cấu trúc thư mục project

```text
monitor-alert-app/
├── docker-compose.yml
├── .env
├── flask-api/
│   ├── app.py
│   ├── requirements.txt
│   └── Dockerfile
├── nginx/
│   ├── default.conf
│   └── html/
│       ├── index.html
│       ├── style.css
│       └── app.js
├── mariadb/
│   └── init.sql
├── nodered/
│   └── data/
└── backups/
```

---
## 9. Tạo project

```bash
cd ~
mkdir monitor-alert-app
cd monitor-alert-app

mkdir -p flask-api nginx/html mariadb nodered/data backups
```

---

## 10. File `.env`

Tạo file:

```bash
nano .env
```

Nội dung:

```env
MYSQL_ROOT_PASSWORD=root123
MYSQL_DATABASE=monitor_db
MYSQL_USER=monitor_user
MYSQL_PASSWORD=monitor_pass

INFLUXDB_ADMIN_USER=admin
INFLUXDB_ADMIN_PASSWORD=admin123456
INFLUXDB_ORG=my-org
INFLUXDB_BUCKET=monitor_bucket
INFLUXDB_TOKEN=my-super-token
```
## 12. File khởi tạo MariaDB

Tạo file:

```bash
nano mariadb/init.sql
```

Nội dung:

```sql
CREATE TABLE IF NOT EXISTS realtime_data (
    id INT AUTO_INCREMENT PRIMARY KEY,
    source_name VARCHAR(100),
    metric_name VARCHAR(100),
    metric_value DOUBLE,
    status VARCHAR(50),
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

INSERT INTO realtime_data (
    source_name,
    metric_name,
    metric_value,
    status
)
VALUES (
    'simulator',
    'temperature',
    0,
    'INIT'
);
```

---

## 13. Flask API

### 13.1. File `requirements.txt`

```bash
nano flask-api/requirements.txt
```

Nội dung:

```txt
flask
flask-cors
pymysql
```

### 13.2. File `Dockerfile`

```bash
nano flask-api/Dockerfile
```

Nội dung:

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

### 13.3. File `app.py`

```bash
nano flask-api/app.py
```

Nội dung:

```python
from flask import Flask, jsonify
from flask_cors import CORS
import pymysql
import os

app = Flask(__name__)
CORS(app)

def get_connection():
    return pymysql.connect(
        host=os.getenv("MYSQL_HOST", "mariadb"),
        user=os.getenv("MYSQL_USER", "monitor_user"),
        password=os.getenv("MYSQL_PASSWORD", "monitor_pass"),
        database=os.getenv("MYSQL_DATABASE", "monitor_db"),
        cursorclass=pymysql.cursors.DictCursor
    )

@app.route("/api/realtime")
def get_realtime():
    conn = get_connection()
    try:
        with conn.cursor() as cursor:
            cursor.execute("""
                SELECT source_name, metric_name, metric_value, status, updated_at
                FROM realtime_data
                ORDER BY updated_at DESC
                LIMIT 1
            """)
            row = cursor.fetchone()

        if row is None:
            return jsonify({"message": "No data"}), 404

        row["updated_at"] = str(row["updated_at"])
        return jsonify(row)

    finally:
        conn.close()

@app.route("/api/health")
def health():
    return jsonify({"status": "API OK"})

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

API chính:

```text
/api/realtime
```

---

## 14. Nginx Web Frontend

### 14.1. File `nginx/default.conf`

```bash
nano nginx/default.conf
```

Nội dung:

```nginx
server {
    listen 80;

    server_name localhost;

    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api/ {
        proxy_pass http://flask-api:5000/api/;
    }
}
```

### 14.2. File `index.html`

```bash
nano nginx/html/index.html
```

Nội dung:

```html
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <title>APP MONITOR + ALERT DATA REALTIME</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <h1>APP MONITOR + ALERT DATA REALTIME</h1>

    <div class="card realtime-card">
        <h2>Dữ liệu tức thời từ MariaDB</h2>

        <p><b>Nguồn:</b> <span id="source">...</span></p>
        <p><b>Thông số:</b> <span id="metric">...</span></p>
        <p><b>Giá trị:</b> <span id="value">...</span></p>
        <p><b>Trạng thái:</b> <span id="status">...</span></p>
        <p><b>Cập nhật:</b> <span id="time">...</span></p>
    </div>

    <div class="card grafana-card">
        <h2>Biểu đồ lịch sử từ Grafana</h2>

        <iframe
            src="http://192.168.220.129:3001/d-solo/adsxct2/monitor-alert-dashboard?orgId=1&from=now-15m&to=now&timezone=browser&refresh=5s&panelId=1&theme=dark"
            width="100%"
            height="750"
            frameborder="0">
        </iframe>
    </div>

    <script src="app.js"></script>
</body>
</html>
```

### 14.3. File `style.css`

```bash
nano nginx/html/style.css
```

Nội dung:

```css
body {
    font-family: Arial, sans-serif;
    background: #f2f4f8;
    margin: 0;
    padding: 25px;
}

h1 {
    text-align: center;
    color: #222;
    margin-bottom: 25px;
}

.card {
    background: white;
    padding: 20px;
    margin: 20px auto;
    border-radius: 10px;
    max-width: 1300px;
    box-shadow: 0 0 10px #ccc;
}

.card h2 {
    margin-top: 0;
    color: #111;
}

.realtime-card {
    max-width: 900px;
}

.grafana-card {
    max-width: 1300px;
}

p {
    font-size: 16px;
}

#status {
    font-weight: bold;
}

iframe {
    width: 100%;
    height: 750px;
    border: none;
    border-radius: 8px;
}
```
### 14.4. File `app.js`

```bash
nano nginx/html/app.js
```

Nội dung:

```javascript
async function loadRealtimeData() {
    try {
        const response = await fetch("/api/realtime");
        const data = await response.json();

        document.getElementById("source").innerText = data.source_name;
        document.getElementById("metric").innerText = data.metric_name;
        document.getElementById("value").innerText = data.metric_value;
        document.getElementById("status").innerText = data.status;
        document.getElementById("time").innerText = data.updated_at;

        const statusElement = document.getElementById("status");

        if (data.status === "OK") {
            statusElement.style.color = "green";
        } else if (data.status === "ALERT_LOW") {
            statusElement.style.color = "orange";
        } else if (data.status === "ALERT_HIGH") {
            statusElement.style.color = "red";
        } else {
            statusElement.style.color = "black";
        }

    } catch (error) {
        console.error("Lỗi lấy dữ liệu:", error);
    }
}

loadRealtimeData();
setInterval(loadRealtimeData, 5000);
```

---

## 15. File `docker-compose.yml`

```bash
nano docker-compose.yml
```

Nội dung:

```yaml
services:
  mariadb:
    image: mariadb:11
    container_name: monitor_mariadb
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - mariadb_data:/var/lib/mysql
      - ./mariadb/init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - monitor_net

  influxdb:
    image: influxdb:2.7
    container_name: monitor_influxdb
    restart: always
    ports:
      - "8086:8086"
    environment:
      DOCKER_INFLUXDB_INIT_MODE: setup
      DOCKER_INFLUXDB_INIT_USERNAME: ${INFLUXDB_ADMIN_USER}
      DOCKER_INFLUXDB_INIT_PASSWORD: ${INFLUXDB_ADMIN_PASSWORD}
      DOCKER_INFLUXDB_INIT_ORG: ${INFLUXDB_ORG}
      DOCKER_INFLUXDB_INIT_BUCKET: ${INFLUXDB_BUCKET}
      DOCKER_INFLUXDB_INIT_ADMIN_TOKEN: ${INFLUXDB_TOKEN}
    volumes:
      - influxdb_data:/var/lib/influxdb2
    networks:
      - monitor_net

  nodered:
    image: nodered/node-red:latest
    container_name: monitor_nodered
    restart: always
    ports:
      - "1881:1880"
    environment:
      TZ: Asia/Ho_Chi_Minh
    volumes:
      - nodered_data:/data
    depends_on:
      - mariadb
      - influxdb
    networks:
      - monitor_net

  flask-api:
    build: ./flask-api
    image: monitor-flask-api:1.0
    container_name: monitor_flask_api
    restart: always
    environment:
      MYSQL_HOST: mariadb
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    depends_on:
      - mariadb
    networks:
      - monitor_net

  grafana:
    image: grafana/grafana:latest
    container_name: monitor_grafana
    restart: always
    ports:
      - "3001:3000"
    environment:
      GF_SERVER_ROOT_URL: "http://192.168.220.129:3001/"
      GF_SECURITY_ALLOW_EMBEDDING: "true"
      GF_AUTH_ANONYMOUS_ENABLED: "true"
      GF_AUTH_ANONYMOUS_ORG_ROLE: "Viewer"
      GF_FEATURE_TOGGLES_ENABLE: "publicDashboards"
    volumes:
      - grafana_data:/var/lib/grafana
    depends_on:
      - influxdb
    networks:
      - monitor_net

  nginx:
    image: nginx:latest
    container_name: monitor_nginx
    restart: always
    ports:
      - "8088:80"
    volumes:
      - ./nginx/html:/usr/share/nginx/html
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - flask-api
      - grafana
    networks:
      - monitor_net

volumes:
  mariadb_data:
  influxdb_data:
  grafana_data:
  nodered_data:

networks:
  monitor_net:
    driver: bridge
```

---

## 16. Chạy ứng dụng

Kiểm tra compose:

```bash
docker compose config
```

Build và chạy:

```bash
docker compose up -d --build
```

Kiểm tra container:

```bash
docker ps
```

Hoặc:

```bash
docker ps | grep monitor
```

Kết quả gồm các container:

```text
monitor_mariadb
monitor_influxdb
monitor_nodered
monitor_flask_api
monitor_grafana
monitor_nginx
```

**Chỗ dán ảnh docker ps:**

```markdown
![Danh sách container đang chạy](images/docker-ps-monitor.png)
```

---

## 17. Truy cập các service

Trên trình duyệt:

```text
Web chính:  http://192.168.220.129:8088
Node-RED:   http://192.168.220.129:1881
Grafana:    http://192.168.220.129:3001
InfluxDB:   http://192.168.220.129:8086
```

Kiểm tra Flask API:

```bash
curl http://localhost:8088/api/health
curl http://localhost:8088/api/realtime
```

Kết quả API thực tế:

```json
{
  "metric_name": "temperature_2m",
  "metric_value": 28.7,
  "source_name": "open_meteo_thai_nguyen",
  "status": "OK",
  "updated_at": "2026-06-06 18:13:26"
}
```

<img width="1121" height="635" alt="image" src="https://github.com/user-attachments/assets/73162f34-5fdb-4169-9a68-97fc0b42c49e" />

---

# PHẦN III. CẤU HÌNH NODE-RED

## 18. Luồng xử lý trong Node-RED

Flow Node-RED gồm các nhánh chính:

```text
timestamp
   ↓
Get real weather
   ↓
Process Real Weather
   ├── monitor_db → debug
   ├── Format for InfluxDB → influxdb
   └── Check Alert → Build Telegram HTTP → http request → debug
```

<img width="1072" height="544" alt="image" src="https://github.com/user-attachments/assets/ba14a5a2-b852-4630-80d8-7b2a6833b478" />


---

## 19. Node `Get real weather`

Node này là `http request`, dùng để lấy dữ liệu thời tiết thật từ Open-Meteo.

Cấu hình:

```text
Method: GET
URL: https://api.open-meteo.com/v1/forecast?latitude=21.59&longitude=105.84&current=temperature_2m&timezone=Asia%2FBangkok
Return: a parsed JSON object
```

Ý nghĩa:

- Lấy nhiệt độ hiện tại tại khu vực Thái Nguyên
- Trường dữ liệu trả về là `current.temperature_2m`

---

## 20. Node `Process Real Weather`

Node này xử lý dữ liệu thời tiết, phân loại trạng thái và tạo câu SQL ghi MariaDB.

```javascript
let value = msg.payload.current.temperature_2m;

value = Number(value.toFixed(2));

let A = 20;
let B = 35;

let status = "OK";

if (value < A) {
    status = "ALERT_LOW";
} else if (value > B) {
    status = "ALERT_HIGH";
}

msg.topic = `
UPDATE realtime_data
SET
    source_name = 'open_meteo_thai_nguyen',
    metric_name = 'temperature_2m',
    metric_value = ${value},
    status = '${status}',
    updated_at = NOW()
WHERE id = 1;
`;

msg.payload = {
    source_name: "open_meteo_thai_nguyen",
    metric_name: "temperature_2m",
    metric_value: value,
    status: status,
    min_ok: A,
    max_ok: B,
    time: new Date().toISOString()
};

return msg;
```

Logic cảnh báo:

```text
value < 20         -> ALERT_LOW
20 <= value <= 35  -> OK
value > 35         -> ALERT_HIGH
```

---

## 21. Ghi dữ liệu tức thời vào MariaDB

Node MySQL/MariaDB cấu hình:

```text
Host: mariadb
Port: 3306
User: monitor_user
Password: monitor_pass
Database: monitor_db
```

Lưu ý: không dùng `localhost`. Vì MariaDB chạy trong Docker Compose nên hostname là tên service `mariadb`.

---

## 22. Node `Format for InfluxDB`

Node này chuyển dữ liệu sang định dạng để ghi vào InfluxDB.

```javascript
msg.measurement = "temperature_monitor";

msg.payload = [
    {
        temperature: msg.payload.metric_value,
        status_code: msg.payload.status === "OK" ? 0 : 1
    },
    {
        source: msg.payload.source_name,
        status: msg.payload.status
    }
];

return msg;
```

---

## 23. Cấu hình InfluxDB trong Node-RED

Node `influxdb out` cấu hình:

```text
Version: 2.0
URL: http://influxdb:8086
Token: my-super-token
Organization: my-org
Bucket: monitor_bucket
Measurement: temperature_monitor
```

---

## 24. Node `Check Alert`

Node này chống gửi Telegram khi dữ liệu đang OK và chống spam cảnh báo liên tục.

```javascript
let status = msg.payload.status;
let value = msg.payload.metric_value;

if (status === "OK") {
    flow.set("lastAlertStatus", "OK");
    return null;
}

let lastAlertStatus = flow.get("lastAlertStatus") || "OK";
let lastAlertTime = flow.get("lastAlertTime") || 0;

let now = Date.now();
let cooldown = 60 * 1000;

if (status !== lastAlertStatus || now - lastAlertTime > cooldown) {
    flow.set("lastAlertStatus", status);
    flow.set("lastAlertTime", now);
    return msg;
}

return null;
```

---

## 25. Node `Build Telegram HTTP`

Node này tạo request gửi tin nhắn Telegram.

```javascript
const token = "TOKEN_BOT_TELEGRAM";
const chatId = "CHAT_ID_GROUP";

let icon = "⚠️";

if (msg.payload.status === "ALERT_HIGH") {
    icon = "🚨";
} else if (msg.payload.status === "ALERT_LOW") {
    icon = "⚠️";
}

let value = msg.payload.metric_value;
let source = msg.payload.source_name;
let metric = msg.payload.metric_name;
let status = msg.payload.status;
let minOk = msg.payload.min_ok;
let maxOk = msg.payload.max_ok;

let time = new Date().toLocaleString("vi-VN", {
    timeZone: "Asia/Ho_Chi_Minh"
});

let text =
`${icon} ${status}

Nguồn: ${source}
Thông số: ${metric}
Giá trị hiện tại: ${value} °C
Ngưỡng hợp lệ: ${minOk} °C - ${maxOk} °C
Trạng thái: ${status}
Thời gian: ${time}`;

msg.method = "POST";
msg.url = `https://api.telegram.org/bot${token}/sendMessage`;

msg.headers = {
    "Content-Type": "application/json"
};

msg.payload = {
    chat_id: chatId,
    text: text
};

return msg;
```

---

## 26. Test Telegram Bot

Kiểm tra bot:

```bash
curl "https://api.telegram.org/bot<TOKEN>/getMe"
```

Kết quả đúng:

```json
{"ok":true}
```

Lấy chat ID:

```bash
curl "https://api.telegram.org/bot<TOKEN>/getUpdates"
```

Gửi thử tin nhắn:

```bash
curl -X POST "https://api.telegram.org/bot<TOKEN>/sendMessage" \
-d "chat_id=<CHAT_ID_GROUP>" \
--data-urlencode "text=Test alert từ Ubuntu server"
```

<img width="1260" height="2800" alt="image" src="https://github.com/user-attachments/assets/31432e1e-8df9-4fc0-92ff-c6b28b7b473e" />

---

# PHẦN IV. CẤU HÌNH INFLUXDB VÀ GRAFANA

## 27. Kiểm tra InfluxDB

Truy cập:

```text
http://192.168.220.129:8086
```

Đăng nhập:

```text
Username: admin
Password: admin123456
```

Bucket:

```text
monitor_bucket
```

Measurement:

```text
temperature_monitor
```
<img width="1360" height="751" alt="image" src="https://github.com/user-attachments/assets/62a8318e-ca00-49ea-89a3-722a72199f0b" />

---

## 28. Cấu hình Grafana Data Source

Truy cập Grafana:

```text
http://192.168.220.129:3001
```

Đăng nhập mặc định:

```text
Username: admin
Password: admin
```

Thêm Data Source:

```text
Connections -> Data sources -> Add data source -> InfluxDB
```

Cấu hình:

```text
Query language: Flux
URL: http://influxdb:8086
Organization: my-org
Token: my-super-token
Default Bucket: monitor_bucket
```

Bấm:

```text
Save & test
```

Kết quả mong muốn:

```text
datasource is working
```

<img width="1342" height="670" alt="image" src="https://github.com/user-attachments/assets/87957c88-5ebd-4d70-b8c9-563e0cabb65e" />

---

## 29. Query Grafana vẽ biểu đồ

Query Flux dùng để vẽ nhiệt độ từ Open-Meteo:

```flux
from(bucket: "monitor_bucket")
  |> range(start: -30m)
  |> filter(fn: (r) => r["_measurement"] == "temperature_monitor")
  |> filter(fn: (r) => r["_field"] == "temperature")
  |> filter(fn: (r) => r["source"] == "open_meteo_thai_nguyen")
  |> group(columns: ["_field"])
  |> aggregateWindow(every: 30s, fn: mean, createEmpty: false)
  |> yield(name: "temperature")
```


<img width="1366" height="753" alt="image" src="https://github.com/user-attachments/assets/3b3fd770-e6e4-4d96-ad0a-1debd7ed54e8" />

---

## 30. Nhúng Grafana vào Web bằng iframe

Ví dụ iframe:

```html
<iframe
    src="http://192.168.220.129:3001/d-solo/adsxct2/monitor-alert-dashboard?orgId=1&from=now-15m&to=now&timezone=browser&refresh=5s&panelId=1&theme=dark"
    width="100%"
    height="750"
    frameborder="0">
</iframe>
```

Để Grafana cho phép nhúng iframe, trong `docker-compose.yml` đã thêm:

```yaml
environment:
  GF_SERVER_ROOT_URL: "http://192.168.220.129:3001/"
  GF_SECURITY_ALLOW_EMBEDDING: "true"
  GF_AUTH_ANONYMOUS_ENABLED: "true"
  GF_AUTH_ANONYMOUS_ORG_ROLE: "Viewer"
  GF_FEATURE_TOGGLES_ENABLE: "publicDashboards"
```
Ảnh chạy dữ liệu thật 
<img width="1313" height="631" alt="image" src="https://github.com/user-attachments/assets/30a7811f-1357-4f50-a076-6b6f54c25792" />
Do bài đo nhiệt độ biến động nhiệt mất quá nhiều thời gian nên biểu đồ không thấy rõ

<img width="1366" height="767" alt="image" src="https://github.com/user-attachments/assets/91de5b18-c353-4298-9ef3-ec29249905f5" />

---

# PHẦN V. EXPORT, XÓA VÀ RESTORE CONTAINER

## 31. Xuất Docker image ra file nén

```bash
cd ~/monitor-alert-app

docker save -o monitor-alert-images.tar \
grafana/grafana:latest \
influxdb:2.7 \
mariadb:11 \
monitor-flask-api:1.0 \
nginx:latest \
nodered/node-red:latest
```

Kiểm tra:

```bash
ls -lh monitor-alert-images.tar
```

<img width="1121" height="629" alt="image" src="https://github.com/user-attachments/assets/cde2bf04-40d3-40cd-87c3-db531e854e82" />

---

## 32. Đóng gói source code và backup volume

```bash
cd ~
rm -f monitor-alert-app-source.tar.gz

tar --exclude='monitor-alert-app/monitor-alert-images.tar' \
-czvf monitor-alert-app-source.tar.gz monitor-alert-app
```

Kiểm tra:

```bash
ls -lh monitor-alert-app-source.tar.gz
ls -lh ~/monitor-alert-app/monitor-alert-images.tar
```

---

## 33. Copy file từ máy ảo Ubuntu ra máy chính Windows

Trên Windows PowerShell:

```powershell
mkdir D:\DockerBackup
```

Copy file:

```powershell
scp admin1@192.168.220.129:/home/admin1/monitor-alert-app/monitor-alert-images.tar D:\DockerBackup\
scp admin1@192.168.220.129:/home/admin1/monitor-alert-app-source.tar.gz D:\DockerBackup\
```

Kiểm tra:

```powershell
dir D:\DockerBackup
```

Kết quả:

```text
monitor-alert-app-source.tar.gz
monitor-alert-images.tar
```

<img width="1118" height="633" alt="image" src="https://github.com/user-attachments/assets/21c2da4f-6db8-4da5-a2e5-93d44b3a93b7" />

---

## 34. Xóa mọi container đang chạy

Kiểm tra container:

```bash
docker ps
```

Dừng toàn bộ container đang chạy:

```bash
docker stop $(docker ps -q)
```

Xóa toàn bộ container:

```bash
docker rm $(docker ps -aq)
```

Kiểm tra lại:

```bash
docker ps -a
```

Nếu không còn container nào thì xóa thành công.

<img width="1118" height="630" alt="image" src="https://github.com/user-attachments/assets/da1e507b-7f4c-4ab1-a8e4-1607d925d9d1" />


---

## 35. Load lại image từ file nén

Nếu image đã bị xóa hoặc triển khai sang máy mới, load image bằng:

```bash
docker load -i monitor-alert-images.tar
```

Kiểm tra image:

```bash
docker images
```

Phải có:

```text
grafana/grafana:latest
influxdb:2.7
mariadb:11
monitor-flask-api:1.0
nginx:latest
nodered/node-red:latest
```

<img width="1123" height="638" alt="image" src="https://github.com/user-attachments/assets/0c6afeb7-d293-4de7-9fd2-204a8a3751a9" />

---

## 36. Khôi phục container bằng Docker Compose

Giải nén source:

```bash
cd ~
tar -xzvf monitor-alert-app-source.tar.gz
cd monitor-alert-app
```

Chạy container:

```bash
docker compose up -d
```

Nếu cần khôi phục đúng dữ liệu backup, dừng app:

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
docker ps
```

---

## 37. Kiểm tra sau khi restore

Kiểm tra API:

```bash
curl http://localhost:8088/api/realtime
```

Mở web:

```text
http://192.168.220.129:8088
```

Mở Node-RED:

```text
http://192.168.220.129:1881
```

Mở Grafana:

```text
http://192.168.220.129:3001
```


---

# PHẦN VI. KẾT QUẢ ĐẠT ĐƯỢC

## 38. Kết quả thực hiện

Ứng dụng đã hoàn thành các chức năng:

- Dùng Docker Compose quản lý nhiều service
- Node-RED lấy dữ liệu nhiệt độ thực tế từ Open-Meteo
- MariaDB lưu dữ liệu tức thời
- InfluxDB lưu dữ liệu lịch sử
- Flask API đọc dữ liệu tức thời từ MariaDB
- Nginx chạy web frontend HTML/CSS/JS
- Web tự động cập nhật dữ liệu mới
- Grafana trực quan hóa dữ liệu lịch sử
- Iframe nhúng Grafana vào web
- Telegram Bot gửi cảnh báo khi dữ liệu bất thường
- Có cơ chế chống spam Telegram
- Đóng gói image bằng `docker save`
- Backup volume
- Copy file ra máy chính
- Có thể restore app bằng `docker load` và `docker compose up -d`

---

## 39. Các địa chỉ truy cập

```text
Web chính:  http://192.168.220.129:8088
Node-RED:   http://192.168.220.129:1881
Grafana:    http://192.168.220.129:3001
InfluxDB:   http://192.168.220.129:8086
```

---


## 40. Kết luận

BT5 đã xây dựng thành công ứng dụng **APP MONITOR + ALERT DATA REALTIME** bằng Docker Compose với nhiều service gồm Node-RED, MariaDB, InfluxDB, Grafana, Flask API và Nginx.

Node-RED lấy dữ liệu nhiệt độ thực tế từ Open-Meteo, xử lý ngưỡng cảnh báo, lưu dữ liệu tức thời vào MariaDB và lưu lịch sử vào InfluxDB. Flask API đọc dữ liệu từ MariaDB để cung cấp cho web frontend. Nginx đóng vai trò webserver chạy giao diện HTML/CSS/JS. Grafana trực quan hóa dữ liệu lịch sử từ InfluxDB và được nhúng vào web bằng iframe. Khi giá trị vượt ngưỡng, Node-RED gửi cảnh báo Telegram vào group với nội dung rõ ràng, có giá trị gây cảnh báo, trạng thái và thời gian.

Toàn bộ Docker image, source code, cấu hình và dữ liệu volume đã được backup để triển khai lại trên máy chủ không có Internet bằng `docker load` và `docker compose up -d`.

---

