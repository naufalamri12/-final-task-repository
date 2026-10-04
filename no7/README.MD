***Instruction***

[ Testing ]

On your cicd workflow, use sonarqube for testing your quality code.

You can use others testing tools like trivy for scanning your images or some.

And try test your running app by using wget spider

jawaban
pergi ke docker file Jenkins
setelah itu ganti
sebelumnya
apt-get install -y ca-certificates curl gnupg && \
menjadi
apt-get install -y ca-certificates curl wget gnupg && \
setelah itu coba testing lagi

opsinal jika ngeleg naikan vm ke m7i-flex.large dengan 2 vCPU and 8 GB memory

untuk sonarkube, bisa mencari package diweb
setelah itu masukan ke ansible
setelah di playbook bisa langsung docker ps
kalua sudah langsung akses web untuk awal id dan pssa admin
