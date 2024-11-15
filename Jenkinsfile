pipeline {
    agent {
        label 'docker'
    }

    environment {
        ECR_REPO = '866934333672.dkr.ecr.eu-west-3.amazonaws.com/guy-ecr'
        IMAGE_NAME = 'app-image'
        TAG = "${env.BRANCH_NAME}-${env.BUILD_ID}"
        AWS_REGION = "eu-west-3"
    }

    stages {
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
        }
        stage('Push to ECR') {
            steps {
                withAWS(credentials: 'aws-credentials', region: "${AWS_REGION}") {
                    sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${env.ECR_REPO}"
                    sh "docker push ${ECR_REPO}:${TAG}"
                }
            }
            post {
                success {
                    emailext(
                        subject: "Jenkins Job - Docker Image Pushed to ECR Successfully",
                        body: """
                            Hello,

                            The Docker image '${env.IMAGE_NAME
