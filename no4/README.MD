intructions
[ Docker Registry ]
Deploy Docker Registry Private on this server
Push your image into Your Own Docker Registry
reverse proxy for docker registry was registry.{your_name}.studentdumbways.my.id


jawaban 
setting diansible dan mengarahkan semuanya ke vm
setelah itu kita melakukan git clone, git build
jangan luoa membuat /etc/docker/daemon.json dan diisi {
  "insecure-registries": [
    "15.232.52.124:5000"
  ]
}

setelah itu docker push 15.232.52.124:5000/frontend:latestst
untuk mengecheck image http://15.232.52.124:5000/v2/_catalog
untuk melihat tags http://15.232.52.124:5000/v2/frontend/tags/list

setelah itu kita masuk ke cloudfire
dan mengganti /etc/nginx/sites-available/registry dengan server tujuan kita
