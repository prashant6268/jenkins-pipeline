# Jenkins CI/CD Pipeline with Docker on Amazon Linux 2023

This project demonstrates how to build a simple CI/CD pipeline using Jenkins, Docker, GitHub, and AWS EC2. Whenever code is pushed to GitHub, Jenkins automatically pulls the latest code, builds a Docker image, and deploys a new container.

## Project Overview

The goal of this project is to automate application deployment using Jenkins.

Workflow:

1. Developer pushes code to GitHub.
2. GitHub triggers Jenkins using a webhook.
3. Jenkins downloads the latest code.
4. Jenkins builds a Docker image.
5. Jenkins stops and removes the old container.
6. Jenkins deploys a new container.
7. The updated application becomes available on the EC2 instance.

---

## Technologies Used

* AWS EC2 (Amazon Linux 2023)
* Jenkins
* Docker
* Git & GitHub
* Apache Web Server
* Linux

---

## EC2 Server Setup

### Update the Server

```bash
sudo yum update -y
```

### Install Java

Jenkins requires Java to run.

```bash
sudo dnf install java-21-amazon-corretto -y
```

Verify installation:

```bash
java -version
```

---

## Install Jenkins

Add the Jenkins repository and install Jenkins.

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo \
https://pkg.jenkins.io/redhat-stable/jenkins.repo
```

```bash
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2026.key
```

```bash
sudo dnf install jenkins -y
```

Start Jenkins:

```bash
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

Check status:

```bash
sudo systemctl status jenkins
```

Get the initial admin password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Access Jenkins:

```text
http://<EC2-PUBLIC-IP>:8080
```

---

## Install Git

Git is required for Jenkins to pull code from GitHub.

```bash
sudo yum install git -y
```

Verify:

```bash
git --version
```

---

## Install Docker

Docker is used to package and run the application.

```bash
sudo dnf install docker -y
```

Start Docker:

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

Verify:

```bash
docker --version
```

---

## Configure Jenkins for Docker

By default, Jenkins cannot access Docker.

Add Jenkins to the Docker group:

```bash
sudo usermod -aG docker jenkins
```

Restart services:

```bash
sudo systemctl restart docker
sudo systemctl restart jenkins
```

Verify:

```bash
groups jenkins
```

Expected output:

```text
jenkins docker
```

---

## Application Files

### Dockerfile

The Dockerfile creates an Apache web server container and copies the website files into it.

```dockerfile
FROM ubuntu:latest

RUN apt-get update && \
    apt-get install -y apache2 && \
    rm -rf /var/lib/apt/lists/*

COPY index.html /var/www/html/index.html

EXPOSE 80

CMD ["apachectl", "-D", "FOREGROUND"]
```

### index.html

```html
<h1>Welcome to Jenkins CI/CD Pipeline Project</h1>
```

---

## Jenkins Pipeline

The Jenkins pipeline automates the deployment process.

```groovy
pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t my-website:latest .'
            }
        }

        stage('Stop Old Container') {
            steps {
                sh '''
                    docker stop my-website || true
                    docker rm my-website || true
                '''
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                    docker run -d \
                    --name my-website \
                    -p 8081:80 \
                    my-website:latest
                '''
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully.'
        }

        failure {
            echo 'Deployment failed.'
        }
    }
}
```

---

## GitHub Webhook Integration

To trigger Jenkins automatically after every code push:

1. Open GitHub Repository.
2. Go to **Settings → Webhooks**.
3. Click **Add Webhook**.
4. Enter:

```text
http://<JENKINS-IP>:8080/github-webhook/
```

5. Content Type:

```text
application/json
```

6. Select:

```text
Just the push event
```

7. Enable:

```text
Active
```

8. Save the webhook.

---

## Jenkins Job Configuration

1. Create a new Pipeline project.
2. Open **Configure**.
3. Select **Pipeline Script from SCM**.
4. Choose **Git**.
5. Enter repository URL.
6. Select the main branch.
7. Enable:

```text
GitHub hook trigger for GITScm polling
```

8. Save the configuration.

---

## Deployment Verification

After a successful build:

Jenkins:

```text
http://<EC2-PUBLIC-IP>:8080
```

Website:

```text
http://<EC2-PUBLIC-IP>:8081
```

---

## Common Issues and Solutions

### Port 80 Already in Use

Check:

```bash
sudo ss -tulpn | grep :80
```

If Apache is running:

```bash
sudo systemctl stop httpd
```

### Port 8080 Already in Use

Jenkins uses port 8080 by default.

Deploy the application on another port such as:

```text
8081
```

### Check Running Containers

```bash
docker ps
```

### View Container Logs

```bash
docker logs my-website
```

### Remove Container

```bash
docker rm -f my-website
```

---

## Learning Outcomes

Through this project, you will learn:

* Jenkins installation and configuration
* GitHub integration with Jenkins
* Docker image creation
* Container deployment
* CI/CD pipeline creation
* Webhook automation
* AWS EC2 management
* Linux administration basics

## Author

**Prashant Tiwari**

DevOps Enthusiast | AWS | Docker | Jenkins | CI/CD
