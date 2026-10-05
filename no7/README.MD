# 7. 🧪 Testing

## Instructions

- Use **SonarQube** in the CI/CD workflow to test code quality.
- Optional: use other testing tools such as **Trivy** for Docker image vulnerability scanning.
- Test the running application using `wget --spider`.

---

# A. 🔍 SonarQube — Code Quality Testing

SonarQube digunakan untuk melakukan pengecekan kualitas source code pada proses CI/CD.

Pertama, saya menyiapkan SonarQube pada server menggunakan **Ansible**.

Saya memasukkan konfigurasi SonarQube ke dalam Ansible Playbook agar proses deployment dapat dilakukan secara otomatis.

Setelah playbook dijalankan:

```bash
ansible-playbook -i Inventory sonarqube.yaml
```

Kemudian melakukan pengecekan container:

```bash
docker ps
```

Jika container SonarQube sudah berjalan, SonarQube dapat diakses melalui browser menggunakan:

```text
http://<SONARQUBE_IP>:9000
```

Pada konfigurasi awal, saya melakukan login menggunakan:

```text
Username: admin
Password: admin
```

Setelah login pertama kali, SonarQube akan meminta perubahan password administrator.

---

# B. 🔄 SonarQube dalam CI/CD

SonarQube kemudian diintegrasikan ke dalam Jenkins pipeline.


# C. 📦 Install Wget for Application Testing

Untuk melakukan testing terhadap aplikasi yang sedang berjalan, saya menggunakan `wget`.

Karena image/container Jenkins sebelumnya belum memiliki `wget`, saya melakukan perubahan pada Dockerfile Jenkins.

### Sebelumnya

```dockerfile
apt-get install -y ca-certificates curl gnupg && \
```

### Setelah diperbaiki

```dockerfile
apt-get install -y ca-certificates curl wget gnupg && \
```

Setelah Dockerfile diperbarui, Jenkins image dibuild kembali dan container Jenkins dijalankan ulang.

Kemudian saya melakukan testing kembali untuk memastikan `wget` sudah tersedia.

```bash
wget --version
```

---

# D. 🌐 Testing Running Application

Setelah aplikasi berhasil di-deploy oleh CI/CD, saya melakukan pengecekan terhadap aplikasi yang sedang berjalan menggunakan:

```bash
wget --spider <APPLICATION_URL>
```

Contoh:

```bash
wget --spider http://frontend.example.com
```

Untuk backend:

```bash
wget --spider http://api.example.com
```

Option `--spider` digunakan untuk mengecek resource tanpa benar-benar melakukan download file.

Jika aplikasi dapat diakses, akan mendapatkan response HTTP dari server.


---

# F. 🖥️ Resource Server

Jika proses SonarQube atau Jenkins terasa lambat / mengalami lag, resource VM dapat ditingkatkan.

Salah satu konfigurasi yang dapat digunakan:

```text
Instance: m7i-flex.large
vCPU: 2
Memory: 8 GB
```

Peningkatan resource dapat membantu terutama ketika Jenkins dan SonarQube berjalan pada server yang sama atau menjalankan proses build dan analysis secara bersamaan.

---

Dengan konfigurasi tersebut, testing tidak hanya dilakukan pada source code, tetapi juga pada **Docker image dan aplikasi yang sudah berjalan**.
