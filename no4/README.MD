# 4. 🐳 Docker Registry Private

## Instructions

- Deploy Docker Registry Private on the server.
- Push Docker image into your own Docker Registry.
- Configure reverse proxy for Docker Registry:
  `registry.{your_name}.studentdumbways.my.id`

## Step 1 — Server Configuration Using Ansible

Pertama, saya melakukan konfigurasi server menggunakan **Ansible** dan mengarahkan konfigurasi Docker Registry ke VM yang telah disiapkan sebelumnya.

Ansible digunakan untuk membantu melakukan konfigurasi secara otomatis pada server.


## Step 2 — Clone Repository

Setelah server siap, saya melakukan clone repository aplikasi ke server:

```bash
git clone <repository-url>
```

Kemudian masuk ke directory project:

```bash
cd <project-directory>
```

---

## Step 3 — Build Docker Image

Selanjutnya melakukan build Docker image untuk aplikasi frontend:

```bash
docker build -t frontend:latest .
```

Setelah image berhasil dibuat, image diberi tag sesuai dengan alamat Private Docker Registry:

```bash
docker tag frontend:latest 15.232.52.124:5000/frontend:latest
```

---

## Step 4 — Configure Docker Insecure Registry

Karena Docker Registry yang digunakan masih menggunakan HTTP pada port `5000`, Docker perlu dikonfigurasi agar mengizinkan koneksi ke insecure registry.

Membuat atau mengedit file:

```bash
sudo nano /etc/docker/daemon.json
```

Kemudian menambahkan konfigurasi:

```json
{
  "insecure-registries": [
    "15.232.52.124:5000"
  ]
}
```

Setelah itu restart Docker:

```bash
sudo systemctl restart docker
```

Kemudian memastikan Docker kembali berjalan:

```bash
sudo systemctl status docker
```

---

## Step 5 — Push Image to Private Registry

Setelah konfigurasi selesai, Docker image dapat di-push ke Private Registry:

```bash
docker push 15.232.52.124:5000/frontend:latest
```

Jika proses berhasil, image akan tersimpan di Private Docker Registry.

---

## Step 6 — Check Docker Registry

Untuk memastikan image sudah tersimpan di registry, dapat dilakukan pengecekan menggunakan endpoint Registry API.

### Check Image

```text
http://15.232.52.124:5000/v2/_catalog
```

Endpoint tersebut digunakan untuk melihat daftar repository/image yang tersedia di registry.

### Check Image Tags

Untuk melihat tag dari image `frontend`:

```text
http://15.232.52.124:5000/v2/frontend/tags/list
```

Contoh response:

```json
{
  "name": "frontend",
  "tags": [
    "latest"
  ]
}
```

---

## Step 7 — Configure Reverse Proxy

Setelah Private Docker Registry berjalan, saya melakukan konfigurasi **Nginx Reverse Proxy** agar registry dapat diakses menggunakan domain.

Domain yang digunakan:

```text
registry.{your_name}.studentdumbways.my.id
```

Konfigurasi Nginx berada di:

```bash
/etc/nginx/sites-available/registry
```

Contoh konfigurasi:

```nginx
server {
    listen 80;
    server_name registry.{your_name}.studentdumbways.my.id;

    location / {
        proxy_pass http://15.232.52.124:5000;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Kemudian konfigurasi diaktifkan dan Nginx direload:

```bash
sudo ln -s /etc/nginx/sites-available/registry \
/etc/nginx/sites-enabled/registry

sudo nginx -t

sudo systemctl reload nginx
```

---

## Step 8 — Configure Cloudflare

Selanjutnya saya melakukan konfigurasi DNS pada **Cloudflare** untuk mengarahkan domain registry ke server yang menjalankan Docker Registry.

Contoh:

```text
registry.{your_name}.studentdumbways.my.id
            ↓
       Server IP
            ↓
           Nginx
            ↓
     Docker Registry
          :5000
```

Dengan konfigurasi tersebut, Docker Registry yang sebelumnya dapat diakses melalui:

```text
http://15.232.52.124:5000
```

dapat diakses melalui domain:

```text
registry.{your_name}.studentdumbways.my.id
```

