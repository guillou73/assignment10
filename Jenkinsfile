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
                    sh 'apt update && apt install -y docker.io'
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
        }
        stage('Push to ECR') {
            steps {
                withAWS(credentials: 'aws-credentials', region: "${AWS_REGION}") {
                    sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}"
                    sh "docker push ${ECR_REPO}:${TAG}"
                }
            }
            post {
                success {
                    emailext(
                        subject: "Jenkins Job - Docker Image Pushed to ECR Successfully",
                        body: "Hello,\n\nThe Docker image '${env.IMAGE_NAME}:${env.TAG}' has been successfully pushed to ECR.\n\nBest regards,\nJenkins",
                        recipientProviders: [[$class: 'DevelopersRecipientProvider']],
                        to: "guymonthe2001@yahoo.fr"
                    )
                }
            }
        }
        stage('Static Code Analysis - SonarQube') {
            steps {
                script {
                    withSonarQubeEnv('SonarQubeServer') {
                        sh 'mvn sonar:sonar -Dsonar.organization=guillou73'
                    }
                }
            }
        }
        stage('Container Security Scan - Trivy') {
            steps {
                script {
                    sh "trivy --timeout 1m image ${ECR_REPO}:${TAG} > 'trivyscan.txt'"
                }
            }
            post {
                success {
                    emailext(
                        subject: "Trivy scan result",
                        body: "Hello,\n\nTrivy scan result in attachment.\n\nBest regards,\nJenkins",
                        recipientProviders: [[$class: 'DevelopersRecipientProvider']],
                        to: "guymonthe2001@yahoo.fr",
                        attachmentsPattern: 'trivyscan.txt'
                    )
                }
            }
        }
        stage('Deploy to Environment test') {
            steps {
                sshagent(['ec2-ssh-key']) {
                    sh 'echo "Starting SSH connection test"'
                    sh 'ssh -tt -o StrictHostKeyChecking=no ubuntu@51.44.25.177 ls'
                }
            }
        }
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
                        sshagent(['ec2-ssh-key']) {
                            sh """
                            ssh -tt -o StrictHostKeyChecking=no ubuntu@${targetHost} << EOF
                            aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}
                            docker pull ${ECR_REPO}:${TAG}
                            docker stop ${IMAGE_NAME} || true
                            docker rm ${IMAGE_NAME} || true
                            docker run -d --name ${IMAGE_NAME} -p 8080:8080 -p 8090:8090 ${ECR_REPO}:${TAG}
                            exit 0
EOF
                            """
                        }
                    }
                }
            }
        }
    }
    post {
        always {
            cleanWs()  // Clean up workspace after the build
        }
        success {
            emailext(
                subject: "Jenkins Job - Success Notification",
                body: "Hello,\n\nThe Jenkins job completed successfully. The Docker image '${env.IMAGE_NAME}:${env.TAG}' has been successfully pushed to ECR and deployed.\n\nBest regards,\nJenkins",
                recipientProviders: [[$class: 'DevelopersRecipientProvider']],
                to: "guyseutcheu@gmail.com"
            )
        }
        failure {
            emailext(
                subject: "Jenkins Job - Failure Notification",
                body: "Hello,\n\nThe Jenkins job failed during the process. Please check the logs for details.\n\nBest regards,\nJenkins",
                recipientProviders: [[$class: 'DevelopersRecipientProvider']],
                to: "guyseutcheu@gmail.com"
            )
        }
    }
}
