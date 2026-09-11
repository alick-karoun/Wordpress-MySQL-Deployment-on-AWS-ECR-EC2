# AWS Containerized Application Deployment (ECR + EC2 + Docker)

A complete DevOps hands-on project demonstrating containerization, container registry management on AWS ECR, and cloud hosting on AWS EC2 with Docker bridge networking.

---

# Architecture Overview

1. **Local Development**: Built application container image using Docker Desktop from a cloned github repo.
2. **AWS ECR (Elastic Container Registry)**: Pushed compiled image to a private cloud registry in `eu-north-1`.
3. **AWS EC2 Hosting**:
   - made Linux EC2 instance.
   - Installed Docker Engine.
   - Created isolated Docker bridge network (`wp-network`).
   - Provisioned persistent **MySQL 5.7** database container.
   - Deployed custom application container connected to MySQL and exposed on port `8080`.

---

## 🚀 Step-by-Step Deployment Guide

### Phase 1: Local Image Build & ECR Push

``bash
# 1. Authenticate Docker CLI to AWS ECR
aws ecr get-login-password --region eu-north-1 | docker login --username AWS --password-stdin <ACCOUNT_ID>.dkr.ecr.eu-north-1.amazonaws.com

# 2. Build local Docker image
docker build -t aca-wordpress .

# 3. Tag image for ECR
docker tag aca-wordpress:latest <ACCOUNT_ID>[.dkr.ecr.eu-north-1.amazonaws.com/aca-wordpress:latest](https://.dkr.ecr.eu-north-1.amazonaws.com/aca-wordpress:latest)

# 4. Push image to AWS ECR
docker push <ACCOUNT_ID>[.dkr.ecr.eu-north-1.amazonaws.com/aca-wordpress:latest](https://.dkr.ecr.eu-north-1.amazonaws.com/aca-wordpress:latest)

--

### Phase 2 Inside the EC2:
# 1. Connect to EC2 via SSH and install Docker
sudo apt-get update -y && sudo apt-get install -y docker.io
sudo usermod -aG docker $USER
newgrp docker

# 2. Create Docker Network
docker network create wp-network

# 3. Launch Persistent MySQL Container
docker run -d \
  --name mysql-db \
  --network wp-network \
  -e MYSQL_ROOT_PASSWORD=somewordpress \
  -e MYSQL_DATABASE=wordpress \
  -e MYSQL_USER=wordpress \
  -e MYSQL_PASSWORD=wordpress \
  -v mysql_data:/var/lib/mysql \
  mysql:5.7

# 4. Authenticate EC2 to ECR & Launch Application Container
aws ecr get-login-password --region eu-north-1 | docker login --username AWS --password-stdin <ACCOUNT_ID>.dkr.ecr.eu-north-1.amazonaws.com

docker run -d \
  -p 8080:5000 \
  --name running-app \
  --network wp-network \
  -e WORDPRESS_DB_HOST=mysql-db:3306 \
  -e WORDPRESS_DB_USER=wordpress \
  -e WORDPRESS_DB_PASSWORD=wordpress \
  -e WORDPRESS_DB_NAME=wordpress \
  <ACCOUNT_ID>[.dkr.ecr.eu-north-1.amazonaws.com/aca-wordpress:latest](https://.dkr.ecr.eu-north-1.amazonaws.com/aca-wordpress:latest)
