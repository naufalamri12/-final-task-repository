# 6. 🔄 CI/CD — Jenkins

## Instructions

Using Jenkins, create a pipeline running:

1. Repository Pull
2. Image Build
3. Code Testing
4. Push Image into Private Docker Registry
5. SSH into Application Server
6. Pull Image from Private Docker Registry
7. Redeploy Application
8. For Staging, use SonarQube for code quality testing

---

# A. 🤖 Jenkins Server Preparation

Pertama, saya melakukan konfigurasi Jenkins menggunakan **Ansible**.

Setelah konfigurasi selesai, saya melakukan pengecekan container untuk memastikan Jenkins sudah berjalan:

```bash
docker ps
```

Jika container Jenkins sudah berjalan, Jenkins dapat diakses melalui web browser menggunakan:

```text
http://<JENKINS_IP>:8080
```

---

# B. 🔐 Jenkins Initial Setup

Setelah membuka Jenkins melalui browser, saya mengambil initial administrator password dari container Jenkins.

```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Password tersebut kemudian digunakan untuk login ke Jenkins.

Setelah login, saya melakukan konfigurasi awal Jenkins dan membuat credentials yang diperlukan untuk pipeline.

---

# C. 🔑 Jenkins Credentials

Credentials yang diperlukan antara lain:

- GitHub credentials
- SSH credentials
- Docker Registry credentials
- SonarQube credentials


GitHub credentials digunakan agar Jenkins dapat melakukan pull source code dari repository.

SSH credentials digunakan agar Jenkins dapat melakukan koneksi ke application server.

---

# D. 🔑 SSH Configuration

Agar Jenkins dapat melakukan deployment ke application server, saya membuat SSH key khusus Jenkins.

Contoh:

```text
~/.ssh/jenkins_appserver
```

Public key kemudian ditambahkan ke server tujuan pada:

```text
~/.ssh/authorized_keys
```

Dengan konfigurasi tersebut, Jenkins dapat melakukan SSH ke application server tanpa harus memasukkan password secara manual.

Private key kemudian disimpan pada Jenkins Credentials dan digunakan oleh pipeline.

---

# E. 🐳 Docker Configuration

Jenkins membutuhkan akses ke Docker untuk melakukan proses:

- Build image
- Test container
- Push image
- Pull image

Saya memastikan Docker tersedia dan Jenkins dapat menggunakan Docker.

Pengecekan:

```bash
docker ps
```

Jika Jenkins dijalankan sebagai container, Jenkins container juga harus memiliki akses ke Docker daemon sesuai konfigurasi yang digunakan.

---

# F. 📦 Docker Ignore

Sebelum melakukan Docker build, saya membuat `.dockerignore` pada frontend dan backend.

Contoh:

```text
node_modules
.git
.gitignore
README.md
.env
npm-debug.log
```

Untuk backend dapat digunakan:

```text
.git
.gitignore
.env
README.md
tmp
```

`.dockerignore` digunakan agar file yang tidak diperlukan tidak ikut masuk ke Docker build context.

---

# G. 🔄 Jenkins Pipeline

Pipeline Jenkins dibuat untuk menjalankan proses CI/CD secara otomatis.


---

# H. 📥 Repository Pull

Tahap pertama pipeline mengambil source code terbaru dari repository.

Contoh:

```bash
git clone <repository-url>
```

atau menggunakan Jenkins Pipeline:

```groovy
stage('Repository Pull') {
    steps {
        checkout scm
    }
}
```

Dengan demikian Jenkins akan mengambil source code terbaru dari repository.

---

# I. 🧪 Code Testing

Setelah source code berhasil diambil, pipeline menjalankan testing.

Testing dilakukan untuk memastikan source code tidak memiliki error sebelum dibuat menjadi Docker image.

Untuk staging, digunakan **SonarQube** untuk melakukan pengecekan code quality.

---

# J. 🔍 SonarQube — Staging

Pada environment staging, SonarQube dijalankan sebagai bagian dari pipeline.

Jenkins mengirimkan source code ke SonarQube untuk dianalisis.

Contoh konsep pipeline:

```groovy
stage('SonarQube Analysis') {
    steps {
        // Run SonarQube scanner
    }
}


# K. 🐳 Build Docker Image

Setelah testing berhasil, Jenkins melakukan Docker build.

Contoh:

```bash
docker build -t dumbmerch-frontend:staging .
```

Kemudian image diberi tag sesuai alamat private registry:

```bash
docker tag dumbmerch-frontend:staging \
15.232.52.124:5000/dumbmerch-frontend:staging
```

Backend menggunakan proses yang sama:

```bash
docker build -t dumbmerch-backend:staging .
```

---

# L. 📤 Push Image to Private Docker Registry

Setelah image berhasil dibuat, Jenkins melakukan push ke Private Docker Registry.

```bash
docker push 15.232.52.124:5000/dumbmerch-frontend:staging
```

Backend:

```bash
docker push 15.232.52.124:5000/dumbmerch-backend:staging
```

Image kemudian tersimpan pada private registry.

---

# M. 🔐 SSH into Application Server

Setelah image berhasil di-push, Jenkins melakukan SSH ke application server menggunakan SSH credential yang sudah dikonfigurasi sebelumnya.

Contoh:

```bash
ssh -i ~/.ssh/jenkins_appserver amri@<SERVER_IP>
```

Tujuannya adalah menjalankan proses deployment pada application server.

---

# N. 📥 Pull Image from Private Registry

Pada application server, Jenkins menjalankan:

```bash
docker pull 15.232.52.124:5000/dumbmerch-frontend:staging
```

Backend:

```bash
docker pull 15.232.52.124:5000/dumbmerch-backend:staging
```

Dengan demikian application server mendapatkan Docker image terbaru dari private registry.

---

# O. 🚀 Redeploy Application

Setelah image terbaru berhasil di-pull, deployment aplikasi dilakukan kembali.

Container versi lama dihentikan:

```bash
docker stop <container>
docker rm <container>
```

Kemudian container baru dijalankan menggunakan image terbaru.

Contoh:

```bash
docker run -d \
  --name frontend \
  -p 3000:3000 \
  15.232.52.124:5000/dumbmerch-frontend:staging
```

Untuk backend:

```bash
docker run -d \
  --name backend \
  -p 5000:5000 \
  15.232.52.124:5000/dumbmerch-backend:staging
```

Setelah deployment selesai, dilakukan pengecekan:

```bash
docker ps
```

Kemudian aplikasi dapat dicek melalui browser untuk memastikan deployment berhasil.

---


# Q. 🧪 Final Testing

Setelah pipeline selesai, saya melakukan pengecekan untuk memastikan container sudah berjalan:

```bash
docker ps
```

Kemudian melakukan pengecekan image:

```bash
docker images
```

Dan mengecek log container:

```bash
docker logs <container-name>
```

Aplikasi kemudian diakses melalui web browser untuk memastikan frontend dan backend dapat berjalan dengan baik.

---

Jenkins berhasil digunakan untuk mengotomatisasi proses **build, testing, Docker image creation, image push, deployment, dan redeployment** dari source code sampai aplikasi berjalan pada server.
