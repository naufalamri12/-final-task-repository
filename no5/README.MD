# Private Docker Registry

## 1. Check Docker Installation

Before deploying the private Docker Registry, check whether Docker is already installed on the server.

Using Ansible:

```bash
ansible registry -i inventory -m shell -a "docker --version"
```

Check running Docker containers:

```bash
ansible registry -i inventory -m shell -a "docker ps"
```

If Docker is not installed, install Docker first using the Ansible Docker installation playbook.

## 2. Create Docker Registry Playbook

Create an Ansible playbook for deploying the private Docker Registry.

Example:

```yaml
- name: Deploy Private Docker Registry
  hosts: registry
  become: true

  tasks:
    - name: Create registry directory
      ansible.builtin.file:
        path: /home/amri/registry
        state: directory
        owner: amri
        group: amri
        mode: "0755"

    - name: Run Docker Registry
      community.docker.docker_container:
        name: registry
        image: registry:2
        state: started
        restart_policy: unless-stopped
        ports:
          - "5000:5000"
        volumes:
          - /home/amri/registry:/var/lib/registry
```

The Registry uses port `5000`.

Check the Registry container:

```bash
docker ps
```

Test the Registry:

```bash
curl http://localhost:5000/v2/
```

If the Registry is running correctly, the response should be:

```text
{}
```

## 3. Open Required Ports

Make sure the required ports are allowed in the server firewall/security group.

For the Registry:

```bash
sudo ufw allow 5000/tcp
```

If the Registry is accessed through Nginx reverse proxy, HTTP/HTTPS should also be allowed:

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

For AWS, the corresponding ports can be configured through the EC2 Security Group.

## 4. Configure Reverse Proxy

The Docker Registry must be accessible using the following domain:

```text
registry.{your_name}.studentdumbways.my.id
```

Example:

```text
registry.amri.studentdumbways.my.id
```

Create an Nginx reverse proxy configuration:

```nginx
server {
    listen 80;
    server_name registry.amri.studentdumbways.my.id;

    location / {
        proxy_pass http://127.0.0.1:5000;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Test the Nginx configuration:

```bash
sudo nginx -t
```

Restart Nginx:

```bash
sudo systemctl restart nginx
```

Then test the Registry through the domain:

```bash
curl http://registry.amri.studentdumbways.my.id/v2/
```

## 5. Push Docker Image to Private Registry

Build the Docker image first:

```bash
docker build -t frontend:latest .
```

Tag the image with the Registry address:

```bash
docker tag frontend:latest registry.amri.studentdumbways.my.id/frontend:latest
```

Push the image:

```bash
docker push registry.amri.studentdumbways.my.id/frontend:latest
```

Check the image:

```bash
curl http://registry.amri.studentdumbways.my.id/v2/_catalog
```

The Registry should return the available repositories.

## 6. Deploy Using Ansible

After the Registry configuration is ready, use Ansible to automate the deployment.

Run the playbook:

```bash
ansible-playbook -i inventory registry.yaml
```

Verify the deployment:

```bash
ansible registry -i inventory -m shell -a "docker ps"
```

The Registry container should be running:

```text
registry
```

## Result

The final architecture is:

```text
Developer
    |
    | docker push
    v
registry.amri.studentdumbways.my.id
    |
    v
Nginx Reverse Proxy
    |
    v
Docker Registry
    |
    v
Port 5000
```

The private Docker Registry is deployed on the server, exposed through the required domain, and can be used to store and pull Docker images for the staging and production CI/CD pipelines.
