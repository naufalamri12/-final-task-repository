***Instruction***
Staging
[ Database ]
App database using PostgreSQL
Deploy postgresql on top docker
Set the volume location in /home/$USER/storage
Allow database to remote from another server
[ Application ]
Create a Docker image for frontend & backend
Staging & Production: A lightweight docker image (as small as possible)
Building Docker image on every environment using docker multistage build
Create load balancing for frontend and backend

jawaban 

menulis kode di ansible
mengecheck apakah kita sudah install docker apa  belum dan versinya
(ansible database -i Inventory -m shell -a "docker ps") dan  (ansible database -i Inventory -m shell -a "docker --version")
setelah itu jangan lupa untuk membuka pord:5432

masuk kebackend terus ganti .env
klick  untuk migration, download file, dan menjalankannya
grep -Rni "RunMigration" .
go mod download
go run main.go
setelah itu masuk ke database dan klick dt

load balancer pertama kita menghubungkan akun 1 dan 2
jangan lupa membuka security untuk server 2
setelah itu setting ansible dan melakukan playbook
