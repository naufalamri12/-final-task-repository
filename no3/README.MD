## Server User and SSH Configuration

First, install Ansible on the management server:

```bash
sudo apt update
sudo apt install ansible -y
```

Check the Ansible installation:

```bash
ansible --version
```

### 1. Create New User

Create a new user named:

```text
finaltask-$USER
```

The new user is used as the main user for the final task playground.

The user can be created manually:

```bash
sudo adduser finaltask-$USER
```

Or using Ansible by creating a YAML playbook.
### 2. SSH Key Authentication

Generate one SSH key that will be used for all purposes:

```bash
ssh-keygen -t ed25519
```

The public key is:

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the public key to all servers.

The same SSH key is used for:

- Server SSH access
- GitHub/GitLab repository access
- CI/CD
- Ansible
- Deployment

This allows all servers and automation processes to use a single SSH key.

### 3. SSH Password Authentication

The server is configured to allow login using both:

- SSH key authentication
- Password authentication

For password authentication, configure:

```bash
sudo nano /etc/ssh/sshd_config
```

Make sure the following configuration is enabled:

```text
PasswordAuthentication yes
PubkeyAuthentication yes
```

After changing the configuration:

```bash
sudo systemctl restart ssh
```

### 4. SSH Config

Create an SSH configuration file:

```bash
nano ~/.ssh/config
```

Example:


Set the correct permission:

```bash
chmod 600 ~/.ssh/config
```

After this configuration, servers can be accessed using the host alias:

```bash
ssh staging-frontend
```

instead of typing the complete SSH command every time.

### 5. Change SSH Port

Change the default SSH port from `22` to `3333`.

Edit:

```bash
sudo nano /etc/ssh/sshd_config
```

Change:

```text
#Port 22
```

to:

```text
Port 3333
```

Then restart SSH:

```bash
sudo systemctl restart ssh
```

Test the new SSH port:

```bash
ssh -p 3333 finaltask-$USER@<SERVER_IP>
```

**Important:** Do not close the current SSH session until the new port has been tested successfully.

### 6. UFW Firewall

Enable UFW and allow only the ports required by the server.

First, allow the new SSH port:

```bash
sudo ufw allow 3333/tcp
```

For a web server, allow HTTP and HTTPS:

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

Check the firewall rules:

```bash
sudo ufw status numbered
```

Enable UFW:

```bash
sudo ufw enable
```

Only the ports required by each server are opened.

For AWS, the same restriction can also be configured using **Security Groups**, allowing only the required ports for each server.

### 7. Verify Server Access

Finally, test the server using the SSH configuration:

```bash
ssh staging-frontend
```

Or directly:

```bash
ssh -i ~/.ssh/id_ed25519 -p 3333 finaltask-$USER@<SERVER_IP>
```

The final configuration provides:

- A dedicated `finaltask-$USER` user.
- SSH key and password authentication.
- One SSH key for repository, CI/CD, Ansible, and server access.
- An SSH config for easier server login.
- SSH running on port `3333` instead of `22`.
- UFW enabled with only required ports allowed.
- Server access ready for the final task deployment.
                                    
