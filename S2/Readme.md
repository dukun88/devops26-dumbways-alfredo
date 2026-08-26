# Wayshub Deployment Documentation

Dokumentasi pengerjaan deployment **Wayshub Backend & Frontend**, MySQL Database, SSH Key Authentication, Role Based Access Control (RBAC), Remote Database Access, PM2, dan Nginx.

---

## 1. Architecture

Deployment menggunakan 2 server:

```text
                         LOCAL COMPUTER
                              |
                    SSH Key / MySQL Client
                              |
             +----------------+----------------+
             |                                 |
             v                                 v
       APP SERVER                         GATEWAY SERVER
       MySQL Database                     Wayshub Backend
       Ubuntu                             Wayshub Frontend
                                          Nginx
                                          PM2
             |                                 |
             +---------- Private Network ------+
```

### Server

| Server         | IP Address    | Service                       |
| -------------- | ------------- | ----------------------------- |
| App Server     | `10.10.10.10` | MySQL Database                |
| Gateway Server | `10.10.10.20` | Backend, Frontend, Nginx, PM2 |
| Local Computer | -             | SSH Client & MySQL Client     |

> **Note:** Ganti IP address di atas dengan IP server yang digunakan pada deployment sebenarnya.

---

# 2. Server Preparation

## 2.1 App Server

App Server digunakan untuk:

* MySQL Database
* Database `demo`
* Table `transaction`
* MySQL User
* MySQL Role

## 2.2 Gateway Server

Gateway Server digunakan untuk:

* Wayshub Backend
* Wayshub Frontend
* Node.js 14
* PM2
* Nginx / Web Server

---

# 3. SSH Key Authentication

Requirement:

> Server hanya dapat login menggunakan SSH Key dan tidak menggunakan password.

## 3.1 Generate SSH Key

Pada komputer lokal:

```bash
ssh-keygen -t ed25519
```

Cek SSH Key:

```bash
ls ~/.ssh
```

File yang dihasilkan:

```text
id_ed25519
id_ed25519.pub
```

Public key:

```text
id_ed25519.pub
```

Private key:

```text
id_ed25519
```

> **WARNING:** Jangan membagikan private key kepada siapa pun.

---

# 4. Create User on App Server

Login menggunakan user awal server:

```bash
ssh root@10.10.10.10
```

Buat user baru:

```bash
adduser marco
```

Tambahkan user ke group sudo:

```bash
usermod -aG sudo marco
```

Buat directory SSH:

```bash
mkdir -p /home/marco/.ssh
```

Buat file authorized keys:

```bash
nano /home/marco/.ssh/authorized_keys
```

Paste isi dari:

```text
id_ed25519.pub
```

Atur permission:

```bash
chmod 700 /home/marco/.ssh
chmod 600 /home/marco/.ssh/authorized_keys
chown -R marco:marco /home/marco/.ssh
```

---

# 5. Create User on Gateway Server

Login ke Gateway:

```bash
ssh root@10.10.10.20
```

Buat user:

```bash
adduser marco
```

Tambahkan sudo:

```bash
usermod -aG sudo marco
```

Buat directory SSH:

```bash
mkdir -p /home/marco/.ssh
```

Buat authorized keys:

```bash
nano /home/marco/.ssh/authorized_keys
```

Paste public key:

```text
id_ed25519.pub
```

Atur permission:

```bash
chmod 700 /home/marco/.ssh
chmod 600 /home/marco/.ssh/authorized_keys
chown -R marco:marco /home/marco/.ssh
```

---

# 6. Test SSH Key

Dari komputer lokal:

```bash
ssh marco@10.10.10.10
```

Test Gateway:

```bash
ssh marco@10.10.10.20
```

Pastikan kedua server dapat diakses menggunakan SSH Key.

---

# 7. Disable SSH Password Authentication

> **Important:** Pastikan login menggunakan SSH Key sudah berhasil sebelum melakukan langkah ini.

Edit konfigurasi SSH pada App Server:

```bash
sudo nano /etc/ssh/sshd_config
```

Ubah:

```text
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin no
```

Restart SSH:

```bash
sudo systemctl restart ssh
```

Lakukan langkah yang sama pada Gateway Server.

Test:

```bash
ssh marco@10.10.10.10
ssh marco@10.10.10.20
```

Login menggunakan password harus ditolak.

---

# 8. Install MySQL

Login ke App Server:

```bash
ssh marco@10.10.10.10
```

Update package:

```bash
sudo apt update
sudo apt upgrade -y
```

Install MySQL:

```bash
sudo apt install mysql-server -y
```

Cek service:

```bash
sudo systemctl status mysql
```

---

# 9. MySQL Secure Installation

Jalankan:

```bash
sudo mysql_secure_installation
```

Konfigurasi keamanan:

```text
Remove anonymous users: Y
Disallow root login remotely: Y
Remove test database: Y
Reload privilege tables: Y
```

Screenshot:

```text
![MySQL Secure Installation](docs/screenshots/mysql-secure-installation.png)
```

---

# 10. Set MySQL Root Password

Login ke MySQL:

```bash
sudo mysql
```

Set password:

```sql
ALTER USER 'root'@'localhost'
IDENTIFIED WITH mysql_native_password BY 'RootPassword123!';
```

Reload privilege:

```sql
FLUSH PRIVILEGES;
```

Keluar:

```sql
EXIT;
```

Test:

```bash
mysql -u root -p
```

---

# 11. Create Database

Login:

```bash
mysql -u root -p
```

Buat database:

```sql
CREATE DATABASE demo;
```

Cek:

```sql
SHOW DATABASES;
```

Database yang harus tersedia:

```text
demo
```

---

# 12. Create MySQL User

Buat user:

```sql
CREATE USER 'demo_user'@'%'
IDENTIFIED BY 'DemoPassword123!';
```

Berikan privilege:

```sql
GRANT ALL PRIVILEGES
ON demo.*
TO 'demo_user'@'%';
```

Reload:

```sql
FLUSH PRIVILEGES;
```

Cek:

```sql
SHOW GRANTS FOR 'demo_user'@'%';
```

---

# 13. Configure MySQL Bind Address

Edit:

```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

Cari:

```text
bind-address = 127.0.0.1
```

Ubah menjadi:

```text
bind-address = 0.0.0.0
```

Restart MySQL:

```bash
sudo systemctl restart mysql
```

Cek port MySQL:

```bash
sudo ss -lntp | grep 3306
```

MySQL harus listen pada port:

```text
3306
```

---

# 14. Create Transaction Table

Login ke MySQL:

```bash
mysql -u root -p
```

Pilih database:

```sql
USE demo;
```

Buat table:

```sql
CREATE TABLE transaction (
    id INT AUTO_INCREMENT PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    quantity INT NOT NULL,
    price DECIMAL(12,2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

# 15. Insert Dummy Data

```sql
INSERT INTO transaction
(product_name, quantity, price)
VALUES
('Laptop', 2, 15000000),
('Mouse', 5, 250000),
('Keyboard', 3, 500000);
```

Test:

```sql
SELECT * FROM transaction;
```

---

# 16. Create Admin Role

Buat role:

```sql
CREATE ROLE 'admin';
```

Berikan permission:

```sql
GRANT SELECT, INSERT, UPDATE, DELETE
ON demo.transaction
TO 'admin';
```

Cek:

```sql
SHOW GRANTS FOR 'admin';
```

Admin memiliki permission:

```text
SELECT
INSERT
UPDATE
DELETE
```

---

# 17. Create Guest Role

Buat role:

```sql
CREATE ROLE 'guest';
```

Berikan hanya SELECT:

```sql
GRANT SELECT
ON demo.transaction
TO 'guest';
```

Cek:

```sql
SHOW GRANTS FOR 'guest';
```

Guest hanya memiliki:

```text
SELECT
```

---

# 18. Create Admin User

Ganti `marco` dengan username yang digunakan.

```sql
CREATE USER 'marco'@'%'
IDENTIFIED BY 'your_password';
```

Tambahkan ke role admin:

```sql
GRANT 'admin'
TO 'marco'@'%';
```

Set default role:

```sql
SET DEFAULT ROLE 'admin'
TO 'marco'@'%';
```

---

# 19. Create Guest User

Buat user:

```sql
CREATE USER 'guest'@'%'
IDENTIFIED BY 'guest';
```

Tambahkan role:

```sql
GRANT 'guest'
TO 'guest'@'%';
```

Set default role:

```sql
SET DEFAULT ROLE 'guest'
TO 'guest'@'%';
```

---

# 20. Test Admin User

Login:

```bash
mysql -u marco -p -h 10.10.10.10
```

Pilih database:

```sql
USE demo;
```

## SELECT

```sql
SELECT * FROM transaction;
```

Expected:

```text
SUCCESS
```

## INSERT

```sql
INSERT INTO transaction
(product_name, quantity, price)
VALUES
('Monitor', 2, 2500000);
```

Expected:

```text
SUCCESS
```

## UPDATE

```sql
UPDATE transaction
SET quantity = 10
WHERE id = 1;
```

Expected:

```text
SUCCESS
```

## DELETE

```sql
DELETE FROM transaction
WHERE id = 3;
```

Expected:

```text
SUCCESS
```

---

# 21. Test Guest User

Login:

```bash
mysql -u guest -p -h 10.10.10.10
```

Pilih database:

```sql
USE demo;
```

## SELECT

```sql
SELECT * FROM transaction;
```

Expected:

```text
SUCCESS
```

## INSERT

```sql
INSERT INTO transaction
(product_name, quantity, price)
VALUES
('Test', 1, 1000);
```

Expected:

```text
ERROR 1142
```

Operation harus ditolak.

## UPDATE

```sql
UPDATE transaction
SET quantity = 99
WHERE id = 1;
```

Expected:

```text
ERROR 1142
```

Operation harus ditolak.

## DELETE

```sql
DELETE FROM transaction
WHERE id = 1;
```

Expected:

```text
ERROR 1142
```

Operation harus ditolak.

---

# 22. Remote MySQL Access

Install MySQL Client pada komputer lokal.

Kemudian:

```bash
mysql -h 10.10.10.10 -P 3306 -u marco -p
```

Masukkan password.

Test:

```sql
USE demo;
```

Kemudian:

```sql
SHOW TABLES;
```

Test:

```sql
SELECT * FROM transaction;
```

Jika data muncul, remote database berhasil.

---

# 23. Install Node.js 14

Login Gateway:

```bash
ssh marco@10.10.10.20
```

Install NVM:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
```

Reload shell:

```bash
source ~/.bashrc
```

Install Node.js 14:

```bash
nvm install 14
```

Gunakan Node.js 14:

```bash
nvm use 14
```

Set default:

```bash
nvm alias default 14
```

Cek:

```bash
node -v
npm -v
```

Expected:

```text
v14.x.x
```

---

# 24. Clone Wayshub Backend

Buat directory:

```bash
sudo mkdir -p /var/www
sudo chown -R $USER:$USER /var/www
```

Masuk:

```bash
cd /var/www
```

Clone repository:

```bash
git clone <WAYS HUB BACKEND REPOSITORY>
```

Masuk ke project:

```bash
cd wayshub-backend
```

Install dependency:

```bash
npm install
```

---

# 25. Configure Wayshub Backend

Edit:

```bash
nano config/config.json
```

Sesuaikan konfigurasi database:

```json
{
  "development": {
    "username": "demo_user",
    "password": "DemoPassword123!",
    "database": "demo",
    "host": "10.10.10.10",
    "dialect": "mysql"
  }
}
```

Konfigurasi:

| Configuration | Value          |
| ------------- | -------------- |
| Username      | `demo_user`    |
| Password      | Password MySQL |
| Database      | `demo`         |
| Host          | IP App Server  |
| Dialect       | `mysql`        |

> **Important:** Jangan menggunakan `localhost` untuk database karena MySQL berada di App Server.

---

# 26. Install Sequelize CLI

Install:

```bash
npm install -g sequelize-cli
```

Cek:

```bash
sequelize --version
```

Atau:

```bash
npx sequelize-cli --version
```

---

# 27. Run Database Migration

Dari directory backend:

```bash
npx sequelize-cli db:migrate
```

Jika berhasil, cek database:

```bash
mysql -u root -p
```

```sql
USE demo;
```

```sql
SHOW TABLES;
```

---

# 28. Test Backend

Jalankan backend:

```bash
npm start
```

atau:

```bash
npm run start
```

Gunakan command yang tersedia pada `package.json`.

Test menggunakan:

```bash
curl http://localhost:5000
```

> Ganti port `5000` sesuai port backend yang digunakan.

Jika berhasil, hentikan:

```text
CTRL + C
```

---

# 29. Deploy Backend Using PM2

Install PM2:

```bash
npm install -g pm2
```

Start backend:

```bash
pm2 start npm --name wayshub-backend -- start
```

Cek:

```bash
pm2 status
```

Expected:

```text
wayshub-backend    online
```

Cek log:

```bash
pm2 logs wayshub-backend
```

Save PM2:

```bash
pm2 save
```

Enable startup:

```bash
pm2 startup
```

Jalankan command yang diberikan oleh PM2.

Kemudian:

```bash
pm2 save
```

---

# 30. Clone Wayshub Frontend

Masuk:

```bash
cd /var/www
```

Clone:

```bash
git clone <WAYS HUB FRONTEND REPOSITORY>
```

Masuk:

```bash
cd wayshub-frontend
```

Gunakan Node.js 14:

```bash
nvm use 14
```

Install dependency:

```bash
npm install
```

---

# 31. Configure Wayshub Frontend

Edit:

```bash
nano src/config/api.js
```

Ubah API URL agar mengarah ke backend.

Contoh:

```javascript
const API = "http://10.10.10.20:5000";
```

Sesuaikan dengan struktur konfigurasi Wayshub Frontend.

Backend harus mengarah ke:

```text
Gateway Server
```

---

# 32. Build Frontend

Jalankan:

```bash
npm run build
```

Setelah selesai, pastikan directory hasil build tersedia.

Biasanya:

```text
build/
```

atau:

```text
dist/
```

Tergantung project.

---

# 33. Deploy Frontend Using PM2

Install static server:

```bash
npm install -g serve
```

Jika hasil build berada di `build`:

```bash
pm2 start "serve -s build -l 3000" --name wayshub-frontend
```

Jika hasil build berada di `dist`:

```bash
pm2 start "serve -s dist -l 3000" --name wayshub-frontend
```

Cek:

```bash
pm2 status
```

Expected:

```text
wayshub-backend     online
wayshub-frontend    online
```

Save:

```bash
pm2 save
```

---

# 34. Install Nginx

Install:

```bash
sudo apt install nginx -y
```

Cek:

```bash
sudo systemctl status nginx
```

Test:

```bash
curl http://localhost
```

---

# 35. Configure Nginx

Buat konfigurasi:

```bash
sudo nano /etc/nginx/sites-available/wayshub
```

Contoh konfigurasi:

```nginx
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:3000;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:5000/;

        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

> Port `3000`, `5000`, dan path `/api/` harus disesuaikan dengan aplikasi Wayshub yang digunakan.

Enable configuration:

```bash
sudo ln -s /etc/nginx/sites-available/wayshub /etc/nginx/sites-enabled/
```

Hapus default configuration:

```bash
sudo rm /etc/nginx/sites-enabled/default
```

Test Nginx:

```bash
sudo nginx -t
```

Expected:

```text
syntax is ok
test is successful
```

Restart:

```bash
sudo systemctl restart nginx
```

---

# 36. Final Testing

## 36.1 Test SSH

App Server:

```bash
ssh marco@10.10.10.10
```

Gateway:

```bash
ssh marco@10.10.10.20
```

Password authentication harus disabled.

---

## 36.2 Test MySQL

```bash
mysql -h 10.10.10.10 -P 3306 -u marco -p
```

```sql
USE demo;
```

```sql
SELECT * FROM transaction;
```

---

## 36.3 Test RBAC

### Admin

```text
SELECT  = ALLOWED
INSERT  = ALLOWED
UPDATE  = ALLOWED
DELETE  = ALLOWED
```

### Guest

```text
SELECT  = ALLOWED
INSERT  = DENIED
UPDATE  = DENIED
DELETE  = DENIED
```

---

## 36.4 Test PM2

```bash
pm2 status
```

Expected:

```text
wayshub-backend     online
wayshub-frontend    online
```

---

## 36.5 Test Nginx

```bash
sudo systemctl status nginx
```

Expected:

```text
active (running)
```

---

## 36.6 Test Website

Buka browser:

```text
http://10.10.10.20
```

Wayshub Frontend harus dapat diakses.

---

# 37. Documentation Screenshots

Simpan screenshot dokumentasi pada directory:

```text
docs/
└── screenshots/
```

Struktur yang disarankan:

```text
docs/
└── screenshots/
    ├── 01-server.png
    ├── 02-ssh-key.png
    ├── 03-ssh-config.png
    ├── 04-mysql-installation.png
    ├── 05-mysql-secure-installation.png
    ├── 06-mysql-bind-address.png
    ├── 07-database-demo.png
    ├── 08-transaction-table.png
    ├── 09-admin-role.png
    ├── 10-guest-role.png
    ├── 11-admin-test.png
    ├── 12-guest-test.png
    ├── 13-remote-mysql.png
    ├── 14-node-version.png
    ├── 15-backend-config.png
    ├── 16-sequelize-migration.png
    ├── 17-pm2-backend.png
    ├── 18-frontend-config.png
    ├── 19-frontend-build.png
    ├── 20-pm2-frontend.png
    ├── 21-nginx-config.png
    ├── 22-nginx-test.png
    └── 23-wayshub.png
```

---

# 38. Requirement Checklist

## Server

* [ ] App Server berhasil dibuat
* [ ] Gateway Server berhasil dibuat
* [ ] App Server digunakan untuk MySQL
* [ ] Gateway digunakan untuk Frontend, Backend, dan Web Server

## User & SSH

* [ ] Membuat user baru di App Server
* [ ] Membuat user baru di Gateway
* [ ] SSH Key berhasil dibuat
* [ ] SSH Key berhasil digunakan
* [ ] Password SSH authentication disabled
* [ ] Root SSH login disabled

## MySQL

* [ ] MySQL berhasil di-install
* [ ] `mysql_secure_installation` berhasil dijalankan
* [ ] Root MySQL memiliki password
* [ ] User MySQL berhasil dibuat
* [ ] Database `demo` berhasil dibuat
* [ ] MySQL `bind-address` berhasil dikonfigurasi

## RBAC

* [ ] Table `transaction` berhasil dibuat
* [ ] Dummy data berhasil dibuat
* [ ] Role `admin` berhasil dibuat
* [ ] Role `guest` berhasil dibuat
* [ ] Admin memiliki SELECT
* [ ] Admin memiliki INSERT
* [ ] Admin memiliki UPDATE
* [ ] Admin memiliki DELETE
* [ ] Guest hanya memiliki SELECT
* [ ] User `your_name` berhasil dibuat
* [ ] User `your_name` masuk role admin
* [ ] User `guest` berhasil dibuat
* [ ] User `guest` masuk role guest
* [ ] Semua user berhasil dites

## Remote Database

* [ ] MySQL Client berhasil di-install
* [ ] Remote connection berhasil
* [ ] Database `demo` dapat diakses dari komputer lokal

## Backend

* [ ] Node.js 14 berhasil di-install
* [ ] Wayshub Backend berhasil di-clone
* [ ] Dependency berhasil di-install
* [ ] `config/config.json` berhasil dikonfigurasi
* [ ] Sequelize CLI berhasil di-install
* [ ] Migration berhasil dijalankan
* [ ] Backend berhasil dijalankan
* [ ] Backend berhasil dijalankan menggunakan PM2

## Frontend

* [ ] Wayshub Frontend berhasil di-clone
* [ ] Node.js 14 digunakan
* [ ] Dependency berhasil di-install
* [ ] `src/config/api.js` berhasil dikonfigurasi
* [ ] Frontend berhasil di-build
* [ ] Frontend berhasil dijalankan menggunakan PM2

## Web Server

* [ ] Nginx berhasil di-install
* [ ] Nginx berhasil dikonfigurasi
* [ ] `nginx -t` berhasil
* [ ] Nginx berjalan
* [ ] Frontend dapat diakses melalui browser
* [ ] Frontend dapat berkomunikasi dengan Backend
* [ ] Backend dapat berkomunikasi dengan MySQL

---

# 39. Final Architecture

Setelah seluruh deployment selesai, arsitektur menjadi:

```text
                         USER / BROWSER
                               |
                               | HTTP
                               v
                    +----------------------+
                    |    GATEWAY SERVER     |
                    |                      |
                    |       NGINX          |
                    |          |           |
                    |          v           |
                    |      FRONTEND        |
                    |       PM2            |
                    |          |           |
                    |          v           |
                    |      BACKEND         |
                    |       PM2            |
                    +----------+-----------+
                               |
                               | MySQL :3306
                               v
                    +----------------------+
                    |     APP SERVER       |
                    |                      |
                    |       MYSQL          |
                    |                      |
                    |      database:       |
                    |         demo         |
                    |                      |
                    |       table:         |
                    |      transaction     |
                    +----------------------+

                         ^
                         |
                    MySQL Client
                         |
                         |
                   LOCAL COMPUTER
```

---

# 40. Conclusion

Deployment Wayshub telah mencakup:

1. App Server untuk MySQL Database.
2. Gateway Server untuk Backend, Frontend, dan Nginx.
3. User baru pada setiap server.
4. SSH Key Authentication tanpa password.
5. MySQL Secure Installation.
6. MySQL Root Password.
7. Database User.
8. Database `demo`.
9. Table `transaction`.
10. Role `admin` dan `guest`.
11. RBAC untuk Admin dan Guest.
12. Remote MySQL Access.
13. Wayshub Backend menggunakan Node.js 14.
14. Sequelize Migration.
15. Backend menggunakan PM2.
16. Wayshub Frontend menggunakan Node.js 14.
17. Frontend menggunakan PM2.
18. Nginx sebagai Web Server.
19. Testing keseluruhan deployment.
20. Dokumentasi screenshot setiap tahap penting.

---

## Notes

Semua nilai berikut harus disesuaikan dengan environment sebenarnya:

```text
App Server IP
Gateway Server IP
MySQL username
MySQL password
MySQL root password
Backend port
Frontend port
Backend repository
Frontend repository
API URL
Nginx configuration
```

Jangan memasukkan password asli ke repository GitHub atau repository publik.
