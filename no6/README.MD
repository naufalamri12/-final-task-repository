Instructions
[ CI/CD ]
Using Jenkins, create a pipeline running:
Repository pull
Image build
Testing Code
Push Image into your own docker registry private
SSH into your server
Pull image from docker registry private
Redeploy your deployment apps
For testing Stage, u must use sonarqube for testing your code quality

jawaban
menulis kode di ansible dan ngecheck setelah jalan
kita docker ps  ( semisalnya sudah berjalan biar ngecheck diweb)
setelah itu docker exec jenkins docker pull 15.232.52.124:5000/dumbmerch-frontend:staging
setela itu mencari password dan masukan username dan password
setelah login pergi ke global dan setting password GitHub dan ssh
setelah itu pergi ke configuration
terus membuat ssh di monitor dan dimasukan di authorized apps
at ~/.ssh/jenkins_appserver dimasukan ke website Jenkins
jangan lupa membuat dockerignore untuk backend dan frontend
