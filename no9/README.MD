Instructions

Create domains:

[ Monitoring ]

exporter.<name>.studentdumbways.my.id - Node Exporter

prom.<name>.studentdumbways.my.id - Prometheus

monitoring.<name>.studentumbways.my.id - Grafana

[ Docker Registry ]

registry.<name>.studentdumbways.my.id - Docker Registry

[ Staging ]

staging.<name>.studentdumbways.my.id - App

api.staging.<name>.studentdumbways.my.id - Backend API

[ Production ]

<name>.studentdumbways.my.id - App

api.<name>.studentdumbways.my.id - Backend API

All domains are HTTPS

Create Bash Script for Automatic renewal for Certificates

jawaban
buat file di ansible 
ansible-vault encrypt vault.yml
New Vault password:
Confirm New Vault password:
Encryption successful


