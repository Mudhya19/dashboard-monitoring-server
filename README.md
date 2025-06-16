# Monitoring Menggunakan Prometheus dan Grafana

Dokumentasi langkah-langkah instalasi dan konfigurasi **Prometheus**, **Node Exporter**, dan **Grafana** untuk kebutuhan monitoring sistem/server.

---

## Daftar Isi

1. [Instalasi Prometheus dan Node Exporter](#1-instalasi-prometheus-dan-node-exporter)
2. [Konfigurasi User dan Direktori Prometheus](#2-konfigurasi-user-dan-direktori-prometheus)
3. [Konfigurasi Service Prometheus](#3-konfigurasi-service-prometheus)
4. [Instalasi dan Konfigurasi Node Exporter](#4-instalasi-dan-konfigurasi-node-exporter)
5. [Integrasi Node Exporter dengan Prometheus](#5-integrasi-node-exporter-dengan-prometheus)
6. [Menjalankan dan Mengecek Status Service](#6-menjalankan-dan-mengecek-status-service)
7. [Menjalankan Grafana](#7-menjalankan-grafana)

---

## 1. Instalasi Prometheus dan Node Exporter

### Unduh file:

```bash
# Node Exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz

# Prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.53.3/prometheus-2.53.3.linux-amd64.tar.gz
```

### Ekstrak file:

```bash
tar xvf node_exporter-*.tar.gz
tar xvf prometheus-*.tar.gz
```

---

## 2. Konfigurasi User dan Direktori Prometheus

```bash
# Tambah user dan group untuk Prometheus
sudo groupadd --system prometheus
sudo useradd --system -s /sbin/nologin -g prometheus prometheus

# Pindahkan file biner
sudo mv prometheus promtool /usr/local/bin/

# Buat direktori konfigurasi dan data
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus

# Atur kepemilikan
sudo chown -R prometheus:prometheus /var/lib/prometheus

# Pindahkan file konfigurasi
sudo mv consoles/ console_libraries/ prometheus.yml /etc/prometheus/
```

---

## 3. Konfigurasi Service Prometheus

### arahkan kedalam direktori /etc/Prometheus lalu membuka Edit konfigurasi `/etc/prometheus/prometheus.yml`

```yaml
# my global config
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
```

### Tambahkan service systemd:

```bash
sudo nano /etc/systemd/system/prometheus.service
```

Isi file:

```ini
[Unit]
Description=Prometheus
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus   --config.file /etc/prometheus/prometheus.yml   --storage.tsdb.path /var/lib/prometheus/   --web.console.templates=/etc/prometheus/consoles   --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

---

## 4. Instalasi dan Konfigurasi Node Exporter

```bash
# Jalankan node_exporter
./node_exporter

# Pindahkan ke direktori bin
sudo mv node_exporter /usr/local/bin/

#lokasi file binary node_exporter
which node_exporter
```

### Tambahkan service systemd:

```bash
sudo nano /etc/systemd/system/node-exporter.service
```

Isi file:

```ini
[Unit]
Description=Prometheus exporter for machine metrics

[Service]
Restart=always
User=prometheus
ExecStart=/usr/local/bin/node_exporter
ExecReload=/bin/kill -HUP $MAINPID
TimeoutStopSec=20s
SendSIGKILL=no

[Install]
WantedBy=multi-user.target
```

---

## 5. Integrasi Node Exporter dengan Prometheus

Edit kembali file `/etc/prometheus/prometheus.yml`:

```yaml
# my global config
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
  - job_name: "node-exporter"
    static_configs:
      - targets: ["localhost:9100"]
```

---

## 6. Menjalankan dan Mengecek Status Service

```bash
# Reload systemd
sudo systemctl daemon-reload

# Aktifkan dan cek Prometheus
sudo systemctl enable --now prometheus
sudo systemctl status prometheus

# Aktifkan dan cek Node Exporter
sudo systemctl enable --now node-exporter.service
sudo systemctl status node-exporter.service

# Cek port berjalan
sudo lsof -n -i | grep prometheus
sudo lsof -n -i | grep node

# Lihat IP server
hostname -I
```

---

## 7. Menjalankan Grafana

```bash
# Pastikan Grafana sudah terinstal (misalnya versi 9.4)

# Jalankan dan aktifkan service
sudo systemctl enable --now grafana-server.service
sudo systemctl status grafana-server.service

# Cek port Grafana (default 3000)
sudo lsof -n -P -i | grep grafana
```

---

### Referensi

- [Dokumentasi Prometheus](https://prometheus.io/docs/)
- [Dokumentasi Grafana](https://grafana.com/docs/)

## 8 Akses Dashboard Monitoring

Setelah semua proses instalasi dan konfigurasi selesai mengikuti langkah-langkah dalam README ini, Anda dapat mengakses layanan monitoring melalui browser dengan alamat berikut:

- **Grafana Dashboard** – visualisasi metrik sistem/server secara real-time:  
  👉 http://localhost:3000  
  (default username: `admin`, password: `admin` atau yang sudah Anda atur)

- **Prometheus Web UI** – menampilkan metrik, query (PromQL), dan status endpoint yang sedang dimonitor:  
  👉 http://localhost:9090

---

### 🧩 Setup Dashboard Grafana Sesuai Kebutuhan

Setelah berhasil login ke Grafana, Anda bisa melakukan setup dashboard untuk menampilkan metrik sesuai kebutuhan Anda:

1. **Menambahkan Data Source**:
   - Buka menu “⚙️ Configuration” → “Data Sources”.
   - Pilih **Prometheus**.
   - Masukkan URL Prometheus, misalnya `http://localhost:9090`, lalu klik **Save & Test**.

2. **Mengimpor Dashboard Siap Pakai**:
   - Buka menu “+ Create” → “Import”.
   - Masukkan **ID Dashboard dari Grafana.com**, contohnya:
     - **1860** untuk Node Exporter Full (dashboard sistem lengkap)
     - **11074** untuk Linux Server Dashboard
   - Pilih data source Prometheus, lalu klik **Import**.

3. **Membuat Dashboard Kustom Sendiri**:
   - Buka menu “+ Create” → “Dashboard”.
   - Tambahkan panel dan buat query PromQL sesuai kebutuhan, misalnya:
     - `node_cpu_seconds_total` untuk CPU
     - `node_memory_MemAvailable_bytes` untuk memori
     - `node_filesystem_size_bytes` untuk disk
     - `node_network_receive_bytes_total` untuk jaringan

4. **Mengatur Alert (Opsional)**:
   - Anda dapat membuat alert di Grafana berdasarkan metrik tertentu, misalnya saat CPU usage > 90% selama 5 menit.
   - Alert dapat dikirim ke Telegram, Email, Slack, dll jika alerting sudah diaktifkan dan dikonfigurasi.

---

### 🛡️ Pastikan Port Terbuka

Pastikan port default Prometheus (`9090`) dan Grafana (`3000`) tidak diblokir oleh firewall. Untuk sistem berbasis `ufw`:

```bash
sudo ufw allow 9090/tcp
sudo ufw allow 3000/tcp
```
