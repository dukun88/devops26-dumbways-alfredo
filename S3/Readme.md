# Wayshub Deployment, Docker, Jenkins & GitLab CI/CD

Dokumentasi pengerjaan deployment **Wayshub Backend & Frontend** menggunakan Docker, Docker Compose, Nginx Reverse Proxy, SSL Wildcard, Jenkins CI/CD, dan GitLab Runner.

Repository yang digunakan:

* [Wayshub Backend](https://github.com/dumbwaysdev/wayshub-backend.git)
* [Wayshub Frontend](https://github.com/dumbwaysdev/wayshub-frontend.git)
* [Certbot](https://certbot.eff.org/instructions?ws=nginx&os=ubuntufocal)
* [PM2 Runtime with Docker](https://pm2.keymetrics.io/docs/usage/docker-pm2-nodejs)

> **Note:** Ganti seluruh placeholder seperti `team1`, IP server, username Docker Hub, domain, dan credential dengan konfigurasi team masing-masing.

---

# 1. Architecture

Task ini menggunakan 2 environment:

* **Staging**
* **Production**

Staging menggunakan satu server Appserver dengan beberapa container.

Production menggunakan server terpisah untuk setiap komponen.

## 1.1 Staging Architecture

```text
                         INTERNET
                             |
                             v
                    +----------------+
                    |     NGINX      |
                    | Reverse Proxy  |
                    |      SSL       |
                    +-------+--------+
                            |
              +-------------+-------------+
              |                           |
              v                           v
       +-------------+              +-------------+
       |  Frontend   |              |   Backend   |
       |  Container  |              |  Container  |
       +-------------+              +------+------+
                                          |
                                          v
                                   +-------------+
                                   |    MySQL    |
                                   |  Container  |
                                   |   Volume    |
                                   +-------------+
```

Seluruh service menggunakan custom Docker network:

```text
team1
```

---

# 2. Production Architecture

Production menggunakan server terpisah.

```text
                         INTERNET
                             |
                             v
                    +----------------+
                    | NGINX SERVER   |
                    | Reverse Proxy  |
                    | SSL Wildcard   |
                    +-------+--------+
                            |
                +-----------+-----------+
                |                       |
                v                       v
       +----------------+      +----------------+
       | FRONTEND SERVER|      | BACKEND SERVER |
       |                |      |                |
       | Container #1   |      | Container #1   |
       | Container #2   |      | Container #2   |
       +----------------+      +-------+--------+
                                        |
                                        v
                               +----------------+
                               | DATABASE SERVER|
                               |     MySQL      |
                               |     Volume     |
                               +----------------+
```

---

# 3. Server Requirements

Contoh server yang digunakan:

| Server          | Environment | Service                         |
| --------------- | ----------- | ------------------------------- |
| App Server      | Staging     | Nginx, Frontend, Backend, MySQL |
| Database Server | Production  | MySQL                           |
| Backend Server  | Production  | 2 Backend Containers            |
| Frontend Server | Production  | 2 Frontend Containers           |
| Web Server      | Production  | Nginx Reverse Proxy             |
| Jenkins Server  | CI/CD       | Jenkins                         |
| GitLab Runner   | CI/CD       | GitLab Runner                   |

Contoh IP:

```text
App Server       : 10.10.10.10
Database Server  : 10.10.10.11
Backend Server   : 10.10.10.12
Frontend Server  : 10.10.10.13
Web Server       : 10.10.10.14
Jenkins Server   : 10.10.10.15
```

> Sesuaikan IP dengan VM yang tersedia.

---

# 4. Create Team User

Gunakan Appserver yang telah disediakan.

Login:

```bash
ssh root@<APP_SERVER_IP>
```

Buat user berdasarkan nama team.

Contoh:

```bash
adduser team1
```

Tambahkan sudo:

```bash
usermod -aG sudo team1
```

Login menggunakan user:

```bash
ssh team1@<APP_SERVER_IP>
```

Cek:

```bash
whoami
```

Expected:

```text
team1
```

---

# 5. Docker Installation Script

Buat directory:

```bash
mkdir -p ~/docker-install
cd ~/docker-install
```

Buat script:

```bash
nano install-docker.sh
```

Isi:

```bash
#!/bin/bash

set -e

echo "====================================="
echo " Docker Installation Script"
echo "====================================="

echo "[1/5] Updating package repository..."
sudo apt update

echo "[2/5] Installing prerequisites..."
sudo apt install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

echo "[3/5] Adding Docker GPG key..."
sudo install -m 0755 -d /etc/apt/keyrings

sudo curl -fsSL \
    https://download.docker.com/linux/ubuntu/gpg \
    -o /etc/apt/keyrings/docker.asc

sudo chmod a+r /etc/apt/keyrings/docker.asc

echo "[4/5] Adding Docker repository..."

echo \
  "deb [arch=$(dpkg --print-architecture) \
  signed-by=/etc/apt/keyrings/docker.asc] \
  https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update

echo "[5/5] Installing Docker..."

sudo apt install -y \
    docker-ce \
    docker-ce-cli \
    containerd.io \
    docker-buildx-plugin \
    docker-compose-plugin

sudo systemctl enable docker
sudo systemctl start docker

sudo usermod -aG docker "$USER"

echo "====================================="
echo " Docker installation completed!"
echo " Logout and login again."
echo "====================================="
```

Beri permission:

```bash
chmod +x install-docker.sh
```

Jalankan:

```bash
./install-docker.sh
```

Logout:

```bash
exit
```

Login kembali:

```bash
ssh team1@<APP_SERVER_IP>
```

Test Docker:

```bash
docker --version
```

Test Compose:

```bash
docker compose version
```

Test:

```bash
docker run hello-world
```

---

# 6. Repository Structure

Struktur repository deployment:

```text
wayshub-deployment/
├── backend/
│   ├── Dockerfile
│   └── ...
│
├── frontend/
│   ├── Dockerfile
│   └── ...
│
├── nginx/
│   ├── nginx.conf
│   └── ssl/
│
├── staging/
│   ├── docker-compose.yml
│   └── .env
│
├── production/
│   ├── database/
│   │   └── docker-compose.yml
│   │
│   ├── backend/
│   │   └── docker-compose.yml
│   │
│   ├── frontend/
│   │   └── docker-compose.yml
│   │
│   └── nginx/
│       └── docker-compose.yml
│
└── README.md
```

---

# 7. Wayshub Backend

Clone repository:

```bash
git clone https://github.com/dumbwaysdev/wayshub-backend.git
```

Masuk:

```bash
cd wayshub-backend
```

Install dependency:

```bash
npm install
```

---

# 8. Backend Configuration

Edit konfigurasi:

```bash
nano config/config.json
```

Contoh:

```json
{
  "development": {
    "username": "demo_user",
    "password": "DemoPassword123!",
    "database": "demo",
    "host": "mysql",
    "dialect": "mysql"
  }
}
```

Untuk Docker Compose, hostname database menggunakan nama service.

Contoh:

```text
mysql
```

bukan:

```text
localhost
```

Karena backend dan database berjalan pada container yang berbeda.

---

# 9. Backend Dockerfile

Buat:

```text
backend/Dockerfile
```

Contoh multistage build:

```dockerfile
FROM node:14-alpine AS builder

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

FROM node:14-alpine

WORKDIR /app

ENV NODE_ENV=production

COPY --from=builder /app .

EXPOSE 5000

CMD ["npm", "start"]
```

Build image:

```bash
docker build \
    -t team1/dumbflix/backend:staging \
    ./backend
```

---

# 10. Wayshub Frontend

Clone:

```bash
git clone https://github.com/dumbwaysdev/wayshub-frontend.git
```

Masuk:

```bash
cd wayshub-frontend
```

Install:

```bash
npm install
```

---

# 11. Frontend Configuration

Edit:

```bash
nano src/config/api.js
```

Sesuaikan dengan backend.

Staging:

```javascript
const API = "https://api.team1.staging.studentdumbways.my.id";
```

Production:

```javascript
const API = "https://api.team1.studentdumbways.my.id";
```

> Pastikan URL disesuaikan dengan domain team.

---

# 12. Frontend Multistage Dockerfile

Buat:

```text
frontend/Dockerfile
```

Contoh:

```dockerfile
FROM node:14-alpine AS builder

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

RUN npm run build


FROM nginx:alpine

COPY --from=builder /app/build /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

> Jika project menghasilkan directory `dist`, ubah `/app/build` menjadi `/app/dist`.

Build:

```bash
docker build \
    -t team1/dumbflix/frontend:staging \
    ./frontend
```

---

# 13. Docker Image Naming

Gunakan nama image berdasarkan environment.

## Staging

```text
team1/dumbflix/frontend:staging
team1/dumbflix/backend:staging
```

## Production

```text
team1/dumbflix/frontend:production
team1/dumbflix/backend:production
```

Ganti:

```text
team1
```

dengan nama team masing-masing.

---

# 14. Docker Compose Staging

Buat:

```text
staging/docker-compose.yml
```

Contoh:

```yaml
services:

  mysql:
    image: mysql:8.0
    container_name: wayshub-mysql-staging
    restart: unless-stopped

    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}

    volumes:
      - mysql_staging_data:/var/lib/mysql

    networks:
      - team1


  backend:
    image: team1/dumbflix/backend:staging
    container_name: wayshub-backend-staging
    restart: unless-stopped

    depends_on:
      - mysql

    networks:
      - team1


  frontend:
    image: team1/dumbflix/frontend:staging
    container_name: wayshub-frontend-staging
    restart: unless-stopped

    depends_on:
      - backend

    networks:
      - team1


  nginx:
    image: nginx:alpine
    container_name: wayshub-nginx-staging
    restart: unless-stopped

    ports:
      - "80:80"
      - "443:443"

    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - nginx_ssl:/etc/nginx/ssl

    depends_on:
      - frontend
      - backend

    networks:
      - team1


volumes:
  mysql_staging_data:
  nginx_ssl:


networks:
  team1:
    name: team1
    driver: bridge
```

> Ganti `team1` pada nama network dengan nama team masing-masing.

---

# 15. Staging Environment File

Buat:

```bash
nano staging/.env
```

Contoh:

```env
MYSQL_ROOT_PASSWORD=RootPassword123!
MYSQL_DATABASE=demo
MYSQL_USER=demo_user
MYSQL_PASSWORD=DemoPassword123!
```

Jangan commit `.env` yang berisi credential ke repository public.

Tambahkan:

```text
.env
```

ke `.gitignore`.

---

# 16. Staging Nginx Configuration

Buat:

```text
staging/nginx/nginx.conf
```

Contoh:

```nginx
events {}

http {

    upstream frontend {
        server frontend:80;
    }

    upstream backend {
        server backend:5000;
    }

    server {

        listen 80;

        server_name
            team1.staging.studentdumbways.my.id;

        location / {
            proxy_pass http://frontend;

            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }

    server {

        listen 80;

        server_name
            api.team1.staging.studentdumbways.my.id;

        location / {
            proxy_pass http://backend;

            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
```

---

# 17. Deploy Staging Database First

Masuk:

```bash
cd staging
```

Jalankan database terlebih dahulu:

```bash
docker compose up -d mysql
```

Cek:

```bash
docker compose ps
```

Cek volume:

```bash
docker volume ls
```

Test MySQL:

```bash
docker exec -it wayshub-mysql-staging mysql -u root -p
```

Kemudian:

```sql
SHOW DATABASES;
```

Pastikan:

```text
demo
```

tersedia.

---

# 18. Deploy Staging Backend

Setelah database siap:

```bash
docker compose up -d backend
```

Cek:

```bash
docker compose ps
```

Log:

```bash
docker compose logs backend
```

Test:

```bash
docker exec -it wayshub-backend-staging sh
```

Test koneksi ke MySQL:

```bash
ping mysql
```

---

# 19. Deploy Staging Frontend

```bash
docker compose up -d frontend
```

Cek:

```bash
docker compose ps
```

---

# 20. Deploy Staging Nginx

```bash
docker compose up -d nginx
```

Cek:

```bash
docker compose ps
```

Semua service harus berjalan:

```text
mysql       running
backend     running
frontend    running
nginx       running
```

---

# 21. Staging DNS

DNS yang digunakan:

```text
team1.staging.studentdumbways.my.id
api.team1.staging.studentdumbways.my.id
```

Arahkan:

```text
team1.staging.studentdumbways.my.id
        |
        v
<STAGING_SERVER_IP>


api.team1.staging.studentdumbways.my.id
        |
        v
<STAGING_SERVER_IP>
```

---

# 22. SSL Wildcard

SSL menggunakan wildcard.

Contoh:

```text
*.staging.studentdumbways.my.id
```

dan untuk production:

```text
*.studentdumbways.my.id
```

> **Cloudflare SSL: OFF**

DNS harus tetap menggunakan DNS provider yang digunakan untuk domain, tetapi SSL termination dilakukan oleh server sendiri menggunakan Nginx dan certificate.

---

# 23. Install Certbot

Install:

```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx -y
```

Cek:

```bash
certbot --version
```

Reference:

https://certbot.eff.org/instructions?ws=nginx&os=ubuntufocal

---

# 24. Wildcard Certificate

Wildcard certificate membutuhkan DNS challenge.

Contoh konsep:

```bash
sudo certbot certonly \
    --manual \
    --preferred-challenges dns \
    -d "*.staging.studentdumbways.my.id"
```

Certbot akan meminta DNS TXT record.

Tambahkan TXT record yang diberikan Certbot ke DNS.

Setelah propagasi:

```bash
sudo certbot certonly \
    --manual \
    --preferred-challenges dns \
    -d "*.staging.studentdumbways.my.id"
```

Certificate biasanya tersedia di:

```text
/etc/letsencrypt/live/
```

> Untuk production, ulangi dengan domain wildcard production.

---

# 25. Docker Volume for Reverse Proxy

Requirement:

> Gunakan Docker volume untuk membuat reverse proxy.

Volume yang digunakan:

```yaml
volumes:
  nginx_ssl:
```

Contoh mount:

```yaml
volumes:
  - nginx_ssl:/etc/nginx/ssl
```

Cek:

```bash
docker volume ls
```

---

# 26. Production Database Server

Production menggunakan server database terpisah.

Login:

```bash
ssh team1@<DATABASE_SERVER_IP>
```

Install Docker menggunakan script:

```bash
./install-docker.sh
```

Buat:

```text
production/database/docker-compose.yml
```

Contoh:

```yaml
services:

  mysql:
    image: mysql:8.0
    container_name: wayshub-mysql-production
    restart: unless-stopped

    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}

    volumes:
      - mysql_production_data:/var/lib/mysql

    ports:
      - "3306:3306"


volumes:
  mysql_production_data:
```

Jalankan:

```bash
docker compose up -d
```

Cek:

```bash
docker compose ps
```

---

# 27. Production Backend Server

Backend server memiliki **2 container backend**.

Contoh:

```text
Backend Server
│
├── wayshub-backend-1
└── wayshub-backend-2
```

Compose:

```yaml
services:

  backend1:
    image: team1/dumbflix/backend:production
    container_name: wayshub-backend-1
    restart: unless-stopped

    environment:
      DB_HOST: <DATABASE_SERVER_IP>
      DB_NAME: demo
      DB_USER: demo_user
      DB_PASSWORD: ${DB_PASSWORD}

    ports:
      - "5001:5000"


  backend2:
    image: team1/dumbflix/backend:production
    container_name: wayshub-backend-2
    restart: unless-stopped

    environment:
      DB_HOST: <DATABASE_SERVER_IP>
      DB_NAME: demo
      DB_USER: demo_user
      DB_PASSWORD: ${DB_PASSWORD}

    ports:
      - "5002:5000"
```

Jalankan:

```bash
docker compose up -d
```

Cek:

```bash
docker compose ps
```

Expected:

```text
wayshub-backend-1    running
wayshub-backend-2    running
```

---

# 28. Production Frontend Server

Frontend server memiliki 2 container:

```text
Frontend Server
│
├── wayshub-frontend-1
└── wayshub-frontend-2
```

Compose:

```yaml
services:

  frontend1:
    image: team1/dumbflix/frontend:production
    container_name: wayshub-frontend-1
    restart: unless-stopped

    ports:
      - "3001:80"


  frontend2:
    image: team1/dumbflix/frontend:production
    container_name: wayshub-frontend-2
    restart: unless-stopped

    ports:
      - "3002:80"
```

Jalankan:

```bash
docker compose up -d
```

---

# 29. Production Web Server

Web Server menggunakan Nginx dalam Docker.

```text
Web Server
    |
    v
Nginx
    |
    +-------- Frontend Server
    |          |
    |          +-- Frontend #1
    |          +-- Frontend #2
    |
    +-------- Backend Server
               |
               +-- Backend #1
               +-- Backend #2
```

---

# 30. Production Nginx Reverse Proxy

Contoh:

```nginx
events {}

http {

    upstream frontend_servers {
        server <FRONTEND_SERVER_IP>:3001;
        server <FRONTEND_SERVER_IP>:3002;
    }

    upstream backend_servers {
        server <BACKEND_SERVER_IP>:5001;
        server <BACKEND_SERVER_IP>:5002;
    }

    server {

        listen 443 ssl;

        server_name team1.studentdumbways.my.id;

        ssl_certificate
            /etc/nginx/ssl/fullchain.pem;

        ssl_certificate_key
            /etc/nginx/ssl/privkey.pem;

        location / {

            proxy_pass http://frontend_servers;

            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }

    server {

        listen 443 ssl;

        server_name api.team1.studentdumbways.my.id;

        ssl_certificate
            /etc/nginx/ssl/fullchain.pem;

        ssl_certificate_key
            /etc/nginx/ssl/privkey.pem;

        location / {

            proxy_pass http://backend_servers;

            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
```

Nginx akan melakukan load balancing sederhana:

```text
Request
   |
   v
Nginx
   |
   +----> Backend #1
   |
   +----> Backend #2
```

dan:

```text
Request
   |
   v
Nginx
   |
   +----> Frontend #1
   |
   +----> Frontend #2
```

---

# 31. Production DNS

Frontend:

```text
team1.studentdumbways.my.id
```

Backend:

```text
api.team1.studentdumbways.my.id
```

DNS:

```text
team1.studentdumbways.my.id
        |
        v
<WEB_SERVER_IP>


api.team1.studentdumbways.my.id
        |
        v
<WEB_SERVER_IP>
```

---

# 32. Docker Registry

Login:

```bash
docker login
```

Masukkan Docker Hub username dan password/token.

Build staging:

```bash
docker build \
    -t team1/dumbflix/backend:staging \
    ./backend
```

```bash
docker build \
    -t team1/dumbflix/frontend:staging \
    ./frontend
```

Push:

```bash
docker push team1/dumbflix/backend:staging
```

```bash
docker push team1/dumbflix/frontend:staging
```

Production:

```bash
docker build \
    -t team1/dumbflix/backend:production \
    ./backend
```

```bash
docker build \
    -t team1/dumbflix/frontend:production \
    ./frontend
```

Push:

```bash
docker push team1/dumbflix/backend:production
```

```bash
docker push team1/dumbflix/frontend:production
```

---

# 33. Application Testing

Test frontend:

```bash
curl -I https://team1.studentdumbways.my.id
```

Test backend:

```bash
curl -I https://api.team1.studentdumbways.my.id
```

Test aplikasi melalui browser:

```text
https://team1.studentdumbways.my.id
```

Test:

* Register
* Login
* Logout
* CRUD sesuai fitur aplikasi
* Backend API
* Database persistence

---

# 34. Jenkins Installation

Jenkins dapat dijalankan menggunakan Docker.

Buat directory:

```bash
mkdir -p ~/jenkins
cd ~/jenkins
```

Buat:

```text
docker-compose.yml
```

Contoh:

```yaml
services:

  jenkins:
    image: jenkins/jenkins:lts-jdk17
    container_name: jenkins
    restart: unless-stopped

    ports:
      - "8080:8080"
      - "50000:50000"

    volumes:
      - jenkins_home:/var/jenkins_home


volumes:
  jenkins_home:
```

Jalankan:

```bash
docker compose up -d
```

Cek:

```bash
docker compose ps
```

---

# 35. Jenkins Initial Password

Ambil password:

```bash
docker exec jenkins \
    cat /var/jenkins_home/secrets/initialAdminPassword
```

Buka:

```text
http://<JENKINS_SERVER_IP>:8080
```

Masukkan initial password.

Install suggested plugins.

Buat Jenkins admin account.

---

# 36. Jenkins SSH Key

Generate SSH key pada Jenkins server:

```bash
ssh-keygen -t ed25519
```

Cek:

```bash
cat ~/.ssh/id_ed25519.pub
```

Tambahkan public key ke server deployment.

Misalnya:

```text
Backend Server
Frontend Server
Database Server
Web Server
```

Test:

```bash
ssh team1@<SERVER_IP>
```

Jenkins harus dapat login tanpa password.

---

# 37. Jenkins SSH Credential

Pada Jenkins:

```text
Manage Jenkins
    |
    +-- Credentials
          |
          +-- System
                |
                +-- Global credentials
```

Tambahkan:

```text
Kind:
SSH Username with private key

Username:
team1

Private Key:
Jenkins id_ed25519
```

Simpan credential ID, misalnya:

```text
deployment-ssh
```

---

# 38. Jenkins Reverse Proxy

Domain:

```text
jenkins.team1.studentdumbways.my.id
```

Nginx:

```nginx
server {

    listen 443 ssl;

    server_name
        jenkins.team1.studentdumbways.my.id;

    ssl_certificate
        /etc/nginx/ssl/fullchain.pem;

    ssl_certificate_key
        /etc/nginx/ssl/privkey.pem;

    location / {

        proxy_pass http://jenkins:8080;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_http_version 1.1;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

---

# 39. Jenkins Jobs

Buat minimal job:

```text
Wayshub Backend Staging
Wayshub Backend Production
Wayshub Frontend Staging
Wayshub Frontend Production
```

Contoh flow:

```text
Git Repository
      |
      v
Pull Repository
      |
      v
Install Dependencies
      |
      v
Docker Build
      |
      v
Application Test
      |
      v
Docker Push
      |
      v
Deploy Docker
      |
      v
Restart Container
      |
      v
Notification Discord
```

---

# 40. Jenkins Backend Staging Pipeline

Contoh `Jenkinsfile`:

```groovy
pipeline {

    agent any

    environment {
        IMAGE = "team1/dumbflix/backend:staging"
    }

    stages {

        stage('Checkout') {
            steps {
                git 'https://github.com/dumbwaysdev/wayshub-backend.git'
            }
        }

        stage('Install') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test || true'
            }
        }

        stage('Build Docker') {
            steps {
                sh "docker build -t ${IMAGE} ."
            }
        }

        stage('Push Docker') {
            steps {
                sh "docker push ${IMAGE}"
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    ssh team1@<STAGING_SERVER_IP> \
                    "docker pull ${IMAGE} && \
                     docker compose -f ~/staging/docker-compose.yml up -d backend"
                '''
            }
        }
    }
}
```

> Sesuaikan command test, directory deployment, credential, dan Docker registry dengan environment sebenarnya.

---

# 41. Jenkins Production Pipeline

Flow production:

```text
Checkout
   |
Install
   |
Test
   |
Build Docker
   |
Push Docker Hub
   |
SSH Production Server
   |
Docker Pull
   |
Docker Compose Up
   |
Application Running
```

Contoh deployment:

```bash
docker pull team1/dumbflix/backend:production

docker compose \
    -f docker-compose.yml \
    up -d
```

---

# 42. Jenkins SCM Trigger

Gunakan webhook dari repository.

Flow:

```text
Developer Push
       |
       v
GitHub/GitLab
       |
       v
Webhook
       |
       v
Jenkins
       |
       v
Pipeline
       |
       v
Docker Build
       |
       v
Docker Hub
       |
       v
Deployment
```

Aktifkan:

```text
Build Triggers
    |
    +-- GitHub hook trigger for GITScm polling
```

atau trigger SCM yang sesuai dengan source repository.

---

# 43. Jenkins Discord Notification

Install plugin yang dibutuhkan untuk Discord notification atau gunakan Discord Webhook melalui script.

Contoh menggunakan curl:

```bash
curl -H "Content-Type: application/json" \
     -d '{"content":"Wayshub deployment berhasil!"}' \
     "$DISCORD_WEBHOOK"
```

Gunakan environment variable:

```text
DISCORD_WEBHOOK
```

Jangan hardcode Discord webhook di repository.

Contoh Jenkins post:

```groovy
post {

    success {
        sh '''
            curl -H "Content-Type: application/json" \
            -d '{"content":"Wayshub Jenkins deployment SUCCESS"}' \
            "$DISCORD_WEBHOOK"
        '''
    }

    failure {
        sh '''
            curl -H "Content-Type: application/json" \
            -d '{"content":"Wayshub Jenkins deployment FAILED"}' \
            "$DISCORD_WEBHOOK"
        '''
    }
}
```

---

# 44. GitLab Runner

GitLab Runner digunakan untuk pipeline Frontend.

Install Runner pada server:

```bash
sudo apt update
```

Install package:

```bash
sudo apt install gitlab-runner -y
```

Cek:

```bash
gitlab-runner --version
```

---

# 45. Register GitLab Runner

Register:

```bash
sudo gitlab-runner register
```

Masukkan:

```text
GitLab URL:
https://gitlab.com/

Registration Token:
<YOUR_RUNNER_TOKEN>

Description:
wayshub-frontend-runner

Executor:
shell
```

Cek:

```bash
sudo gitlab-runner list
```

---

# 46. GitLab CI/CD Pipeline

Buat file:

```text
.gitlab-ci.yml
```

Contoh:

```yaml
stages:
  - checkout
  - install
  - test
  - build
  - push
  - deploy
  - notification


variables:
  IMAGE_NAME: team1/dumbflix/frontend


checkout:
  stage: checkout
  script:
    - echo "Repository checkout completed"


install:
  stage: install
  script:
    - npm install


test:
  stage: test
  script:
    - npm test || true


build:
  stage: build
  script:
    - docker build -t $IMAGE_NAME:$CI_COMMIT_SHORT_SHA .


push:
  stage: push
  script:
    - docker push $IMAGE_NAME:$CI_COMMIT_SHORT_SHA


deploy_staging:
  stage: deploy
  only:
    - staging

  script:
    - ssh team1@<STAGING_SERVER_IP> "
        docker pull $IMAGE_NAME:$CI_COMMIT_SHORT_SHA &&
        docker compose
        -f ~/staging/docker-compose.yml
        up -d frontend
      "


deploy_production:
  stage: deploy
  only:
    - main

  script:
    - ssh team1@<PRODUCTION_SERVER_IP> "
        docker pull $IMAGE_NAME:$CI_COMMIT_SHORT_SHA &&
        docker compose
        -f ~/production/docker-compose.yml
        up -d
      "


notification:
  stage: notification

  script:
    - |
      curl -H "Content-Type: application/json" \
      -d "{\"content\":\"GitLab CI/CD pipeline ${CI_PIPELINE_STATUS}\"}" \
      "$DISCORD_WEBHOOK"
```

---

# 47. GitLab Environment

Gunakan branch:

```text
staging
main
```

Flow:

```text
feature
   |
   v
staging
   |
   v
Production
```

Staging:

```text
staging branch
       |
       v
GitLab Runner
       |
       v
Build
       |
       v
Test
       |
       v
Docker Push
       |
       v
Staging Deployment
```

Production:

```text
main branch
     |
     v
GitLab Runner
     |
     v
Build
     |
     v
Test
     |
     v
Docker Push
     |
     v
Production Deployment
```

---

# 48. GitLab Auto Trigger

GitLab pipeline otomatis dijalankan ketika terdapat perubahan pada repository.

Contoh:

```bash
git add .
git commit -m "update frontend"
git push origin staging
```

GitLab akan:

```text
Push
 |
 v
Pipeline
 |
 +--> Build
 |
 +--> Test
 |
 +--> Docker Build
 |
 +--> Docker Push
 |
 +--> Deploy
 |
 +--> Discord Notification
```

---

# 49. CI/CD Final Flow

## Jenkins

```text
Developer
    |
    v
Git Repository
    |
    v
Jenkins Webhook
    |
    v
Checkout
    |
    v
Install Dependencies
    |
    v
Test
    |
    v
Docker Build
    |
    v
Push Docker Hub
    |
    v
SSH Deployment Server
    |
    v
Docker Pull
    |
    v
Docker Compose
    |
    v
Application
    |
    v
Discord Notification
```

## GitLab Runner

```text
Developer
    |
    v
GitLab
    |
    v
GitLab Runner
    |
    v
Checkout
    |
    v
Install
    |
    v
Test
    |
    v
Docker Build
    |
    v
Push Docker Hub
    |
    v
Docker Deploy
    |
    v
Discord Notification
```

---

# 50. Security Checklist

* [ ] SSH menggunakan SSH Key
* [ ] Password SSH disabled
* [ ] Root SSH login disabled
* [ ] Docker credential tidak disimpan di repository
* [ ] `.env` tidak di-push
* [ ] Database password tidak di-hardcode pada public repository
* [ ] Docker Hub menggunakan Access Token
* [ ] Jenkins credential menggunakan Jenkins Credentials
* [ ] GitLab CI/CD Variables digunakan untuk secret
* [ ] Discord Webhook disimpan sebagai secret
* [ ] SSL menggunakan certificate valid
* [ ] Cloudflare SSL **OFF**
* [ ] Database production tidak diekspos secara bebas ke internet

---

# 51. Documentation Checklist

## Docker

* [ ] Create team user
* [ ] Docker installation script
* [ ] Docker installed
* [ ] Docker Compose installed
* [ ] Backend Dockerfile
* [ ] Frontend Dockerfile
* [ ] Multistage build
* [ ] Staging Compose
* [ ] Production Compose
* [ ] Custom Docker network
* [ ] Database volume
* [ ] Reverse proxy volume
* [ ] Docker images pushed to registry

## Staging

* [ ] MySQL container
* [ ] Backend container
* [ ] Frontend container
* [ ] Nginx container
* [ ] Custom network
* [ ] DNS configured
* [ ] SSL configured
* [ ] Login tested
* [ ] Register tested

## Production

* [ ] Database server separated
* [ ] Backend server separated
* [ ] Two backend containers
* [ ] Frontend server separated
* [ ] Two frontend containers
* [ ] Web server separated
* [ ] Nginx reverse proxy
* [ ] SSL wildcard
* [ ] DNS configured
* [ ] Login tested
* [ ] Register tested

## Jenkins

* [ ] Jenkins installed
* [ ] Jenkins running
* [ ] Jenkins SSH Key configured
* [ ] SSH deployment tested
* [ ] Jenkins reverse proxy
* [ ] Jenkins domain configured
* [ ] Staging job
* [ ] Production job
* [ ] Repository checkout
* [ ] Docker build
* [ ] Application test
* [ ] Docker push
* [ ] Docker deployment
* [ ] SCM trigger
* [ ] Discord notification

## GitLab

* [ ] GitLab Runner installed
* [ ] GitLab Runner registered
* [ ] Frontend connected to Runner
* [ ] Staging pipeline
* [ ] Production pipeline
* [ ] Repository checkout
* [ ] Docker build
* [ ] Application test
* [ ] Docker push
* [ ] Docker deployment
* [ ] SCM trigger
* [ ] Discord notification

---

# 52. Screenshot Documentation

Simpan screenshot pada:

```text
docs/
└── screenshots/
```

Recommended:

```text
docs/screenshots/
├── 01-create-team-user.png
├── 02-docker-installation.png
├── 03-docker-version.png
├── 04-docker-compose-version.png
├── 05-backend-dockerfile.png
├── 06-frontend-dockerfile.png
├── 07-staging-compose.png
├── 08-staging-network.png
├── 09-staging-mysql.png
├── 10-staging-volume.png
├── 11-staging-backend.png
├── 12-staging-frontend.png
├── 13-staging-nginx.png
├── 14-staging-dns.png
├── 15-staging-ssl.png
├── 16-production-database.png
├── 17-production-backend-1.png
├── 18-production-backend-2.png
├── 19-production-frontend-1.png
├── 20-production-frontend-2.png
├── 21-production-nginx.png
├── 22-production-ssl.png
├── 23-dockerhub-images.png
├── 24-wayshub-login.png
├── 25-wayshub-register.png
├── 26-jenkins.png
├── 27-jenkins-ssh.png
├── 28-jenkins-reverse-proxy.png
├── 29-jenkins-staging-job.png
├── 30-jenkins-production-job.png
├── 31-jenkins-build.png
├── 32-jenkins-webhook.png
├── 33-jenkins-discord.png
├── 34-gitlab-runner.png
├── 35-gitlab-ci.png
├── 36-gitlab-staging.png
├── 37-gitlab-production.png
├── 38-gitlab-docker-build.png
├── 39-gitlab-deployment.png
└── 40-discord-notification.png
```

---

# 53. Final Verification

## Staging

Frontend:

```text
https://team1.staging.studentdumbways.my.id
```

Backend:

```text
https://api.team1.staging.studentdumbways.my.id
```

Check containers:

```bash
docker compose ps
```

Expected:

```text
mysql       running
backend     running
frontend    running
nginx       running
```

---

## Production

Frontend:

```text
https://team1.studentdumbways.my.id
```

Backend:

```text
https://api.team1.studentdumbways.my.id
```

Check Backend Server:

```bash
docker ps
```

Expected:

```text
wayshub-backend-1
wayshub-backend-2
```

Check Frontend Server:

```bash
docker ps
```

Expected:

```text
wayshub-frontend-1
wayshub-frontend-2
```

Check Web Server:

```bash
docker ps
```

Expected:

```text
nginx
```

Check Database:

```bash
docker ps
```

Expected:

```text
wayshub-mysql-production
```

---

# 54. Application Test

Test menggunakan browser:

```text
Staging:
https://team1.staging.studentdumbways.my.id

Production:
https://team1.studentdumbways.my.id
```

Test fitur:

* [ ] Register berhasil
* [ ] Login berhasil
* [ ] Logout berhasil
* [ ] API dapat diakses
* [ ] Backend dapat mengakses database
* [ ] Data tersimpan pada database
* [ ] Frontend dapat berkomunikasi dengan backend
* [ ] SSL valid
* [ ] Reverse proxy berjalan
* [ ] Load balancing backend berjalan
* [ ] Load balancing frontend berjalan

---

# 55. Final Architecture

```text
                              INTERNET
                                  |
                                  |
                         +--------+--------+
                         |                 |
                         v                 v
                  STAGING DOMAIN     PRODUCTION DOMAIN
                         |                 |
                         v                 v
                  +-------------+   +-------------+
                  |   NGINX     |   |   NGINX     |
                  |  STAGING    |   | PRODUCTION  |
                  +------+------+   +------+------+
                         |                 |
              +----------+----------+      |
              |          |          |      |
              v          v          v      |
           Frontend   Backend     MySQL    |
              |          |          |      |
              +----------+----------+      |
                                           |
                           +---------------+---------------+
                           |               |               |
                           v               v               v
                    Frontend Server  Backend Server  Database Server
                           |               |               |
                           |          +----+----+          |
                           |          |         |          |
                           |          v         v          |
                           |      Backend #1 Backend #2    |
                           |                              MySQL
                           |          +--------+             |
                           |                   |             |
                           +-------------------+-------------+
                                               |
                                               v
                                          APPLICATION
```

---

# 56. Final Checklist

### Docker

* [ ] Team user created
* [ ] Docker installation script created
* [ ] Docker installed
* [ ] Docker Compose installed
* [ ] Backend Dockerized
* [ ] Frontend Dockerized
* [ ] Multistage build implemented
* [ ] Staging environment deployed
* [ ] Production environment deployed
* [ ] Docker network configured
* [ ] Database volume configured
* [ ] Nginx volume configured
* [ ] Docker images pushed to Docker Hub

### Staging

* [ ] Web Server
* [ ] Frontend
* [ ] Backend
* [ ] MySQL
* [ ] Custom network
* [ ] DNS
* [ ] SSL
* [ ] Register
* [ ] Login

### Production

* [ ] Separate Database Server
* [ ] Separate Backend Server
* [ ] Backend container #1
* [ ] Backend container #2
* [ ] Separate Frontend Server
* [ ] Frontend container #1
* [ ] Frontend container #2
* [ ] Separate Web Server
* [ ] Nginx Reverse Proxy
* [ ] Wildcard SSL
* [ ] DNS
* [ ] Register
* [ ] Login

### Jenkins

* [ ] Jenkins installed
* [ ] Jenkins Docker/native deployment
* [ ] SSH Key configured
* [ ] Remote SSH login successful
* [ ] Jenkins reverse proxy
* [ ] Jenkins domain
* [ ] Staging job
* [ ] Production job
* [ ] Pull repository
* [ ] Build Docker image
* [ ] Test application
* [ ] Push Docker image
* [ ] Deploy application
* [ ] SCM auto trigger
* [ ] Discord notification

### GitLab

* [ ] GitLab Runner installed
* [ ] Runner registered
* [ ] Frontend connected to Runner
* [ ] Staging pipeline
* [ ] Production pipeline
* [ ] Pull repository
* [ ] Build Docker image
* [ ] Test application
* [ ] Push Docker image
* [ ] Deploy application
* [ ] SCM auto trigger
* [ ] Discord notification

---

# 57. References

* Wayshub Backend: https://github.com/dumbwaysdev/wayshub-backend.git
* Wayshub Frontend: https://github.com/dumbwaysdev/wayshub-frontend.git
* Certbot: https://certbot.eff.org/instructions?ws=nginx&os=ubuntufocal
* PM2 Runtime with Docker: https://pm2.keymetrics.io/docs/usage/docker-pm2-nodejs

---

# Conclusion

Deployment Wayshub telah mencakup:

1. Docker installation menggunakan bash script.
2. Dockerized Backend.
3. Dockerized Frontend.
4. Multistage Docker build.
5. Staging environment.
6. Production environment.
7. MySQL Database dengan persistent volume.
8. Custom Docker network.
9. Nginx Reverse Proxy menggunakan Docker.
10. Wildcard SSL tanpa Cloudflare SSL.
11. Docker Registry.
12. Jenkins CI/CD.
13. Jenkins SSH deployment.
14. Jenkins SCM trigger.
15. Jenkins Discord notification.
16. GitLab Runner.
17. GitLab CI/CD.
18. Automatic deployment.
19. Docker image build dan push.
20. Testing aplikasi.
21. Register dan Login verification.
