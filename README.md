# Task 1
## Konsep DevOps
DevOps adalah gabungan dari kultur kerja, praktik, dan tools yang menyatukan tim pengembang (Development) dan tim operasional (Operations).
##  Langkah Instalasi Ubuntu Server 22.04 LTS (VirtualBox) & Konfigurasi IP Static (.208)
### 1. Persiapan Virtual Machine (VirtualBox)
1. Buka VirtualBox, klik tombol **New**.
2. Isikan konfigurasi dasar:
   - **Name**: `Ubuntu-Server-22.04`
   - **Type**: `Linux`
   - **Version**: `Ubuntu (64-bit)`
3. Alokasikan Hardware:
   - **RAM**: Minimal `2048 MB` (2 GB)
   - **CPU**: Minimal `2 Core`
   - **Hard Disk**: Minimal `20 GB` (Dynamically allocated)
4. Konfigurasi Jaringan (**Sangat Penting**):
   - Buka **Settings** > **Network**.
   - Pada **Adapter 1**, ubah *Attached to* menjadi **Bridged Adapter**.
   - Select kartu jaringan (Wi-Fi / Ethernet) komputer host yang terhubung ke internet.
   > **Catatan**: Mode *Bridged Adapter* memungkinkan VM berada dalam satu subnet jaringan lokal yang sama dengan host, sehingga IP `.208` dapat diakses dari luar VM.
5. Muat file ISO:
   - Buka **Settings** > **Storage**.
   - Klik ikon disk kosong (*Empty*) di bawah *Controller: IDE*, lalu muat file ISO **Ubuntu Server 22.04 LTS**.

---

### 2. Memulai Proses Instalasi Ubuntu Server
1. Jalankan VM, pilih opsi **Try or Install Ubuntu Server**.
2. **Language**: Pilih bahasa pengantar instalasi (Rekomendasi: `English`).
3. **Installer Update**: Pilih `Continue without updating` (jika ada notifikasi pembaruan).
4. **Keyboard Configuration**: Biarkan default `English (US)`, lalu pilih **Done**.
5. **Type of Install**: Pilih opsi default `Ubuntu Server`, lalu **Done**.

---

### 3. Konfigurasi IP Address Statis (`xxx.xxx.xxx.208`)
Pada menu **Network connections**:
1. Pilih interface jaringan aktif Anda (misalnya `eth0` / `enp0s3`).
2. Tekan `Enter` pada nama interface > pilih **Edit IPv4**.
3. Ubah mode dari *Automatic (DHCP)* menjadi **Manual**.
4. Pengisian parameter jaringan (sesuaikan segmen IP subnet lokal Anda):
   - **Subnet**: `192.168.1.0/24` *(Contoh segmen lokal)*
   - **Address**: `192.168.1.208` *(Ganti segmen awal sesuai subnet Anda, misal: `10.0.0.208` / `172.16.0.208`)*
   - **Gateway**: `192.168.1.1`
   - **Name servers**: `8.8.8.8,1.1.1.1`
5. Pilih **Save**, lalu pilih **Done**.

---

### 4. Penyelesaian Setup Sistem
1. **Proxy & Mirror**:
   - Kosongkan Proxy jika tidak menggunakan proxy, pilih **Done**.
   - Biarkan Ubuntu Archive Mirror default, pilih **Done**.
2. **Storage Configuration**:
   - Centang **Use an entire disk** (gunakan default LVM), pilih **Done** > **Continue**.
3. **Profile Setup**:
   - Isi identitas pengguna:
     - *Your name*: `admin`
     - *Your server's name*: `ubuntu-server`
     - *Pick a username*: `ubuntu`
     - *Password*: `[Password Anda]`
4. **SSH Setup**:
   - Beri tanda centang `[X]` pada **Install OpenSSH server** untuk mengaktifkan akses remote via SSH.
5. **Featured Server Snaps**:
   - Langsung pilih **Done** (opsional).
6. **Reboot**:
   - Tunggu hingga proses instalasi selesai, lalu pilih **Reboot Now**.
   - Lepaskan file ISO (*Unmount ISO*) jika diminta, lalu tekan `Enter`.

---

## 🔍 Verifikasi Instalasi & Konfigurasi Jaringan

Setelah server menyala kembali dan Anda berhasil login menggunakan *username* & *password* yang dibuat:

### 1. Cek Alamat IP
Jalankan perintah:
```bash
ip a
```
> Pastikan interface jaringan menampilkan IP **`192.168.1.208`** (atau sesuai segmen `xxx.xxx.xxx.208`).

---

### 2. Pengujian Koneksi Internet (Ping)
Jalankan perintah pengujian koneksi:

* **Test IP Google:**
  ```bash
  ping -c 4 8.8.8.8
  ```
  *Output yang diharapkan:* `0% packet loss` dan menerima balasan `ICMP reply`.

* **Test Domain / DNS:**
  ```bash
  ping -c 4 google.com
  ```
  *Output yang diharapkan:* Server mampu menyelesaikan nama domain `google.com` ke IP address dan menerima balasan paket.

# Task 2
## 1. Diagram Jaringan Komputer (4 Devices)

### Spesifikasi Jaringan
- **IP Class**: Class C
- **CIDR Block**: `192.168.4.0/24`
- **Subnet Mask**: `255.255.255.0`
- **Network ID**: `192.168.4.0`
- **Broadcast ID**: `192.168.4.255`
- **Usable IP Range**: `192.168.4.1` – `192.168.4.254`

### Topologi Jaringan (Star Topology)

```text
                        +--------------------+
                        |   Switch / Router  |
                        |   (192.168.4.1)    |
                        +---------+----------+
                                  |
         +----------------+-------+-------+----------------+
         |                |               |                |
         v                v               v                v
  +--------------+ +--------------+ +--------------+ +--------------+
  |   PC-01      | |   PC-02      | |   PC-03      | |   PC-04      |
  |              | |              | |              | |              |
  | IP:          | | IP:          | | IP:          | | IP:          |
  | 192.168.4.10 | | 192.168.4.11 | | 192.168.4.12 | | 192.168.4.13 |
  | Mask:        | | Mask:        | | Mask:        | | Mask:        |
  | /24          | | /24          | | /24          | | /24          |
  +--------------+ +--------------+ +--------------+ +--------------+
```

### Tabel Konfigurasi Perangkat

| Nama Perangkat | Peran / Fungsi | IP Address | Subnet Mask | Default Gateway |
| :--- | :--- | :--- | :--- | :--- |
| **Router / Gateway** | `192.168.4.1` | `255.255.255.0` | N/A |
| **PC-01** | `192.168.4.10` | `255.255.255.0` | `192.168.4.1` |
| **PC-02** | `192.168.4.11` | `255.255.255.0` | `192.168.4.1` |
| **PC-03** | `192.168.4.12` | `255.255.255.0` | `192.168.4.1` |
| **PC-04** | `192.168.4.13` | `255.255.255.0` | `192.168.4.1` |

---

## 2. Perbedaan SH (Shell) dan BASH (Bourne-Again Shell)

| Fitur / Karakteristik | SH (Bourne Shell) | BASH (Bourne-Again Shell) |
| :--- | :--- | :--- |
| **Pengertian** | Shell standar UNIX asli yang dikembangkan oleh Stephen Bourne di Bell Labs (1979). | Pengembangan/superset modern dari `sh` yang dikembangkan oleh Brian Fox untuk GNU Project (1989). |
| **Lokasi Biner** | Biasanya berada di `/bin/sh` (pada OS modern sering berupa *symlink* ke `dash` atau `bash`). | Terletak di `/bin/bash` atau `/usr/bin/bash`. |
| **Fitur Sintaks** | Sangat minimalis, berfokus pada portabilitas dan kesederhanaan standar POSIX. | Memiliki fitur sintaks lanjutan (array, manipulasi string, ekspresi reguler, dll). |
| **Fitur Interaktif** | Tidak ada *command history*, *auto-completion* (tabbing) sangat terbatas atau tidak ada. | Mendukung *command completion* (Tab), *history search* (Ctrl+R), dan alias. |
| **Array & Loop** | Tidak mendukung array secara *native*. | Mendukung *one-dimensional indexing* dan *associative arrays*. |
| **Pengujian Kondisi** | Hanya mendukung sintaks standar `[ condition ]`. | Mendukung `[[ condition ]]` yang lebih aman dan fleksibel. |
| **Skrip Prompt** | Menggunakan `#!/bin/sh` | Menggunakan `#!/bin/bash` |

---

## 3. Dokumentasi Kumpulan Perintah Linux (Linux Command Reference)

Berikut dokumentasi perintah Linux yang terbagi berdasarkan kategori fungsionalitasnya:

### A. Navigasi & Manajemen File/Direktori
- `ls -la`: Menampilkan seluruh file dan direktori termasuk yang tersembunyi (*dotfiles*) beserta detail hak aksesnya.
- `pwd`: Menampilkan direktori kerja saat ini (*Print Working Directory*).
- `cd /path/to/dir`: Pindah ke direktori tujuan.
- `mkdir -p dir1/dir2`: Membuat direktori baru (secara rekursif jika belum ada parent directory).
- `cp -r source/ destination/`: Menyalin file/folder secara rekursif.
- `mv file1.txt /new/location/`: Memindahkan atau mengubah nama file/folder.
- `rm -rf dir_name`: Menghapus file/direktori secara paksa dan rekursif.
- `touch filename`: Membuat file kosong baru atau memperbarui timestamp file.
- `tree`: Menampilkan struktur direktori dalam bentuk pohon grafis.

### B. Pemrosesan Teks & Pencarian File
- `cat file.txt`: Menampilkan seluruh isi file ke terminal.
- `less file.txt` / `more file.txt`: Menampilkan isi file besar halaman per halaman.
- `head -n 20 file.txt` / `tail -n 20 file.txt`: Menampilkan 20 baris pertama atau terakhir dari sebuah file.
- `grep -rnw '/path/' -e 'search_term'`: Mencari string/teks tertentu secara rekursif di seluruh file dalam direktori.
- `find /path -name "*.log" -type f`: Mencari file berdasarkan nama, tipe, ukuran, atau waktu modifikasi.
- `awk '{print $1}' file.txt`: Bahasa pemrosesan teks untuk manipulasi kolom data.
- `sed -i 's/old/new/g' file.txt`: Stream editor untuk mengganti teks langsung di dalam file.
- `wc -l file.txt`: Menghitung jumlah baris, kata, atau karakter dalam file.

### C. Manajemen Hak Akses & Pengguna (Permissions & Users)
- `chmod 755 script.sh`: Mengubah hak akses file (Read, Write, Execute untuk User/Group/Others).
- `chown user:group filename`: Mengubah pemilik (*owner*) dan grup dari sebuah file/direktori.
- `useradd -m -s /bin/bash newuser`: Membuat pengguna baru beserta direktori home dan shell default.
- `usermod -aG sudo newuser`: Menambahkan pengguna ke grup tertentu (misalnya grup sudoers).
- `passwd username`: Mengubah kata sandi pengguna.
- `visudo`: Mengedit file konfigurasi `/etc/sudoers` dengan aman.

### D. Manajemen Proses, Memori, & Sistem
- `ps aux`: Menampilkan seluruh proses aktif di sistem.
- `top` / `htop`: Monitoring penggunaan CPU, RAM, dan proses sistem secara *real-time* (interaktif).
- `kill -9 <PID>` / `killall <process_name>`: Menghentikan proses berdasarkan PID atau nama proses secara paksa.
- `free -h`: Menampilkan penggunaan memori RAM dan Swap dalam format *human-readable*.
- `df -h`: Menampilkan penggunaan ruang disk (*Disk Free*) pada filesystem.
- `du -sh /path/to/dir`: Menampilkan ukuran total penggunaan disk oleh suatu folder (*Disk Usage*).
- `systemctl status/start/stop/restart/enable <service>`: Mengelola *systemd service*/daemons.
- `journalctl -u service_name -f`: Melihat log sistem/service secara *real-time*.

### E. Jaringan & Komunikasi (Networking)
- `ip a` / `ip route`: Menampilkan informasi interface jaringan dan tabel routing.
- `ping -c 4 google.com`: Menguji konektivitas ke host/IP tujuan.
- `netstat -tulnp` / `ss -tulnp`: Menampilkan port jaringan yang sedang terbuka/listening beserta PID prosesnya.
- `curl -I https://example.com`: Mengirim request HTTP/HTTPS dan menampilkan header respon server.
- `wget https://domain.com/file.zip`: Mengunduh file langsung dari web melalui terminal.
- `traceroute 8.8.8.8` / `mtr 8.8.8.8`: Menelusuri rute/hop jaringan menuju host tujuan.
- `dig domain.com` / `nslookup domain.com`: Melakukan kueri DNS lookup untuk menganalisis record domain.
- `ssh user@host -p 22`: Melakukan koneksi remote aman via Secure Shell.
- `scp -P 22 file.txt user@host:/path/`: Menyalin file antar host jaringan secara aman via SSH.
- `rsync -avzP source/ user@host:/destination/`: Sinkronisasi data antar direktori/server efisien dan dapat di-resume.

### F. Perintah Linux Lanjutan & Tools Tambahan (Advanced/Pro Tools) 🔥
*(Command lanjutan diluar materi dasar)*

- `tmux` / `screen`: Terminal multiplexer yang memungkinkan Anda menjalankan banyak sesi terminal dalam satu jendela dan menjaga proses tetap berjalan di background saat koneksi SSH terputus.
- `nc -zv <host> <port>` (*Netcat*): Alat Swiss-Army Knife untuk jaringan; digunakan untuk melakukan scan/test konektivitas port TCP/UDP spesifik.
- `tcpdump -i eth0 -n vlan`: Sniffer paket jaringan langsung dari terminal untuk analisa lalu lintas data.
- `nmap -sS -p 1-1024 <IP>`: Network mapper & port scanner mendalam untuk memindai port terbuka dan deteksi OS.
- `strace -p <PID>`: Melacak *system calls* dan signal yang diterima oleh suatu proses Linux untuk debugging masalah aplikasi.
- `lsof -i :80`: Menampilkan daftar file dan koneksi socket yang sedang dibuka oleh port atau proses tertentu.
- `iotop`: Monitoring I/O pembacaan/penulisan harddisk per proses secara *real-time*.
- `lsblk -f`: Menampilkan struktur dan hierarki blok penyimpanan (disk & partisi) beserta sistem filenya.
- `uptime`: Menampilkan berapa lama sistem telah menyala beserta beban rata-rata (*load average*).
- `dmesg -T`: Menampilkan pesan kernel/hardware buffer sistem dengan timestamp terformat.

# Task 3
## 1. Akses Server Menggunakan Terminal

Untuk terhubung ke Ubuntu Server via SSH dari komputer lokal (*Windows Terminal / PowerShell / Linux Terminal*):

```bash
ssh username@192.168.4.208
```
*(Ganti `username` dan alamat IP sesuai dengan konfigurasi server Anda).*

---

## 2. Konfigurasi SSH Key-Based Authentication

### Langkah A: Buat SSH Key Pair di Komputer Lokal (Client)
Jalankan perintah berikut di terminal komputer lokal Anda:
```bash
ssh-keygen -t ed25519 -C "admin-key"
```
* Tekan **Enter** untuk menyimpan di lokasi default (`~/.ssh/id_ed25519`).
* Masukkan passphrase jika ingin keamanan ekstra (opsional).

### Langkah B: Salin Public Key ke Server
Gunakan perintah `ssh-copy-id` untuk mengirimkan Public Key ke server:
```bash
ssh-copy-id username@192.168.4.208
```

### Langkah C: Matikan Otentikasi Password di Server (Opsional/Rekomendasi Keamanan)
Login ke server, lalu buka dan edit file konfigurasi SSH Daemon (`sshd_config`):
```bash
sudo nano /etc/ssh/sshd_config
```
Ubah/pastikan baris parameter berikut diset sebagai berikut:
```ini
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
PermitRootLogin no
```
Simpan file (`Ctrl+O` lalu `Enter`), lalu keluar (`Ctrl+X`).

Terapkan konfigurasi baru dengan merestart service SSH:
```bash
sudo systemctl restart ssh
```

---

## 3. Step-by-Step Penggunaan Text Manipulation (`echo`, `cat`, `grep`, `sed`)

Berikut alur praktikum manipulasi teks di terminal Linux:

### Langkah 1: Membuat & Menambah Teks dengan `echo`
* **Membuat file baru:**
  ```bash
  echo "Server status: ACTIVE" > app.log
  ```
* **Menambahkan baris baru ke file tanpa menimpa (*append*):**
  ```bash
  echo "Error 404: Page not found on /api/v1/user" >> app.log
  echo "INFO: User admin logged in successfully" >> app.log
  echo "Error 500: Database connection timeout on /api/v1/db" >> app.log
  ```

### Langkah 2: Menampilkan Isi File dengan `cat`
* **Menampilkan seluruh isi file:**
  ```bash
  cat app.log
  ```
* **Menampilkan isi file lengkap dengan nomor baris:**
  ```bash
  cat -n app.log
  ```

### Langkah 3: Mencari & Memfilter Teks dengan `grep`
* **Mencari kata tertentu (misal: "Error"):**
  ```bash
  grep "Error" app.log
  ```
* **Mencari teks tanpa memedulikan huruf besar/kecil (*case-insensitive*):**
  ```bash
  grep -i "info" app.log
  ```
* **Mencari baris yang TIDAK mengandung kata tertentu (*invert match*):**
  ```bash
  grep -v "Error" app.log
  ```

### Langkah 4: Mengubah/Mengganti Teks dengan `sed`
* **Mengganti kata di layar saja (tanpa mengubah file asli):**
  ```bash
  sed 's/ACTIVE/RUNNING/' app.log
  ```
* **Mengganti kata dan langsung menyimpan ke file asli (*In-place editing*):**
  ```bash
  sed -i 's/Error 500/CRITICAL 500/g' app.log
  ```
* **Menghapus baris tertentu (misal: menghapus baris yang memuat kata "404"):**
  ```bash
  sed -i '/404/d' app.log
  ```

---

## 4. Konfigurasi UFW (Uncomplicated Firewall)

Aktifkan firewall di server dan buka port-port berikut: `22` (SSH), `80` (HTTP), `443` (HTTPS), `3000` (Node.js/React), `5000` (Flask/Python), dan `6969` (Custom App).

### Langkah A: Izinkan Port-Port yang Ditentukan
Jalankan perintah berikut satu per satu:

```bash
# Izinkan Port 22 (SSH - Wajib agar koneksi remote tidak terputus)
sudo ufw allow 22/tcp comment 'SSH Remote Access'

# Izinkan Port 80 & 443 (Web Server)
sudo ufw allow 80/tcp comment 'HTTP Web'
sudo ufw allow 443/tcp comment 'HTTPS Secure Web'

# Izinkan Port Aplikasi (3000, 5000, 6969)
sudo ufw allow 3000/tcp comment 'NodeJS / Dev App'
sudo ufw allow 5000/tcp comment 'Flask / Python App'
sudo ufw allow 6969/tcp comment 'Custom Service'
```

### Langkah B: Aktifkan UFW Firewall
```bash
sudo ufw enable
```
*Tekan `y` saat muncul konfirmasi.*

### Langkah C: Cek Status & Verifikasi Rule Firewall
```bash
sudo ufw status numbered
```

**Output yang diharapkan:**
```text
Status: active

To                         Action      From
--                         ------      ----
[ 1] 22/tcp                     ALLOW IN    Anywhere                   # SSH Remote Access
[ 2] 80/tcp                     ALLOW IN    Anywhere                   # HTTP Web
[ 3] 443/tcp                    ALLOW IN    Anywhere                   # HTTPS Secure Web
[ 4] 3000/tcp                   ALLOW IN    Anywhere                   # NodeJS / Dev App
[ 5] 5000/tcp                   ALLOW IN    Anywhere                   # Flask / Python App
[ 6] 6969/tcp                   ALLOW IN    Anywhere                   # Custom Service
```

# Task 4

## 1. Penjelasan Singkat tentang Git

**Git** adalah sistem pengontrol versi terdistribusi (*Distributed Version Control System / DVCS*) yang digunakan untuk melacak perubahan pada kode sumber atau berkas selama proses pengembangan perangkat lunak. 

* **Kelebihan Utama Git**:
  - **Terdistribusi**: Setiap *developer* memiliki salinan penuh dari seluruh riwayat proyek secara lokal.
  - **Branching & Merging Cepat**: Memudahkan pengerjaan fitur baru tanpa mengganggu kode utama (*main/master*).
  - **Lacak Perubahan**: Memungkinkan rollback ke versi sebelumnya jika terjadi bug atau kesalahan.

---

## 2 & 3. Membuat & Mengelola Repositori via Terminal

### Langkah A: Inisialisasi Repositori Lokal
Buka terminal Anda dan jalankan perintah berikut (ganti `<nama>` dengan nama Anda):

```bash
# 1. Buat direktori proyek baru
mkdir devops26-dumbways-nama
cd devops26-dumbways-nama

# 2. Inisialisasi Git pada direktori
git init
```

### Langkah B: Membuat 3 Berkas Teks
Buat 3 berkas teks di dalam direktori repositori:

```bash
# Membuat file 1
echo "Ini adalah file konfigurasi utama proyek DevOps." > config.txt

# Membuat file 2
echo "Dokumentasi aplikasi dan panduan instalasi." > README.md

# Membuat file 3
echo "Daftar repositori & dependencies aplikasi." > app.txt
```

### Langkah C: Menambahkan & Melakukan Commit Berkas
```bash
# 1. Cek status repositori (akan menampilkan file untracked)
git status

# 2. Tambahkan ketiga file ke Staging Area
git add .

# 3. Simpan perubahan ke dalam riwayat Git (Commit)
git commit -m "feat: initial commit add 3 text files"
```

### Langkah D: Menghubungkan ke GitHub Remote & Push
1. Buat repositori baru di GitHub dengan nama **`devops26-dumbways-<nama>`** (kosongkan opsi *Initialize with README*).
2. Hubungkan repositori lokal ke GitHub dan unggah (*push*) kodenya:

```bash
# Ubah nama branch utama ke main
git branch -M main

# Hubungkan ke remote GitHub (ganti username dengan username GitHub Anda)
git remote add origin https://github.com/username/devops26-dumbways-nama.git

# Unggah seluruh perubahan ke GitHub
git push -u origin main
```

---

## 4. Cara Mencari & Memantau Perubahan Teks (Diff) di GitHub

Berikut cara melihat riwayat dan detail perubahan isi file di GitHub:

### A. Melihat Seluruh Perubahan Berdasarkan Commit (*Commit History*)
1. Buka halaman repositori **`devops26-dumbways-<nama>`** di GitHub.
2. Klik tombol **Commits** (ikon jam di atas daftar file).
3. Klik pada **Pesan Commit** (contoh: `feat: initial commit...`).
4. GitHub akan menampilkan tampilan *Diff*:
   - Teks berwarna **Hijau (`+`)**: Penambahan teks baru.
   - Teks berwarna **Merah (`-`)**: Teks yang dihapus atau diubah.

### B. Melihat Riwayat Perubahan pada File Spesifik (*File History*)
1. Buka file yang ingin dicek (misalnya `README.md`).
2. Klik tombol **History** di pojok kanan atas tampilan file.
3. Anda akan melihat daftar commit khusus yang pernah mengubah file tersebut.

### C. Menggunakan Fitur **Blame** untuk Mengetahui Pembuat Perubahan
1. Buka file yang dituju (misal: `config.txt`).
2. Klik tombol **Blame**.
3. GitHub akan menampilkan siapa yang menulis setiap baris kode, kapan perubahan dilakukan, beserta id commit terkait.

# Task 5
## Ringkasan Deployment & Port

| Aplikasi | Runtime / Framework | Port | Deskripsi / Output | Status UFW |
| :--- | :--- | :--- | :--- | :--- |
| **Wayshub Frontend** | NodeJS (v12 / NVM) | `3000` | Aplikasi Frontend Wayshub | `ALLOW` |
| **Python Web App** | Python 3 (Flask) | `5000` | Menampilkan Teks: Nama Anda | `ALLOW` |
| **Golang Web App** | Go (Golang) | `6969` | Menampilkan Teks: "Golang geming!" | `ALLOW` |

---

## 1. Konfigurasi UFW Firewall

Sebelum atau setelah aplikasi dijalankan, pastikan UFW Firewall aktif dan membuka port-port yang diperlukan agar semua aplikasi dapat diakses melalui browser client.

```bash
# 1. Izinkan Port SSH agar koneksi remote tidak terputus
sudo ufw allow 22/tcp comment 'SSH'

# 2. Izinkan Port Aplikasi (3000, 5000, 6969)
sudo ufw allow 3000/tcp comment 'NodeJS Wayshub'
sudo ufw allow 5000/tcp comment 'Python App'
sudo ufw allow 6969/tcp comment 'Golang App'

# 3. Aktifkan Firewall
sudo ufw enable

# 4. Verifikasi status port
sudo ufw status numbered
```

---

## 2. Deploy Application 1: NodeJS (Wayshub Frontend)

Aplikasi ini memerlukan versi NodeJS lama (v10 / v12). Kita akan menggunakan **NVM (Node Version Manager)** untuk mengisolasi versi NodeJS.

### Langkah-langkah Deployment:
```bash
# 1. Install NVM (Node Version Manager)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh | bash
source ~/.bashrc

# 2. Install dan Gunakan NodeJS v12 (atau v10)
nvm install 12
nvm use 12

# 3. Clone Repositori Wayshub Frontend
git clone https://github.com/dumbwaysdev/wayshub-frontend.git
cd wayshub-frontend

# 4. Install Dependencies & Build
npm install

# 5. Jalankan Aplikasi di Port 3000 (Gunakan PM2 agar berjalan di Background)
npm install -g pm2
pm2 start npm --name "wayshub-frontend" -- start -- -p 3000
pm2 save
```

* **Pengujian**: Buka browser dan akses `http://192.168.4.208:3000`

---

## 3. Deploy Application 2: Python (Flask App)

Aplikasi Python Sederhana yang menampilkan nama Anda dan berjalan di port 5000.

### Langkah-langkah Deployment:
```bash
# 1. Install Python & Virtual Environment
sudo apt update && sudo apt install -y python3 python3-pip python3-venv

# 2. Buat Direktori Proyek
mkdir -p ~/apps/python-app && cd ~/apps/python-app

# 3. Buat file app.py
cat << 'EOF' > app.py
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return "<h1>Hello, Nama Saya: [NAMA KALIAN]</h1>"

if __name__ == '__main__':
    # Running di port 5000 pada semua interface (0.0.0.0)
    app.run(host='0.0.0.0', port=5000)
EOF

# 4. Buat Virtual Environment & Install Flask
python3 -m venv venv
source venv/bin/activate
pip install flask

# 5. Jalankan Python App di Background (PM2)
pm2 start "python3 app.py" --name "python-app"
```

* **Pengujian**: Buka browser dan akses `http://192.168.4.208:5000`

---

## 4. Deploy Application 3: Golang App

Aplikasi Golang sederhana yang menampilkan teks `"Golang geming!"`.

### Langkah-langkah Deployment:
```bash
# 1. Install Golang Compiler
sudo apt install -y golang-go

# 2. Buat Direktori Proyek
mkdir -p ~/apps/golang-app && cd ~/apps/golang-app

# 3. Buat file main.go
cat << 'EOF' > main.go
package main

import (
    "fmt"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Golang geming!")
}

func main() {
    http.HandleFunc("/", handler)
    fmt.Println("Server Running on port 6969...")
    http.ListenAndServe(":6969", nil)
}
EOF

# 4. Build Binary File Golang
go build -o golang-app main.go

# 5. Jalankan Binary Golang dengan PM2
pm2 start ./golang-app --name "golang-app"
```

* **Pengujian**: Buka browser dan akses `http://192.168.4.208:6969`

---

## Verifikasi Akhir Seluruh Service

Jalankan perintah berikut untuk memastikan ketiga aplikasi berjalan stabil di background:

```bash
# Cek status PM2 Process List
pm2 list

# Cek listener port aktif
ss -tulnp | grep -E '3000|5000|6969'
```

# Task 6
## 1. Arsitektur & Cara Kerja Reverse Proxy

### Topologi / Diagram Struktur Web Server

```text
                                +-----------------------------------+
                                |          Ubuntu Server            |
                                |          (192.168.4.208)          |
                                |                                   |
[ User / Browser ]              |   +---------------------------+   |
        |                       |   |     Nginx Reverse Proxy   |   |
        |--- ( HTTP Port 80 ) ----->|   |     (Port 80 / 443)     |   |
        |                       |   +-------------+-------------+   |
        |                       |                 |                 |
        v                       |     +-----------+-----------+     |
  http://bule.xyz --------------+---->|  Proxy Pass Rules     |     |
                                |     +-----+-----+-----+-----+     |
                                |           |     |     |           |
                                |  +--------+     |     +--------+  |
                                |  v              v              v  |
                                | [Wayshub]   [Python]       [Golang]|
                                | Port 3000   Port 5000      Port 6969
                                +-----------------------------------+
```

### Cara Kerja Reverse Proxy
1. **Request dari Client**: Pengguna mengakses alamat domain (misalnya `http://bule.xyz` atau `http://app.bule.xyz`) melalui web browser.
2. **Penerimaan oleh Nginx**: Nginx yang bertindak sebagai Reverse Proxy menerima request masuk di **Port 80** (HTTP) atau **Port 443** (HTTPS).
3. **Pencocokan Host/Path**: Nginx memeriksa konfigurasi `server_name` atau `location` untuk menentukan aplikasi tujuan.
4. **Proxy Pass (Internal Redirection)**: Nginx meneruskan (*forward*) request tersebut ke port aplikasi internal yang sesuai (`127.0.0.1:3000` untuk Wayshub, `127.0.0.1:5000` untuk Python, `127.0.0.1:6969` untuk Golang).
5. **Respons ke Client**: Aplikasi memproses request dan mengirimkan respons kembali ke Nginx, lalu Nginx menyampaikannya kembali ke browser pengguna. Client tidak mengetahui port internal tempat aplikasi sebenarnya berjalan.

---

## 2. Implementasi Reverse Proxy dengan Nginx

### Langkah A: Instalasi Nginx Web Server
Jalankan perintah berikut pada Ubuntu Server Anda:

```bash
sudo apt update
sudo apt install -y nginx
sudo systemctl enable --now nginx
```

---

### Langkah B: Konfigurasi Virtual Host / Reverse Proxy

Buat file konfigurasi server block baru untuk domain Anda (contoh menggunakan domain `bule.xyz`):

```bash
sudo nano /etc/nginx/sites-available/bule.xyz
```

Isikan konfigurasi Nginx di bawah ini:

```nginx
# 1. Konfigurasi Domain Utama (Wayshub Frontend - Port 3000)
server {
    listen 80;
    server_name bule.xyz www.bule.xyz;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}

# 2. Konfigurasi Subdomain Python App (Port 5000)
server {
    listen 80;
    server_name python.bule.xyz;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# 3. Konfigurasi Subdomain Golang App (Port 6969)
server {
    listen 80;
    server_name golang.bule.xyz;

    location / {
        proxy_pass http://127.0.0.1:6969;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

### Langkah C: Aktivasi Konfigurasi & Reload Nginx

1. **Buat Symlink** ke direktori `sites-enabled`:
   ```bash
   sudo ln -s /etc/nginx/sites-available/bule.xyz /etc/nginx/sites-enabled/
   ```

2. **Uji Sintaks Konfigurasi**:
   ```bash
   sudo nginx -t
   ```
   *Pastikan muncul pesan `syntax is ok` dan `test is successful`.*

3. **Reload Nginx Service**:
   ```bash
   sudo systemctl reload nginx
   ```

---

### Langkah D: Setting Local Local DNS / Hosts File (Untuk Pengujian Local Domain)

Jika domain `bule.xyz` belum dibeli secara publik, petakan IP server (`192.168.4.208`) ke nama domain pada komputer client Anda:

* **Windows**: Edit file `C:\Windows\System32\drivers\etc\hosts` (Buka dengan Administrator)
* **Linux/MacOS**: Edit file `/etc/hosts`

Tambahkan baris berikut:
```text
192.168.4.208    bule.xyz www.bule.xyz
192.168.4.208    python.bule.xyz
192.168.4.208    golang.bule.xyz
```

---

### Langkah E: Update UFW Firewall Rules

Karena trafik HTTP kini dipusatkan melalui Nginx pada **Port 80**, pastikan port 80 telah diizinkan pada UFW Firewall:

```bash
# Izinkan HTTP & HTTPS Nginx Profile
sudo ufw allow 'Nginx Full'

# Aktifkan UFW
sudo ufw enable

# Cek Status Firewall
sudo ufw status
```

---

## 🔍 Pengujian Reverse Proxy melalui Browser

Buka browser komputer client dan akses domain berikut tanpa menyebutkan nomor port:

* **Wayshub Frontend**: `http://bule.xyz`
* **Python App**: `http://python.bule.xyz`
* **Golang App**: `http://golang.bule.xyz`
