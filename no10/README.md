Instructions

No 5 (Deployment) app runs inside a kubernetes cluster (include frontend, backend, and database) for production Environment

Apps running 100% with backend and db integration

For SSL you can use Cert Manager

CICD integration for production

Ingress Nginx or others.

JAWABAN

buat kode setelah itu

cek kode pvc menggunakan kubectl get pvc -n production
PVC kemungkinan belum langsung Bound dan tapi pending

buat dan start ostgresql
cek database dari dalam pods
kubectl exec -it postgres-744ddb5dd9-llxmh -n production -- psql -U dumbmerch -d dumbmerch
\l atau \conninfo

setelah itu kita membuat image baru
docker tag \
15.232.52.124:5000/dumbmerch-backend:staging \
registry.amri.studentdumbways.my.id/dumbmerch-backend:kubernetes
mengecheck
docker images | grep dumbmerch-backend

buat secret untuk migration nano backend-secret.yaml
buat backend di nano backend.yaml

cek logs backend
kubectl logs -n production -l app=backend
jangan lupa ubah env dan push docker image lagi

untuk ssl
kita install
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set crds.enabled=true

setelah itu buat
clusterissuer.yaml

ubah backend.yaml dan tambahkan
annotations:
  cert-manager.io/cluster-issuer: letsencrypt-prod
setelah itu masukan ini
kubectl create secret generic cloudflare-api-token-secret \
  --namespace cert-manager \
  --from-literal=api-token='tokencloudfire'
setelah itu buat file  clusterissuer-dns.yaml
kita cek
kubectl get clusterissuer
setelah itu kita juga ubah di backend-ingress.yaml
cert-manager.io/cluster-issuer: letsencrypt-prod menjadi cert-manager.io/cluster-issuer: letsencrypt-prod-dns

Tunggu dan periksa status kubectl get certificate -n production -w

setelah itu kita gnati yang frontend sama seperti yang backend
membuat frontend dan inggres frontend
semisalnya ada eror jalankan kubectl describe pod frontend-555d8bcd6b-l8wdb -n production
kalua ini (http: server gave HTTP response to HTTPS client)

pergi ke workers
sudo mkdir -p /etc/rancher/k3s
sudo nano /etc/rancher/k3s/registries.yaml
isi dengan
mirrors:
  "15.232.52.124:5000":
    endpoint:
      - "http://15.232.52.124:5000"


Jenkins ci/cd
nano jenkins-rbac.yaml
membuat token (kubectl create token jenkins -n production)
install di vm Jenkins sudo snap install kubectl --classic

dimaster jalankan
CA=$(sudo cat /var/lib/rancher/k3s/server/tls/server-ca.crt | base64 -w0)
TOKEN=$(kubectl create token jenkins -n production)
echo "$TOKEN"

setelah itu di vm master
kubectl create serviceaccount jenkins -n production
kubectl apply -f jenkins-rbac.yaml
kubectl create token jenkins -n production

setelah itu di vm jenkins
nano ~/.kube/config
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

di vm Jenkins
kubectl get pods -n production
kubectl get deployments -n production
cek registery dan kubectl
curl -k https://registry.amri.studentdumbways.my.id/v2/
kubectl version --client

setelah itu
docker exec -u root -it jenkins bash
apt-get update
apt-get install -y ca-certificates curl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client

setelah itu
sudo mkdir -p /var/lib/docker/volumes/jenkins_home/_data/.kube
sudo cp /home/amri/.kube/config \
  /var/lib/docker/volumes/jenkins_home/_data/.kube/config
sudo chown -R 1000:1000 \
  /var/lib/docker/volumes/jenkins_home/_data/.kube
sudo chmod 700 \
  /var/lib/docker/volumes/jenkins_home/_data/.kube
sudo chmod 600 \
  /var/lib/docker/volumes/jenkins_home/_data/.kube/config


setelah itu
nano /home/amri/jenkins/Dockerfile
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

USER Jenkins


setelah itu
docker build -t jenkins-custom:lts-jdk17 .
ls -l ~/.kube/config
docker stop jenkins
docker rm jenkins
docker run -d \
  --name jenkins \
  --restart unless-stopped \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /home/amri/.kube/config:/var/jenkins_home/.kube/config:ro \
  jenkins-custom:lts-jdk17

