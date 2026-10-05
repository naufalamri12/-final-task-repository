# 8. 📊 Monitoring

## Instructions

- Create Basic Auth for Prometheus.
- Monitor resources for all servers.
- Create a fully working Grafana dashboard.
- Monitor Disk, Memory, CPU, and VM Network.
- Monitor all container resources on the VM.
- Configure Grafana Alert / Prometheus Alertmanager.
- Send notifications to Telegram.
- Create alerts for CPU Usage, RAM Usage, Free Storage, and Network I/O / Nginx Monitoring.

---

## Step 1 — Create Ansible Configuration

Pertama, saya membuat file **Ansible Playbook** untuk melakukan konfigurasi monitoring server.

Ansible digunakan untuk melakukan konfigurasi dan deployment monitoring tools seperti:

- Node Exporter
- Prometheus
- Grafana
- cAdvisor
- Alertmanager

Contoh file:

```text
monitoring.yaml
```

---

## Step 2 — Check Server Connectivity

Sebelum menjalankan playbook, saya melakukan pengecekan koneksi ke seluruh server menggunakan Ansible Ping.

```bash
ansible all -i Inventory -m ping
```

Jika seluruh server memberikan response:

```text
SUCCESS
"ping": "pong"
```

berarti Ansible dapat terhubung ke server dan konfigurasi dapat dilanjutkan.

---

## Step 3 — Run Ansible Playbook

Setelah koneksi seluruh server berhasil, saya menjalankan Ansible Playbook:

```bash
ansible-playbook -i Inventory monitoring.yaml
```

Playbook akan melakukan konfigurasi dan menjalankan service monitoring pada server yang sudah ditentukan.

---

## Step 4 — Open Monitoring Ports

Setelah monitoring berhasil di-deploy, saya membuka port yang diperlukan pada **Security Group / Firewall**.

### Node Exporter

```text
Port: 9100
```

Digunakan oleh Node Exporter untuk menyediakan metrics dari server.

### Prometheus

```text
Port: 9090
```

Digunakan untuk mengakses Prometheus dan mengumpulkan metrics dari server.

### Grafana

```text
Port: 3001
```

Digunakan untuk mengakses Grafana Dashboard.

Sehingga:

```text
Node Exporter → 9100
Prometheus    → 9090
Grafana       → 3001
```

---

## Step 5 — Monitoring Server Resources

Node Exporter digunakan untuk mengumpulkan resource metrics dari seluruh server.

Metrics yang dimonitor:

- CPU Usage
- Memory Usage
- Disk Usage
- VM Network

## Step 6 — Container Monitoring

Untuk monitoring resource container pada VM, digunakan **cAdvisor**.

cAdvisor digunakan untuk memonitor:

- Container CPU
- Container Memory
- Container Network
- Container Resource Usage


## Step 7 — Prometheus Basic Auth

Prometheus dikonfigurasi menggunakan **Basic Authentication** agar akses ke Prometheus membutuhkan username dan password.

Prometheus dapat diakses melalui:

```text
http://<PROMETHEUS_IP>:9090
```

Basic Auth digunakan untuk meningkatkan keamanan akses ke Prometheus.

---

## Step 8 — Grafana Dashboard

Prometheus kemudian digunakan sebagai **Data Source** pada Grafana.

Grafana Dashboard dibuat untuk menampilkan:

```text
CPU Usage
Memory Usage
Disk Usage
VM Network
Container Resources
```

Grafana dapat diakses melalui:

```text
http://<GRAFANA_IP>:3001
```

---

## Step 9 — Alerting

Selanjutnya dibuat alert menggunakan **Grafana Alerting / Prometheus Alertmanager**.

Alert yang dibuat meliputi:

### CPU Usage

Memberikan alert apabila penggunaan CPU melebihi threshold yang ditentukan.

### RAM Usage

Memberikan alert apabila penggunaan RAM melebihi threshold.

### Free Storage

Memberikan alert apabila kapasitas storage yang tersedia terlalu rendah.

### Network I/O

Melakukan monitoring terhadap network traffic.

### Nginx Monitoring

Monitoring dilakukan terhadap traffic dan kondisi Nginx sebagai web server.

---

## Step 10 — Telegram Notification

Alert kemudian dihubungkan dengan Telegram sebagai notification channel dan pada saat saya mekaukan instal k3s, notifikasi di telegram berbunyi
