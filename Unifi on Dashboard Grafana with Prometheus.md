# 📘 Tutorial: Monitoring UniFi dengan Docker

## 📋 Prasyarat
Sebelum mulai, pastikan:
- Docker dan Docker Compose sudah terinstall di server kamu.
- Sudah memiliki akses ke UniFi Controller.
- Telah membuat network Docker bernama `unifi-net`.

---

## 🐳 1. Install Docker dan Docker Compose

### 🔄 Update sistem:
```bash
sudo apt update && sudo apt upgrade -y
```

### ⚙️ Install dependensi:
```bash
sudo apt install ca-certificates curl gnupg lsb-release -y
```

### 🔐 Tambahkan GPG Key:
```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg \
  --dearmor -o /etc/apt/keyrings/docker.gpg
```

### 📦 Tambahkan repository Docker:
```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### 📥 Install Docker:
```bash
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

### ✅ Cek versi Docker:
```bash
docker --version
docker compose version
```

### 🙋‍♂️ (Opsional) Jalankan Docker tanpa `sudo`:
```bash
sudo usermod -aG docker $USER
newgrp docker
```

---

## 🗂️ 2. Buat Struktur Folder
```bash
mkdir unifi-monitoring && cd unifi-monitoring
mkdir prometheus grafana
```

---

## 📝 3. Buat File `docker-compose.yml`

```yaml
version: '3'

services:
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus:/etc/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
    networks:
      - unifi-net

  unifi-poller:
    image: golift/unifi-poller
    container_name: unifi-poller
    restart: always
    environment:
      - UP_UNIFI_DEFAULT_USER=administrator
      - UP_UNIFI_DEFAULT_PASS=4RADrDtd
      - UP_UNIFI_DEFAULT_URL=https://10.1.1.74:8443
      - UP_UNIFI_CONTROLLER_TYPE=unifi_os
      - UP_PROMETHEUS_DISABLE=false
      - UP_PROMETHEUS_NAMESPACE=unpoller
    ports:
      - "9130:9130"
    networks:
      - unifi-net

  grafana:
    image: grafana/grafana
    container_name: grafana
    restart: always
    ports:
      - "3000:3000"
    networks:
      - unifi-net
    volumes:
      - ./grafana:/var/lib/grafana

networks:
  unifi-net:
    external: true
```

---

## ⚙️ 4. Buat File `prometheus.yml`

Buat file di dalam folder `prometheus/`:

```yaml
global:
  scrape_interval: 30s

scrape_configs:
  - job_name: 'unifi-poller'
    static_configs:
      - targets: ['unifi-poller:9130']
```

---

## 🌐 5. Buat Docker Network (jika belum ada)

```bash
docker network create unifi-net
```

---

## 🚀 6. Jalankan Stack

```bash
docker-compose up -d
```

---

## 🔍 7. Cek Semua Berjalan

- Prometheus: [http://localhost:9090](http://localhost:9090)  
- UniFi Poller: [http://localhost:9130/metrics](http://localhost:9130/metrics)  
- Grafana: [http://localhost:3000](http://localhost:3000)  
  (default login: `admin / admin`)

---

## 📊 8. Tambahkan Data Source di Grafana

- Masuk ke menu `Settings > Data Sources`
- Tambahkan **Prometheus**
- URL: `http://prometheus:9090`

---

## 📈 9. Import Dashboard UniFi

- Di Grafana → klik `+` → `Import`
- Gunakan salah satu dashboard ID:
  - `11310` → UniFi General
  - `11311` → UniFi Client Insights

---

## 🔐 10. Tips Keamanan

- Jangan gunakan password default.
- Simpan variabel rahasia dalam `.env` file.
- Aktifkan HTTPS jika sistem diakses dari internet.

---

