# MEAN Stack CRUD Application Deployment

This repository contains the source code and deployment configuration for a MEAN (MongoDB, Express, Angular, Node.js) stack application.

## Project Structure

- `frontend/`: Angular 15 application.
- `backend/`: Node.js/Express application.
- `docker-compose.yml`: Orchestration for local and production deployment.
- `.github/workflows/ci-cd.yml`: GitHub Actions pipeline for automated deployment.

## Prerequisites

- Docker & Docker Compose
- Node.js (for local development)
- A GitHub account
- A Docker Hub account
- An Ubuntu Virtual Machine (AWS EC2, Azure VM, etc.)

## Setup Instructions

### 1. Repository Setup

1.  Create a new repository on GitHub.
2.  Initialize git in this folder and push the code:
    ```bash
    git init
    git add .
    git commit -m "Initial commit"
    git branch -M main
    git remote add origin <YOUR_GITHUB_REPO_URL>
    git push -u origin main
    ```

### 2. Docker Hub Setup

1.  Create two repositories in your Docker Hub:
    - `mean-backend`
    - `mean-frontend`

### 3. VM Setup (Ubuntu)

1.  Provision an Ubuntu VM on your cloud provider.
2.  SSH into the VM and install Docker & Docker Compose:
    ```bash
    sudo apt-get update
    sudo apt-get install -y ca-certificates curl gnupg
    sudo mkdir -m 0755 -p /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    echo \
      "deb [arch=\"$(dpkg --print-architecture)\" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update
    sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-compose
    ```
3.  Ensure your user has permission to run Docker:
    ```bash
    sudo usermod -aG docker $USER
    ```
    (Log out and log back in for this to take effect).

### 4. GitHub Secrets

Go to your GitHub Repository -> Settings -> Secrets and variables -> Actions -> New repository secret. Add the following:

- `DOCKER_USERNAME`: Your Docker Hub username.
- `DOCKER_PASSWORD`: Your Docker Hub access token or password.
- `HOST_IP`: The public IP address of your VM.
- `SSH_USER`: The username to SSH into your VM (e.g., `ubuntu`).
- `SSH_KEY`: The private SSH key (content of `.pem` file) to access your VM.

### 5. Deployment

Once the secrets are set, any push to the `main` branch will trigger the CI/CD pipeline.

1.  **Build**: GitHub Actions will build the Docker images for frontend and backend.
2.  **Push**: Images will be pushed to Docker Hub.
3.  **Deploy**: The workflow will SSH into your VM, copy the `docker-compose.yml`, pull the new images, and restart the containers.

## Accessing the Application

Open your browser and navigate to `http://<YOUR_VM_IP>`.
- The Angular frontend is served on port 80.
- API requests are proxied to the backend via Nginx.

## Local Development

To run the application locally using Docker Compose:

```bash
docker-compose up --build
```

Access the app at `http://localhost`.
