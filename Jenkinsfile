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
        stage('Git verify') {
            steps {
                script{
                    sh 'git --version'
                } //script
            } //steps
        } //stage
        stage('Checkout') {
            steps {
                git branch: "${env.main}", url: 'https://github.com/guillou73/assignment10.git', credentialsId: 'jenkins-pat'
            } //steps
        } //stage
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${env.ECR_REPO}:${env.TAG}")
                } //script
            } //steps
        } //stage
        stage('Push to ECR') {
            steps {
                withAWS(credentials: 'aws-credentials', region: "${eu-west-2}")
                {
                    sh "aws ecr get-login-password --region ${eu-west-2} | docker login --username AWS --password-stdin ${env.866934333672.dkr.ecr.eu-west-3.amazonaws.com/guy-ecr }"
                    sh "docker push ${env.866934333672.dkr.ecr.eu-west-3.amazonaws.com/guy-ecr }:${env.TAG}"
                } //withAWS
            } //steps
                post {
                success {
                    // Send email notification after successful image push to ECR
                    emailext(
                        subject: "Jenkins Job - Docker Image Pushed to ECR Successfully",
                        body: "Hello,\n\nThe Docker image '${env.IMAGE_NAME}:${env.TAG}' has been successfully pushed to ECR.\n\nBest regards,\nJenkins",
                        recipientProviders: [[$class: 'DevelopersRecipientProvider']],
                        to: "guymonthe2001@yahoo.fr"
                    )
                } //success
            } //post
        } //stage
        stage('Static Code Analysis - SonarQube') {
            steps {
                script {
                    withSonarQubeEnv('SonarQubeServer') {
                        sh 'mvn sonar:sonar -Dsonar.organization=guillou73'
                    } //withSonarQubeEnv
                } //script
            } //steps
        } //stage

        stage('Container Security Scan - Trivy') {
            steps {
                script {
                    sh "trivy --timeout 1m image ${ECR_REPO}:${TAG} > 'trivyscan.txt'"
                } //script
            } //steps
            post {
                success{
                    emailext(
                        subject: "Trivy scan result",
                        body: "Hello, \n Trivy scan result in attachment \n Best regards, \n Jenkins \n ",
                        recipientProviders: [[$class: 'DevelopersRecipientProvider']],
                        to: "guymonthe2001@yahoo.fr",
                        attachmentsPattern: 'trivyscan.txt'
                    )
                } //success
            } //post
        } //stage

        stage('Deploy to Environment test') {
            steps {
                    sshagent(['ec2-ssh-key']) {
                    sh 'echo "Starting SSH connection test"'
                    sh 'ssh -tt -o StrictHostKeyChecking=no ubuntu@51.44.25.177 ls'
                } //sshagent
            } //steps
        } //stage

        stage('Deploy to Environment') {
            steps {
                script {
                    def targetHost = ''
                    if (env.BRANCH_NAME == 'docker1') {
                        targetHost = '15.188.246.138'
                    } else if (env.BRANCH_NAME == 'docker2') {
                        targetHost = '15.237.143.77'
                    } else if (env.BRANCH_NAME == 'docker3') {
                        targetHost = '51.44.25.177'
                    } else if (env.BRANCH_NAME == 'main') {
                        targetHost = '51.44.25.177'
                    }
                    withCredentials([usernamePassword(credentialsId: 'aws-ecr', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sshagent(['ec2-ssh-key']){
                    sh """
                    ssh -tt -o StrictHostKeyChecking=no ubuntu@${targetHost} << EOF
                    aws ecr get-login-password --region eu-west-2 | docker login --username AWS --password-stdin ${env.ECR_REPO}
                    docker pull ${ECR_REPO}:${TAG}
                    docker stop ${IMAGE_NAME} || true
                    docker rm ${IMAGE_NAME} || true
                    docker run -d --name ${IMAGE_NAME} -p 8080:8080 -p 8090:8090 ${ECR_REPO}:${TAG}
                    exit 0
EOF
"""
                    } //withCredentials
                    } //sshagent
                } //script
            } //steps
        } //stage (Deploy to Environment)
    } //stages
    post {
        always {
            cleanWs()  // Clean up workspace after the build
        } //always
        success {
            emailext(
                subject: "Jenkins Job - Docker Image Pushed to ECR Successfully",
                body: "Hello,\n\nThe Docker image '${env.IMAGE_NAME}:${env.TAG}' has been successfully pushed to ECR.\n\nBest regards,\nJenkins",
                recipientProviders: [[$class: 'DevelopersRecipientProvider']],
                to: "guyseutcheu@gmail.com"
            )
        } //success
        failure {
            emailext(
                subject: "Jenkins Job - Failure Notification",
                body: "Hello,\n\nThe Jenkins job failed during the process. Please check the logs for details.\n\nBest regards,\nJenkins",
                recipientProviders: [[$class: 'DevelopersRecipientProvider']],
                to: "guyseutcheu@gmail.com"
            )
        } //failure
    } //post
} //pipeline
