# AWS EC2 Deployment Guide

This guide details the steps to deploy the MEAN stack application on an AWS EC2 instance.

## 1. Launch an EC2 Instance

1.  **Log in to AWS Console**: Navigate to the EC2 Dashboard.
2.  **Launch Instance**: Click the orange "Launch instances" button.
3.  **Name**: Give your instance a name (e.g., `MeanStackApp`).
4.  **Application and OS Images (AMI)**:
    *   Select **Ubuntu**.
    *   Choose **Ubuntu Server 22.04 LTS (HVM), SSD Volume Type** (Free tier eligible).
5.  **Instance Type**:
    *   Select **t2.micro** (Free tier eligible) or **t3.small** (if you need more performance).
6.  **Key Pair (Login)**:
    *   Click **Create new key pair**.
    *   Name: `mean-app-key`.
    *   Key pair type: **RSA**.
    *   Private key file format: **.pem**.
    *   Click **Create key pair**. **IMPORTANT**: The `.pem` file will download automatically. Save this file safely; you cannot download it again.
7.  **Network Settings**:
    *   Click **Edit** (optional, but good to verify).
    *   **Security Group**: Create security group.
    *   **Inbound Security Group Rules**:
        *   Type: **ssh** | Protocol: **TCP** | Port: **22** | Source: **Anywhere** (0.0.0.0/0) or My IP.
        *   Type: **HTTP** | Protocol: **TCP** | Port: **80** | Source: **Anywhere** (0.0.0.0/0).
        *   (Optional) Type: **Custom TCP** | Port: **8080** | Source: **Anywhere** (If you want to access backend directly for testing).
8.  **Launch**: Click **Launch instance**.

## 2. Connect to Your Instance

1.  Open your terminal on your local machine.
2.  Navigate to the folder where you saved `mean-app-key.pem`.
3.  Change the permissions of the key file to be read-only:
    ```bash
    chmod 400 mean-app-key.pem
    ```
4.  Copy the **Public IPv4 address** of your instance from the AWS Console.
5.  Connect via SSH:
    ```bash
    ssh -i "mean-app-key.pem" ubuntu@<YOUR_EC2_PUBLIC_IP>
    ```
    *Type `yes` when asked about authenticity.*

## 3. Install Docker on EC2

Once connected to the EC2 instance (inside the SSH session), run the following commands block by block:

```bash
# 1. Update package index
sudo apt-get update

# 2. Install prerequisite packages
sudo apt-get install -y ca-certificates curl gnupg lsb-release

# 3. Add Docker’s official GPG key
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 4. Set up the repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 5. Install Docker Engine and Docker Compose
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-compose

# 6. Add the 'ubuntu' user to the docker group
sudo usermod -aG docker ubuntu
```

**CRITICAL STEP**: After running the above commands, you must **disconnect and reconnect** for the group changes to take effect.
*   Type `exit` to disconnect.
*   Run the SSH command again to reconnect.
*   Verify installation by running `docker run hello-world`.

## 4. Configure GitHub Secrets for CI/CD

Now, link your GitHub repository to this EC2 instance.

1.  Go to your GitHub Repository.
2.  Navigate to **Settings** > **Secrets and variables** > **Actions**.
3.  Click **New repository secret** and add the following:

| Secret Name | Value |
| :--- | :--- |
| `HOST_IP` | The **Public IPv4 address** of your EC2 instance. |
| `SSH_USER` | `ubuntu` |
| `SSH_KEY` | Open your `mean-app-key.pem` file in a text editor. Copy the **entire content** (including `-----BEGIN RSA PRIVATE KEY-----` and `-----END...`) and paste it here. |
| `DOCKER_USERNAME` | Your Docker Hub username. |
| `DOCKER_PASSWORD` | Your Docker Hub password (or Access Token). |

## 5. Trigger Deployment

1.  Make a small change to your code (e.g., update README) or simply re-run the latest workflow in the **Actions** tab on GitHub.
2.  Watch the pipeline:
    *   It will build the images.
    *   Push them to Docker Hub.
    *   Log in to your EC2 instance.
    *   Pull the images and start the containers.

## 6. Verify Deployment

1.  Open your web browser.
2.  Go to `http://<YOUR_EC2_PUBLIC_IP>`.
3.  You should see your Angular application running!
