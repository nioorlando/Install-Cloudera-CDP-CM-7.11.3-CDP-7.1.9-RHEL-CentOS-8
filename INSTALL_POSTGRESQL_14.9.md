# 📘 Instalasi & Konfigurasi PostgreSQL 14.9 (Build from Source)

Panduan ini berisi langkah-langkah instalasi PostgreSQL 14.9 dari source, konfigurasi dasar (logging + akses), serta pembuatan user & database untuk aplikasi.  
**Catatan placeholder:** isi nilai yang diapit tanda kurung, misal `(passwordnya)`, `(username)`, `(alamat_ip_server)` sesuai kebutuhanmu.

---

## 1. Download & Ekstrak PostgreSQL 14.9

```bash
cd /opt
wget https://ftp.postgresql.org/pub/source/v14.9/postgresql-14.9.tar.gz
tar -xvzf postgresql-14.9.tar.gz
cd postgresql-14.9
```

---

## 2. Instalasi Dependensi

```bash
yum groupinstall -y "Development Tools"
yum install -y readline-devel zlib-devel
```

---

## 3. Kompilasi & Instalasi PostgreSQL

```bash
./configure --prefix=/opt/postgresql-14.9
make -j$(nproc)
make install
```

> Opsi: tambahkan bin PostgreSQL ke PATH (agar `psql` dll mudah dipanggil)
```bash
echo 'export PATH=/opt/postgresql-14.9/bin:$PATH' > /etc/profile.d/postgres14.sh
source /etc/profile.d/postgres14.sh
```

---

## 4. Membuat User & Direktori Data

```bash
# Membuat system user untuk PostgreSQL
useradd postgres

# Direktori instalasi & data
mkdir -p /opt/postgresql-14.9/data
chown -R postgres:postgres /opt/postgresql-14.9
chmod 700 /opt/postgresql-14.9/data
```

---

## 5. Inisialisasi Database

```bash
# Switch ke user postgres
su - postgres

# Inisialisasi cluster database
/opt/postgresql-14.9/bin/initdb -D /opt/postgresql-14.9/data
```

---

## 6. Konfigurasi PostgreSQL

### a) `postgresql.conf` (aktifkan logging & listen_addresses)

```bash
vi /opt/postgresql-14.9/data/postgresql.conf
```

Tambahkan/ubah parameter berikut:
```ini
# Logging
logging_collector = on
log_directory = 'log'
log_min_duration_statement = 1000   # Log query > 1 detik

# (Opsional) Dengarkan di semua alamat (untuk akses remote)
listen_addresses = '*'
```

### b) `pg_hba.conf` (atur akses klien)

```bash
vi /opt/postgresql-14.9/data/pg_hba.conf
```

Tambahkan baris berikut (akses dari mana saja dengan password). Sesuaikan CIDR jika perlu.
```ini
host    all     all     0.0.0.0/0       md5
host    all     all     ::/0            md5
```

> Setelah mengubah konfigurasi, **reload/restart** service agar aktif.

---

## 7. Menjalankan PostgreSQL

### Opsi A — Manual

```bash
/opt/postgresql-14.9/bin/pg_ctl -D /opt/postgresql-14.9/data -l /opt/postgresql-14.9/data/postgresql.log start
```

> Kekurangan: kalau mati/boot ulang, perlu start manual lagi.

### Opsi B — Systemd (disarankan)

Buat unit service:
```bash
vi /etc/systemd/system/postgresql-14.9.service
```

Isi dengan:
```ini
[Unit]
Description=PostgreSQL 14.9 Database Server
After=network.target

[Service]
User=postgres
Group=postgres
ExecStart=/opt/postgresql-14.9/bin/postgres -D /opt/postgresql-14.9/data
ExecStop=/opt/postgresql-14.9/bin/pg_ctl -D /opt/postgresql-14.9/data stop
Restart=always

[Install]
WantedBy=multi-user.target
```

Reload dan aktifkan service:
```bash
systemctl daemon-reload
systemctl enable postgresql-14.9
systemctl start postgresql-14.9
```

---

## 8. Mengecek Status PostgreSQL

```bash
systemctl status postgresql-14.9
```

---

## 9. Mengakses PostgreSQL (psql)

```bash
# Login sebagai user postgres (superuser)
su - postgres

# Masuk ke psql (akan langsung ke DB default "postgres")
/opt/postgresql-14.9/bin/psql
```

> Jika akses dari remote atau butuh host/port spesifik:
```bash
psql -U (username) -d (namadb) -h (alamat_ip_server) -p 5432
# lalu masukkan (passwordnya) saat diminta
```

---

## 10. Verifikasi Logging

Di dalam `psql`:
```sql
SHOW log_directory;
SHOW logging_collector;
```

Cek file log di OS:
```bash
ls -lah /opt/postgresql-14.9/data/log/
tail -n 100 /opt/postgresql-14.9/data/log/postgresql-YYYY-MM-DD.log
```

---

## 11. Membuat User & Database untuk Pengujian

Masuk `psql` sebagai superuser `postgres`:
```bash
su - postgres
psql
```

Buat user & DB uji:
```sql
CREATE USER dbuser WITH PASSWORD '(passwordnya)';
CREATE DATABASE testdb OWNER dbuser;
```

Cek daftar DB & role:
```sql
\l
\du
```

---

## 12. Memberikan Hak Akses (contoh umum)

```sql
-- Hak penuh pada database
GRANT ALL PRIVILEGES ON DATABASE testdb TO dbuser;

-- Atau granular (umum dipakai)
GRANT CONNECT ON DATABASE testdb TO dbuser;
GRANT USAGE ON SCHEMA public TO dbuser;
GRANT SELECT, INSERT ON ALL TABLES IN SCHEMA public TO dbuser;

-- Agar tabel yang dibuat di masa depan juga otomatis ter-grant:
ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT SELECT, INSERT ON TABLES TO dbuser;
```

---

## 13. Menguji Koneksi sebagai User Baru

```bash
psql -U dbuser -d testdb -h 127.0.0.1 -p 5432
# masukkan (passwordnya) saat prompt
```

Uji query:
```sql
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100) UNIQUE
);

INSERT INTO customers (name, email) VALUES ('Budi Santoso', 'budi@example.com');
SELECT * FROM customers;
```

---

## 14. Menguji Logging & Monitoring

```bash
# Lihat streaming log harian
tail -f /opt/postgresql-14.9/data/log/postgresql-YYYY-MM-DD.log

# Cari query yang terekam durasinya
grep "duration" /opt/postgresql-14.9/data/log/postgresql-YYYY-MM-DD.log
```

---

## 15. Membuat User/Role & Database untuk Aplikasi (Cloudera dsb.)

Jalankan di `psql` sebagai superuser (`postgres`):
```sql
-- Buat user/role (isi password dengan (passwordnya) masing-masing)
CREATE ROLE scm            WITH LOGIN PASSWORD '(passwordnya)';
CREATE ROLE rman           WITH LOGIN PASSWORD '(passwordnya)';
CREATE ROLE rangeradmin    WITH LOGIN PASSWORD '(passwordnya)';
CREATE ROLE rangerkms      WITH LOGIN PASSWORD '(passwordnya)';
CREATE ROLE hue            WITH LOGIN PASSWORD '(passwordnya)';
CREATE ROLE hive           WITH LOGIN PASSWORD '(passwordnya)';
CREATE ROLE oozie          WITH LOGIN PASSWORD '(passwordnya)';
CREATE ROLE schemaregistry WITH LOGIN PASSWORD '(passwordnya)';
CREATE ROLE smm            WITH LOGIN PASSWORD '(passwordnya)';
CREATE ROLE yarn           WITH LOGIN PASSWORD '(passwordnya)';
CREATE ROLE kafka          WITH LOGIN PASSWORD 'kafka';
CREATE ROLE cmf            WITH LOGIN PASSWORD 'cmf';

-- Buat database dengan owner masing-masing
CREATE DATABASE scm            OWNER scm            ENCODING 'UTF8';
CREATE DATABASE rman           OWNER rman           ENCODING 'UTF8';
CREATE DATABASE rangeradmin    OWNER rangeradmin    ENCODING 'UTF8';
CREATE DATABASE rangerkms      OWNER rangerkms      ENCODING 'UTF8';
CREATE DATABASE hue            OWNER hue            ENCODING 'UTF8';
CREATE DATABASE hive           OWNER hive           ENCODING 'UTF8';
CREATE DATABASE oozie          OWNER oozie          ENCODING 'UTF8';
CREATE DATABASE schemaregistry OWNER schemaregistry ENCODING 'UTF8';
CREATE DATABASE smm            OWNER smm            ENCODING 'UTF8';
CREATE DATABASE yarn           OWNER yarn           ENCODING 'UTF8';

-- Grant akses penuh (umumnya redundant karena owner sudah punya, tapi eksplisitkan)
GRANT ALL PRIVILEGES ON DATABASE scm            TO scm;
GRANT ALL PRIVILEGES ON DATABASE rman           TO rman;
GRANT ALL PRIVILEGES ON DATABASE rangeradmin    TO rangeradmin;
GRANT ALL PRIVILEGES ON DATABASE rangerkms      TO rangerkms;
GRANT ALL PRIVILEGES ON DATABASE hue            TO hue;
GRANT ALL PRIVILEGES ON DATABASE hive           TO hive;
GRANT ALL PRIVILEGES ON DATABASE oozie          TO oozie;
GRANT ALL PRIVILEGES ON DATABASE schemaregistry TO schemaregistry;
GRANT ALL PRIVILEGES ON DATABASE smm            TO smm;
GRANT ALL PRIVILEGES ON DATABASE yarn           TO yarn;
GRANT ALL PRIVILEGES ON DATABASE kafka          TO kafka;
```
