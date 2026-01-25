##  Project Overview

end-to-end DevOps pipeline for a full-stack application using
*  Backend: Django 6.0 (REST API)
*  Frontend: React (Vite, TypeScript)

The implementation covers:

* Docker multi-stage builds
* Non-root containers
* Service orchestration with Docker Compose
* CI/CD using GitHub Actions
* Automated deployment to AWS EC2

Repository: [https://github.com/Nexgensis/devops-assessment](https://github.com/Nexgensis/devops-assessment)

---

##  Architecture

``` bash
User
  │
  ▼
React (Nginx, Port 3000)
  │  API Calls (VITE_API_URL)
  ▼
Django REST API (Port 8000)
```

* Docker network enables service-to-service communication
* Environment variables control runtime behavior

---

## 🛠 Prerequisites

* Python 3.10+
* Node.js 18+
* npm 9+
* Git
* Docker installed
* Docker Compose installed
* Port 22, 3000, 8000, 80 open in Security Group

---

Create & Use Non-Root User 
For security, the application is deployed using a **non-root user** named `devops`.

```bash
useradd devops
passwd devops
```
- set password and add user in /etc/sudoers

> ⚠️ Logout & login again to apply Docker group permissions.

All application files and containers are managed using the `devops` user.

switch to devops user
```bash
sudo su - devops
```

allow docker without sudo
```bash
sudo gpasswd -a devops dockers
```

---

##  Setup Guide

### 1️⃣ Dockerize: Created separate multistage-Dockerfile for the Frontend and Backend and defined environment variables

---

#### Backend (`backend/.env`)

```env
DEBUG=True
DJANGO_ALLOWED_HOSTS=*
```

#### Frontend (`frontend/.env`)

```env
VITE_API_URL=http://django-backend:8000
```

---

### 2️⃣ Orchestration: Created a docker-compose.yml file to run both services. 

``` bash
git clone https://github.com/rishidhotare/devops-assessment.git
cd devops-assessment
```
### 3️⃣ Run Locally Using Docker Compose

```bash
docker-compose up -d --build
```

#### Access Services

* Frontend: http://43.204.36.124:3000/
* Backend API: http://43.204.36.124:8000/api/hello/
* Backend admin : http://43.204.36.124:8000/admin/

---

### 4️⃣ Verify Containers

``` bash
docker ps
```

Expected:

* django-backend running on port 8000
* react-frontend running on port 3000

---

## ☁️ Phase 2: CI/CD Pipeline

Created a GitHub Actions workflow that triggers on every push to the main branch:  

###  GitHub Actions CI/CD Flow for 
1. Build & Push: Build Docker images and pushed in my dockerhub repository.

---

Workflow file: `.github/workflows/ci-cd.yml`

* Triggered automatically on push to main branch
* Checks out latest code from GitHub
* Logs in to Docker Hub using secrets
* Builds & pushes Django backend Docker image
* Builds & pushes React frontend Docker image
* Connects to AWS EC2 via SSH
* Pulls latest code and Docker images on EC2
* Stops existing containers using Docker Compose
* Deploys updated containers using Docker Compose
* Ensures fully automated build, push, and deployment

---

### 2. Deployment:  Automate the deployment to your environment

Made chages in  `.github/workflows/ci-cd.yml` file


    set -e
    cd /home/devops/devops-assessment
	git pull origin main
	
    # for pull images from my dockerhub registy
	
    docker pull ${{ secrets.DOCKERHUB_USERNAME }}/django-backend:latest
    docker pull ${{ secrets.DOCKERHUB_USERNAME }}/react-frontend:latest

    # delete all up containers
    docker-compose down 

    #Automatically deploy to a Cloud provider
    docker-compose up -d 	

---

##  Validation Checklist

* [x] Containers run as non-root users
* [x] Multi-stage Docker builds
* [x] Frontend communicates with backend
* [x] CI/CD pipeline triggers on push
* [x] Automated EC2 deployment


## 🐞 Troubleshooting Log

### Issue 1: React App Could Not Reach Django Backend

**Problem:**
Frontend failed with network error while calling the backend API.

**Root Cause:**
Used `localhost` inside App.txs. Containers cannot access host `localhost`.

**Error:** 
> const response = await axios.get('http://localhost:8000/api/hello/')

**Solution:**
Updated API URL to use Docker service name:
> const response = await axios.get('http://43.204.36.124:8000/api/hello/')

### Issue 2: SSH Deployment Failing in GitHub Actions

**Problem:**
GitHub Actions failed to write SSH key.

**Root Cause:**
`.ssh` directory missing on EC2.

**Solution:**

```bash
sudo vi devops.pem ##added my EC2 machine key content
sudo chmod 400 devops.pem
ssh-keygen -t rsa -b 4096 -f devops
echo "ssh-rsa - devops.pub key content" >> ~/.ssh/authorized_keys
sudo chmod 600 ~/.ssh/authorized_keys
```

#### Important Note - Make sure your machine key content same as your github action secret key.


### GitHub Action Secret Environments which I set :

* DOCKERHUB_PASSWORD
* DOCKERHUB_USERNAME
* EC2_HOST
* EC2_KEY
* EC2_USER
