# 5. 🚀 Staging — Database, Application & Load Balancing

## Instructions

### Database
- Use PostgreSQL as the application database.
- Deploy PostgreSQL using Docker.
- Set the volume location to `/home/$USER/storage`.
- Allow the database to be accessed remotely from another server.

### Application
- Create Docker images for frontend and backend.
- Use lightweight Docker images.
- Build Docker images on every environment using multi-stage Docker builds.
- Create load balancing for frontend and backend.

---

# A. 🗄️ Database — PostgreSQL

## Step 1 — Check Docker on Database Server

Pertama, saya melakukan konfigurasi menggunakan **Ansible** dan memastikan Docker sudah terinstall pada database server.

Untuk mengecek container Docker:

```bash
ansible database -i Inventory -m shell -a "docker ps"
```

Kemudian mengecek versi Docker:

```bash
ansible database -i Inventory -m shell -a "docker --version"
```

Jika Docker belum tersedia, Docker terlebih dahulu di-install menggunakan Ansible.

---

## Step 2 — Open PostgreSQL Port

Karena database akan digunakan oleh backend yang berada di server berbeda, port PostgreSQL harus dapat menerima koneksi dari application server.

Port PostgreSQL:

```text
5432
```

Security Group / firewall dikonfigurasi agar application server dapat mengakses:

```text
Database Server : 5432
```

> Sebaiknya port `5432` hanya dibuka untuk private IP / security group application server, bukan untuk seluruh internet.

---

## Step 3 — Deploy PostgreSQL Using Docker

PostgreSQL dijalankan menggunakan Docker dengan persistent storage.

Storage diarahkan ke:

```text
/home/$USER/storage
```

Contoh struktur:

```text
/home/amri/storage/
└── postgres/
```

Contoh deployment:

```bash
docker run -d \
  --name postgres \
  -e POSTGRES_USER=<username> \
  -e POSTGRES_PASSWORD=<password> \
  -e POSTGRES_DB=<database> \
  -p 5432:5432 \
  -v /home/$USER/storage:/var/lib/postgresql/data \
  postgres
```

Dengan menggunakan volume tersebut, data PostgreSQL tetap tersimpan walaupun container PostgreSQL dibuat ulang.

---

# B. 🔗 Remote Database Connection

Setelah PostgreSQL berjalan, saya memastikan database dapat diakses dari application/backend server.

Dari backend server dilakukan pengecekan koneksi:

```bash
nc -zvw 5 <DATABASE_PRIVATE_IP> 5432
```

Jika berhasil, berarti backend server dapat terhubung ke PostgreSQL melalui port `5432`.

---

# C. ⚙️ Backend Configuration & Migration

Setelah database dapat diakses, saya masuk ke backend server dan melakukan konfigurasi `.env`.

Database configuration diarahkan ke IP database server:

```env
DB_HOST=<DATABASE_PRIVATE_IP>
DB_PORT=5432
DB_USER=<DATABASE_USER>
DB_PASSWORD=<DATABASE_PASSWORD>
DB_NAME=<DATABASE_NAME>
```

Kemudian melakukan pengecekan migration pada source code:

```bash
grep -Rni "RunMigration" .
```

Setelah itu melakukan download dependency Go:

```bash
go mod download
```

Kemudian menjalankan backend:

```bash
go run main.go
```

Saat aplikasi dijalankan, proses migration akan membuat table yang diperlukan pada PostgreSQL.

---

# D. 🐳 Application — Docker Image

Frontend dan backend dibuat menjadi Docker image agar aplikasi dapat dijalankan secara konsisten pada environment staging maupun production.

Application terdiri dari:

```text
Frontend
Backend
Database
```

---

## Frontend Docker Image

Frontend dibuat menggunakan **multi-stage Docker build**.

Konsepnya:

```text
Node.js
   ↓
Install Dependencies
   ↓
Build React Application
   ↓
Production Image
   ↓
Frontend Container
```

Multi-stage build digunakan untuk memisahkan proses build dengan production runtime sehingga image akhir menjadi lebih ringan.

---

## Backend Docker Image

Backend juga dibuat menggunakan multi-stage Docker build.

Konsepnya:

```text
Go Builder
    ↓
Download Dependencies
    ↓
Build Application
    ↓
Production Image
    ↓
Backend Container
```

Dengan cara ini, source code dan tools yang hanya dibutuhkan saat proses build tidak perlu dimasukkan ke production image.


---

# E. 🔄 Build Docker Image on Every Environment

Docker image dibuild pada setiap environment sesuai kebutuhan deployment.

Contoh:

```bash
docker build -t frontend:latest .
```

Kemudian image diberi tag sesuai dengan registry:

```bash
docker tag frontend:latest <REGISTRY>/frontend:latest
```

Image kemudian dapat di-push:

```bash
docker push <REGISTRY>/frontend:latest
```

Backend menggunakan proses yang sama:

```bash
docker build -t backend:latest .
docker tag backend:latest <REGISTRY>/backend:latest
docker push <REGISTRY>/backend:latest
```

---

# F. ⚖️ Load Balancing Frontend & Backend

Untuk membuat load balancing, saya menggunakan lebih dari satu server/application instance.

Pertama, saya menghubungkan **AWS Account 1 dan AWS Account 2** sesuai kebutuhan infrastructure.

Kemudian memastikan server pada account kedua dapat berkomunikasi dengan server pada account pertama.

Security Group pada server juga dikonfigurasi agar traffic yang diperlukan dapat diterima.


Setelah konektivitas antar server tersedia, saya melakukan konfigurasi menggunakan Ansible.

---

# g. 🤖 Ansible Load Balancing

Konfigurasi load balancing dilakukan menggunakan Ansible Playbook.

Ansible digunakan untuk:

- Mengkonfigurasi frontend server
- Mengkonfigurasi backend server
- Menginstall Nginx
- Mengatur upstream
- Mengatur reverse proxy
- Mengarahkan traffic ke beberapa server
- Menjalankan konfigurasi secara konsisten

```

Playbook Ansible kemudian dijalankan untuk menerapkan konfigurasi ke server:

```bash
ansible-playbook -i Inventory <playbook>.yaml
```
