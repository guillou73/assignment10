pipeline {
    agent {
        label 'docker'
    }

    environment {
        ECR_REPO = '866934333672.dkr.ecr.eu-west-3.amazonaws.com/guy-ecr'
        IMAGE_NAME = 'ecr-image'
        TAG = "${env.BRANCH_NAME}-${env.BUILD_ID}"
        AWS_REGION = "eu-west-2"
    }

    stages {
        stage('Prepare Environment') {
            steps {
                script {
                    sh '''
                        sudo apt update && sudo apt install -y docker.io awscli
                        if id "jenkins" &>/dev/null; then
                            echo "User jenkins exists"
                        else
                            echo "User jenkins does not exist. Creating user."
                            sudo useradd -m -s /bin/bash jenkins
                            sudo passwd -d jenkins
                        fi
                        sudo usermod -aG docker jenkins
                        sudo systemctl restart jenkins
                        sudo chmod 666 /var/run/docker.sock
                    '''
                }
            }
        }
        stage('Git verify') {
            steps {
                script {
                    sh 'git --version'
                }
            }
        }
        stage('Checkout') {
            steps {
                git branch: "${env.BRANCH_NAME}", url: 'https://github.com/guillou73/assignment10.git', credentialsId: 'jenkins-pat'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${env.ECR_REPO}:${env.TAG}")
                }
            }
