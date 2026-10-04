## Create Repository on GitHub

1. Check whether Git is already installed:

```bash
git --version
```

If Git is not installed, install it with:

```bash
sudo apt update
sudo apt install git -y
```

2. Create a **Private Repository** on GitHub.

3. Add the server/local SSH key to GitHub so the repository can be accessed securely.

4. Configure Git username and email:

```bash
git config --global user.name "naufalamri12"
git config --global user.email "your-email@example.com"
```

5. Clone the repository:

```bash
git clone git@github.com:naufalamri12/nama-repository.git
cd nama-repository
```

6. Create two branches:

```bash
git branch staging
git branch production
```

Or create and switch to the staging branch:

```bash
git checkout -b staging
```

Then create the production branch:

```bash
git checkout -b production
```

The repository has two main branches:

- `staging` → used for staging/development and testing.
- `production` → used for the production environment.

## Docker Compose

Create a `docker-compose.yml` file to define the application services and their configuration.

Example:

```bash
nano docker-compose.yml
```

After creating the configuration, test it with:

```bash
docker compose config
```

Then run:

```bash
docker compose up -d
```

Check the running containers:

```bash
docker ps
```

## SSH Server Testing

Test SSH access to the server using the SSH private key:

```bash
ssh -i ~/.ssh/id_ed25519 ubuntu@15.232.126.53
```

If the connection is successful, the server is ready to be used for the deployment process.

## CI/CD for Each Branch

Each branch has its own CI/CD pipeline.

### Staging

The `staging` branch is connected to the **staging CI/CD pipeline**.

When code is pushed to the `staging` branch:

```bash
git checkout staging
git add .
git commit -m "update staging"
git push origin staging
```

The CI/CD pipeline will automatically:

1. Checkout the source code.
2. Build the Docker image.
3. Run tests/checks.
4. Push the image to the Docker Registry.
5. Deploy the application to the staging server.

### Production

The `production` branch is connected to the **production CI/CD pipeline**.

When code is pushed to the `production` branch:

```bash
git checkout production
git add .
git commit -m "deploy production"
git push origin production
```

The CI/CD pipeline will automatically:

1. Checkout the source code.
2. Build the Docker image.
3. Run tests/checks.
4. Push the image to the Docker Registry.
5. Deploy the application to the production server.

Therefore, the `staging` and `production` environments have separate branches and separate CI/CD processes.
