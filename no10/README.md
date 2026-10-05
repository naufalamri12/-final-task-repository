# 5. ☸️ Production Deployment — Kubernetes

## Instructions

- Application runs inside a Kubernetes cluster for production environment.
- Deploy Frontend, Backend, and Database inside Kubernetes.
- Application must run 100% with Backend and Database integration.
- Configure SSL using Cert-Manager.
- Configure CI/CD integration for production.
- Use Nginx Ingress or another Ingress Controller.

---

## 1. Kubernetes Production

Pada production environment, aplikasi dijalankan di dalam Kubernetes cluster menggunakan namespace `production`.

Komponen utama:

```text
Frontend
Backend
PostgreSQL Database
Ingress
Cert-Manager
Private Docker Registry
Jenkins CI/CD
```

Setelah manifest Kubernetes dibuat, saya melakukan pengecekan resource menggunakan `kubectl`.

---

## 2. PostgreSQL & PVC

Pertama, saya melakukan pengecekan Persistent Volume Claim:

```bash
kubectl get pvc -n production
```

Pada awal deployment, PVC dapat berada dalam status:

```text
Pending
```

Hal tersebut dapat terjadi karena storage belum tersedia atau belum mendapatkan Persistent Volume.

Setelah storage berhasil dikonfigurasi, PVC harus berubah menjadi:

```text
Bound
```

---

## 3. Deploy PostgreSQL

Setelah PVC siap, PostgreSQL dijalankan di dalam Kubernetes.

Kemudian saya mengecek PostgreSQL dari dalam Pod:

```bash
kubectl exec -it postgres-744ddb5dd9-llxmh \
-n production -- \
psql -U dumbmerch -d dumbmerch
```

Setelah masuk ke PostgreSQL, saya melakukan pengecekan database:

```sql
\l
```

atau:

```sql
\conninfo
```

Dengan pengecekan tersebut saya dapat memastikan PostgreSQL berjalan dan database `dumbmerch` dapat diakses.

---

## 4. Backend Docker Image

Setelah database siap, saya membuat image backend yang digunakan oleh Kubernetes.

Image staging diberi tag untuk deployment Kubernetes:

```bash
docker tag \
15.232.52.124:5000/dumbmerch-backend:staging \
registry.amri.studentdumbways.my.id/dumbmerch-backend:kubernetes
```

Kemudian melakukan pengecekan:

```bash
docker images | grep dumbmerch-backend
```

Setelah itu image di-push ke private registry:

```bash
docker push \
registry.amri.studentdumbways.my.id/dumbmerch-backend:kubernetes
```

---

## 5. Backend Secret & Deployment

Saya membuat Secret untuk konfigurasi backend dan database:

```bash
nano backend-secret.yaml
```

Kemudian membuat deployment backend:

```bash
nano backend.yaml
```

Secret digunakan untuk menyimpan konfigurasi seperti:

```text
DB_HOST
DB_PORT
DB_USER
DB_PASSWORD
DB_NAME
```

Deployment kemudian dijalankan:

```bash
kubectl apply -f backend-secret.yaml
kubectl apply -f backend.yaml
```

---

## 6. Backend & Database Integration

Backend diarahkan ke PostgreSQL yang berjalan di Kubernetes.

Setelah deployment, saya mengecek log backend:

```bash
kubectl logs -n production -l app=backend
```

Jika terdapat perubahan pada environment atau konfigurasi backend, Docker image dibuat kembali dan di-push ke private registry.

Kemudian deployment diperbarui menggunakan image terbaru.

Flow:

```text
Frontend
   ↓
Backend
   ↓
PostgreSQL
   ↓
Persistent Volume
```

---

# 7. SSL with Cert-Manager

Untuk menyediakan SSL/HTTPS, saya menggunakan **Cert-Manager**.

Pertama menambahkan Helm repository:

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update
```

Kemudian install Cert-Manager:

```bash
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set crds.enabled=true
```

---

## 8. ClusterIssuer

Setelah Cert-Manager berhasil di-install, saya membuat konfigurasi:

```bash
nano clusterissuer.yaml
```

Untuk menggunakan Cloudflare DNS Challenge, saya membuat Secret Cloudflare:

```bash
kubectl create secret generic cloudflare-api-token-secret \
  --namespace cert-manager \
  --from-literal=api-token='<CLOUDFLARE_API_TOKEN>'
```

Kemudian membuat:

```bash
nano clusterissuer-dns.yaml
```

Setelah diterapkan, saya melakukan pengecekan:

```bash
kubectl get clusterissuer
```

ClusterIssuer digunakan untuk mendapatkan SSL certificate dari Let's Encrypt melalui DNS Challenge.

> API token Cloudflare asli tidak disimpan di repository. Pada dokumentasi digunakan placeholder.

---

# 9. Backend Ingress & SSL

Pada Backend Ingress ditambahkan annotation:

```yaml
annotations:
  cert-manager.io/cluster-issuer: letsencrypt-prod-dns
```

Backend Ingress digunakan untuk mengarahkan traffic dari domain menuju Backend Service.

Setelah deployment dilakukan, saya memonitor certificate:

```bash
kubectl get certificate -n production -w
```

Certificate dianggap berhasil apabila statusnya:

```text
READY: True
```

---

# 10. Frontend Deployment

Setelah backend berhasil, saya melakukan deployment frontend dengan cara yang sama.

Frontend terdiri dari:

```text
Frontend Deployment
Frontend Service
Frontend Ingress
```

Kemudian menjalankan:

```bash
kubectl apply -f frontend.yaml
kubectl apply -f frontend-ingress.yaml
```

Frontend juga menggunakan Cert-Manager untuk mendapatkan SSL certificate.

Annotation:

```yaml
annotations:
  cert-manager.io/cluster-issuer: letsencrypt-prod-dns
```

Kemudian melakukan pengecekan:

```bash
kubectl get pods -n production
kubectl get certificate -n production
```

---

# 11. Kubernetes Troubleshooting

Jika terjadi error pada Pod, saya melakukan pengecekan menggunakan:

```bash
kubectl describe pod <pod-name> -n production
```

Contoh:

```bash
kubectl describe pod \
frontend-555d8bcd6b-l8wdb \
-n production
```

Kemudian mengecek log:

```bash
kubectl logs <pod-name> -n production
```

Hal ini digunakan untuk mengetahui penyebab Pod gagal berjalan atau gagal melakukan koneksi ke service lain.

---

# 12. Private Docker Registry on K3s

Karena private Docker Registry menggunakan HTTP pada port `5000`, worker node Kubernetes perlu dikonfigurasi agar dapat mengakses insecure registry.

Pada worker:

```bash
sudo mkdir -p /etc/rancher/k3s
```

Kemudian:

```bash
sudo nano /etc/rancher/k3s/registries.yaml
```

Isi:

```yaml
mirrors:
  "15.232.52.124:5000":
    endpoint:
      - "http://15.232.52.124:5000"
```

Setelah konfigurasi selesai, service K3s pada worker perlu direstart agar konfigurasi registry diterapkan.

Dengan konfigurasi tersebut, worker Kubernetes dapat melakukan pull image dari private Docker Registry melalui HTTP.

---

# 13. Jenkins CI/CD for Production

Untuk production environment, Jenkins digunakan untuk melakukan CI/CD ke Kubernetes.

Flow CI/CD:

```text
GitHub
   ↓
Jenkins
   ↓
Pull Repository
   ↓
Build Docker Image
   ↓
Testing
   ↓
Push Image
   ↓
Private Docker Registry
   ↓
kubectl
   ↓
Kubernetes Production
   ↓
Redeploy Application
```

---

# 14. Jenkins ServiceAccount & RBAC

Pertama saya membuat ServiceAccount untuk Jenkins:

```bash
kubectl create serviceaccount jenkins -n production
```

Kemudian membuat file:

```bash
nano jenkins-rbac.yaml
```

RBAC digunakan untuk memberikan permission kepada Jenkins agar dapat melakukan deployment pada namespace production.

Kemudian:

```bash
kubectl apply -f jenkins-rbac.yaml
```

Token Jenkins dibuat menggunakan:

```bash
kubectl create token jenkins -n production
```

---

# 15. Kubernetes CA & Token

Pada Kubernetes master, saya mengambil CA certificate:

```bash
CA=$(sudo cat \
/var/lib/rancher/k3s/server/tls/server-ca.crt | base64 -w0)
```

Kemudian membuat token:

```bash
TOKEN=$(kubectl create token jenkins -n production)
```

Token dapat dicek menggunakan:

```bash
echo "$TOKEN"
```

CA dan token tersebut digunakan untuk menghubungkan Jenkins dengan Kubernetes cluster.

---

# 16. Configure kubectl on Jenkins VM

Pada VM Jenkins, saya meng-install `kubectl`:

```bash
sudo snap install kubectl --classic
```

Kemudian membuat Kubernetes configuration:

```bash
nano ~/.kube/config
```

Contoh:

```yaml
apiVersion: v1
kind: Config

clusters:
- name: production-k3s
  cluster:
    server: https://10.1.1.20:6443
    certificate-authority-data: ISI_BASE64_CA_CERT

users:
- name: jenkins
  user:
    token: TOKEN_JENKINS

contexts:
- name: jenkins-production
  context:
    cluster: production-k3s
    user: jenkins
    namespace: production

current-context: jenkins-production
```

Kemudian melakukan testing:

```bash
kubectl get pods -n production
```

dan:

```bash
kubectl get deployments -n production
```

---

# 17. Test Private Registry & kubectl

Saya juga melakukan pengecekan private registry:

```bash
curl -k \
https://registry.amri.studentdumbways.my.id/v2/
```

Kemudian memastikan `kubectl` tersedia:

```bash
kubectl version --client
```

Jika kedua command berhasil, Jenkins VM dapat mengakses private registry dan memiliki `kubectl`.

---

# 18. Install kubectl inside Jenkins Container

Jika Jenkins berjalan menggunakan Docker container, `kubectl` juga perlu tersedia di dalam container Jenkins.

Masuk ke container:

```bash
docker exec -u root -it jenkins bash
```

Kemudian:

```bash
apt-get update
apt-get install -y ca-certificates curl
```

Download kubectl:

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

Install:

```bash
install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

Kemudian cek:

```bash
kubectl version --client
```

---

# 19. Copy Kubernetes Configuration to Jenkins

Kubernetes configuration kemudian diberikan kepada Jenkins container.

Membuat directory:

```bash
sudo mkdir -p \
/var/lib/docker/volumes/jenkins_home/_data/.kube
```

Copy config:

```bash
sudo cp /home/amri/.kube/config \
/var/lib/docker/volumes/jenkins_home/_data/.kube/config
```

Kemudian mengatur permission:

```bash
sudo chown -R 1000:1000 \
/var/lib/docker/volumes/jenkins_home/_data/.kube

sudo chmod 700 \
/var/lib/docker/volumes/jenkins_home/_data/.kube

sudo chmod 600 \
/var/lib/docker/volumes/jenkins_home/_data/.kube/config
```

---

# 20. Custom Jenkins Docker Image

Agar Jenkins dapat menjalankan Docker CLI dan `kubectl`, saya membuat custom Dockerfile:

```bash
nano /home/amri/jenkins/Dockerfile
```

Dockerfile:

```dockerfile
FROM jenkins/jenkins:lts-jdk17

USER root

RUN apt-get update && \
    apt-get install -y ca-certificates curl wget gnupg && \
    install -m 0755 -d /etc/apt/keyrings && \
    curl -fsSL https://download.docker.com/linux/debian/gpg \
    -o /etc/apt/keyrings/docker.asc && \
    chmod a+r /etc/apt/keyrings/docker.asc && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian $(. /etc/os-release && echo "$VERSION_CODENAME") stable" \
    > /etc/apt/sources.list.d/docker.list && \
    apt-get update && \
    apt-get install -y docker-ce-cli && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Install kubectl
RUN KUBECTL_VERSION=$(curl -L -s https://dl.k8s.io/release/stable.txt) && \
    curl -LO "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl" && \
    install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl && \
    rm kubectl

RUN groupadd -g 986 docker && \
    usermod -aG docker jenkins

USER jenkins
```

> `USER jenkins` menggunakan huruf kecil karena user bawaan Jenkins bernama `jenkins`.

---

# 21. Build Custom Jenkins Image

Build image:

```bash
docker build -t jenkins-custom:lts-jdk17 .
```

Kemudian mengecek Kubernetes config:

```bash
ls -l ~/.kube/config
```

---

# 22. Recreate Jenkins Container

Stop dan remove Jenkins lama:

```bash
docker stop jenkins
docker rm jenkins
```

Kemudian menjalankan Jenkins menggunakan custom image:

```bash
docker run -d \
  --name jenkins \
  --restart unless-stopped \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /home/amri/.kube/config:/var/jenkins_home/.kube/config:ro \
  jenkins-custom:lts-jdk17
```

Dengan konfigurasi tersebut:

- Jenkins memiliki Docker CLI.
- Jenkins memiliki `kubectl`.
- Jenkins dapat mengakses Docker daemon.
- Jenkins memiliki Kubernetes configuration.
- Jenkins dapat berkomunikasi dengan Kubernetes production cluster.

---
