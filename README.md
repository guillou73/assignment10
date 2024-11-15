# CI/CD Pipeline with Jenkins, Docker, and AWS

This project demonstrates a complete CI/CD pipeline setup using Jenkins running as a Docker container on an AWS EC2 instance (Jenkins master), with a Jenkins agent (slave) on a separate EC2 instance to execute build tasks. The pipeline automates code checkout, Docker image building, security scanning, testing, and multi-environment deployment (DEV, STAGING, PROD) to EC2 instances. It also features secure AWS IAM role-based access and email notifications upon successful image push to Amazon ECR.

## 📋 Project Overview

### Pipeline Stages:
1. **Checkout Code**: Pulls the code from the respective branch (dev, staging, or main).
2. **Build Docker Image**: Builds a Docker image from the application code.
3. **Push to Amazon ECR**: Pushes the built Docker image to an Amazon ECR repository.
4. **Static Code Analysis**: Analyzes the code using SonarQube for code quality.
5. **Container Security Scan**: Scans the Docker image for vulnerabilities using Trivy.
6. **Deployment**: Deploys the Docker container to the target EC2 instance based on the branch (dev, staging, or main).
7. **Email Notification**: Sends an email notification after a successful image push to ECR.

### Technologies Used:
- **CI/CD**: Jenkins (Master in Docker on EC2, Slave on another EC2)
- **Containerization**: Docker
- **Source Control**: GitHub (with branch protection and PR-based merging)
- **Container Registry**: Amazon ECR
- **Secrets Management**: AWS Secrets Manager
- **Security Scanning**: Trivy, SonarQube
- **Notifications**: Email (Jenkins Email Extension Plugin)

## 🚀 Getting Started

### Prerequisites:
- AWS Account with access to IAM, EC2, and ECR services.
- GitHub Account for source code management.
- AWS CLI configured on the Jenkins master instance.
- SSH access to EC2 instances.

## 🏗️ Project Setup

### Step 1: GitHub Repository Setup
1. Create a GitHub repository and add the application code, `Dockerfile`, and `Jenkinsfile`.
2. Create the following branches:
   - `dev`: Development branch.
   - `staging`: Pre-production branch.
   - `main`: Production branch.
3. Set up branch protection rules:
   - Require pull request reviews before merging.
   - Enable status checks (e.g., CI/CD checks) before merging.

### Step 2: Jenkins Master and Agent Setup

#### Jenkins Master Setup
1. Launch an EC2 instance (Amazon Linux 2 or Ubuntu) for Jenkins master.
2. Install Docker and start Jenkins in a Docker container:
   ```bash
   sudo yum update -y
   sudo yum install docker -y
   sudo service docker start
   sudo usermod -aG docker ec2-user
   docker pull jenkins/jenkins:lts
   docker run -d -p 8080:8080 -p 50000:50000 --name jenkins-master \
   -v jenkins_home:/var/jenkins_home \
   -v /var/run/docker.sock:/var/run/docker.sock \
   jenkins/jenkins:lts
# assignment10

