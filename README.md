# Monitoring Menggunakan Prometheus dan Grafana

Dokumentasi langkah-langkah instalasi dan konfigurasi **Prometheus**, **Node Exporter**, dan **Grafana** untuk kebutuhan monitoring sistem.

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

### Edit konfigurasi `/etc/prometheus/prometheus.yml`

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