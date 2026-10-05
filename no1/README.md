# 1. Provisioning

## Instructions

- Attach SSH keys & IP configuration to all VMs.
- Prepare your server with Terraform.
- Server Configuration using Ansible.

## Step 1 — Prepare AWS Access

Pertama, saya mempersiapkan **AWS Access Key** dan **Secret Access Key** untuk menghubungkan Terraform dengan AWS.

Setelah itu, saya melakukan pengecekan versi Terraform dan AWS CLI untuk memastikan tools yang dibutuhkan sudah terinstall.

```bash
terraform version
aws --version
```

## Step 2 — Create Terraform Configuration

Setelah Terraform dan AWS CLI tersedia, saya membuat file konfigurasi Terraform, yaitu:

```text
main.tf
terraform.tf
```

## Step 3 — Initialize Terraform

Selanjutnya menjalankan:

```bash
terraform init
```

Command ini digunakan untuk menginisialisasi project Terraform dan mengunduh provider yang dibutuhkan.

> **Note:** Lama proses `terraform init` bergantung pada koneksi internet dan proses download provider. Pada kondisi tertentu proses ini dapat membutuhkan waktu lebih lama.

## Step 4 — Validate Configuration

Setelah initialization selesai, saya melakukan validasi konfigurasi Terraform:

```bash
terraform validate
```

Command ini digunakan untuk memastikan konfigurasi Terraform tidak memiliki kesalahan syntax atau konfigurasi dasar.

## Step 5 — Create Execution Plan

Kemudian membuat execution plan:

```bash
terraform plan
```

Pada tahap ini Terraform menampilkan resource yang akan dibuat, diubah, atau dihapus tanpa melakukan perubahan pada infrastructure.

## Step 6 — Provision Infrastructure

Jika plan sudah sesuai, saya menjalankan:

```bash
terraform apply
```

Kemudian melakukan konfirmasi dengan mengetik:

```text
yes
```

Terraform akan membuat infrastructure AWS sesuai konfigurasi yang telah dibuat.

## Step 7 — Server Configuration Using Ansible

Setelah VM berhasil dibuat oleh Terraform, saya melanjutkan konfigurasi server menggunakan **Ansible**.
