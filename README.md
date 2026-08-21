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
