# 9. 🌐 Domain, HTTPS & Certificate Renewal

## Instructions

Create the following domains:

### Monitoring

| Domain | Function |
|---|---|
| `exporter.<name>.studentdumbways.my.id` | Node Exporter |
| `prom.<name>.studentdumbways.my.id` | Prometheus |
| `monitoring.<name>.studentdumbways.my.id` | Grafana |

### Docker Registry

| Domain | Function |
|---|---|
| `registry.<name>.studentdumbways.my.id` | Private Docker Registry |

### Staging

| Domain | Function |
|---|---|
| `staging.<name>.studentdumbways.my.id` | Frontend Application |
| `api.staging.<name>.studentdumbways.my.id` | Backend API |

### Production

| Domain | Function |
|---|---|
| `<name>.studentdumbways.my.id` | Frontend Application |
| `api.<name>.studentdumbways.my.id` | Backend API |

All domains must use **HTTPS**.

A Bash script must also be created for automatic certificate renewal.

---

## A. 🤖 Ansible Configuration

Pertama, saya membuat file dan konfigurasi **Ansible** untuk membantu melakukan deployment dan konfigurasi server.

Ansible digunakan untuk mengatur:

- Nginx
- Domain
- Reverse Proxy
- SSL/HTTPS
- Certificate
- Docker Registry
- Monitoring
- Staging
- Production

Contoh struktur:

```text id="g7x9a0"
ansible/
├── Inventory
├── playbooks/
│   ├── nginx.yaml
│   ├── ssl.yaml
│   ├── monitoring.yaml
│   └── registry.yaml
├── templates/
└── vault.yml
```

---

# B. 🔐 Ansible Vault

Credential dan informasi sensitif tidak disimpan secara langsung di repository.

Untuk mengamankan informasi tersebut, saya menggunakan **Ansible Vault**.

Membuat file Vault:

```bash id="l2bq5f"
ansible-vault encrypt vault.yml
```

Kemudian memasukkan password:

```text id="5k9b1x"
New Vault password:
Confirm New Vault password:
Encryption successful
```

File `vault.yml` kemudian digunakan untuk menyimpan data sensitif seperti:

- Cloudflare API Token
- Password
- Secret
- Credential lainnya

Contoh:

```yaml id="v1h9ro"
cloudflare_api_token: <SECRET>
```

Dengan Ansible Vault, credential tidak disimpan dalam bentuk plaintext.

---

# C. 🌐 Domain Configuration

Setiap domain diarahkan ke server atau service yang sesuai.



# D. 📊 Monitoring Domains

Monitoring menggunakan tiga domain utama.

### Node Exporter

```text id="q3i2jw"
exporter.amri.studentdumbways.my.id
```

Digunakan untuk mengakses Node Exporter.

### Prometheus

```text id="n5ys7z"
prom.armi.studentdumbways.my.id
```

Digunakan untuk mengakses Prometheus.

### Grafana

```text id="8b3h6t"
monitoring.armi.studentdumbways.my.id
```

Digunakan untuk mengakses Grafana Dashboard.

---

# E. 🐳 Docker Registry Domain

Private Docker Registry menggunakan:

```text id="8p7p6c"
registry.amri.studentdumbways.my.id
```


# F. 🧪 Staging Domains

Staging application menggunakan dua domain.

Frontend:

```text id="u6c0aw"
staging.amri.studentdumbways.my.id
```

Backend:

```text id="7x1tcb"
api.staging.amri.studentdumbways.my.id
```

# G. 🚀 Production Domains

Production application menggunakan:

Frontend:

```text id="nd8t5c"
amri.studentdumbways.my.id
```

Backend:

```text id="q5l7v2"
api.amri.studentdumbways.my.id
```

Flow:

```text id="k9c4lm"
<name>.studentdumbways.my.id
             ↓
          Frontend
             ↓
api.<name>.studentdumbways.my.id
             ↓
         Backend API
             ↓
        PostgreSQL
```

---

# H. 🔒 HTTPS

Seluruh domain dikonfigurasi menggunakan **HTTPS**.

Contoh:

```text id="8w6xrm"
https://exporter.<name>.studentdumbways.my.id
https://prom.<name>.studentdumbways.my.id
https://monitoring.<name>.studentdumbways.my.id
https://registry.<name>.studentdumbways.my.id
https://staging.<name>.studentdumbways.my.id
https://api.staging.<name>.studentdumbways.my.id
https://<name>.studentdumbways.my.id
https://api.<name>.studentdumbways.my.id
```

SSL digunakan untuk mengenkripsi komunikasi antara client dengan server.

---

# I. 🔐 SSL Certificate

Certificate digunakan pada Nginx / Ingress untuk menyediakan HTTPS.

Untuk Kubernetes production, certificate dapat dikelola menggunakan **Cert-Manager**.

Untuk server yang menggunakan Nginx secara langsung, certificate dapat diperoleh menggunakan Certbot atau mekanisme SSL yang digunakan pada infrastructure.

Setelah certificate tersedia, Nginx dikonfigurasi untuk menerima:

```text id="h1a5jd"
443 HTTPS
```

dan meneruskan traffic ke service yang sesuai.

---

# J. 🔄 Automatic Certificate Renewal

Karena certificate memiliki masa berlaku, saya membuat **Bash Script** untuk melakukan pengecekan dan renewal certificate secara otomatis.

Contoh:

```bash id="1n7kpm"
#!/bin/bash

certbot renew --quiet

systemctl reload nginx
```

Script tersebut menjalankan:

```bash id="5r3w6u"
certbot renew
```

Kemudian melakukan reload Nginx:

```bash id="t7e0z8"
systemctl reload nginx
```

Agar certificate yang baru digunakan oleh Nginx.

Script dapat disimpan sebagai:

```text id="x9q4mb"
/usr/local/bin/renew-cert.sh
```

Kemudian dibuat executable:

```bash id="m3s5jf"
sudo chmod +x /usr/local/bin/renew-cert.sh
```

---

# K. ⏰ Automatic Renewal with Cron

Bash script dapat dijalankan secara otomatis menggunakan Cron.

Contoh:

```bash id="z8g3rx"
sudo crontab -e
```

Kemudian:

```cron id="p0y6zt"
0 3 * * * /usr/local/bin/renew-cert.sh
```

Script akan dijalankan setiap hari pukul 03:00 untuk mengecek apakah certificate perlu diperbarui.

---

# L. 🔐 Secure Configuration

Credential seperti Cloudflare API Token tidak ditulis langsung pada playbook.

Credential disimpan menggunakan:

```bash id="q2v5na"
ansible-vault encrypt vault.yml
```

Ketika menjalankan playbook yang membutuhkan Vault:

```bash id="7y8r2w"
ansible-playbook -i Inventory nginx.yaml --ask-vault-pass
```

Dengan demikian informasi sensitif tetap terlindungi dari repository GitHub.

---

