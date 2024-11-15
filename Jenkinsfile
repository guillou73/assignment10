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

                            The Docker image '${env.IMAGE_NAME}:${env.TAG}' has been successfully pushed to ECR.

                            Best regards,
                            Jenkins
                        """,
                        recipientProviders: [[$class: 'DevelopersRecipientProvider']],
                        to: "m.ehtasham.azhar@gmail.com"
                    )//
                }//
            }//
        }//
        stage('Static Code Analysis - SonarQube') {
            steps {//
                script {//
                    withSonarQubeEnv('SonarQubeServer') {
                        sh 'mv sonar:sonar'
                    }//
                }//
            }//
        }//
        stage('Container Security Scan - Trivy') {
            steps {//
                script {//
                    sh "trivy --timeout 1m image ${ECR_REPO}:${TAG} > 'trivyscan.txt'"
                }//
            }//
            post {//
                success {//
                    emailext(//
                        subject: "Trivy scan result",
                        body: "Hello,\nTrivy scan result in attachment.\nBest regards,\nJenkins",
                        recipientProviders: [[$class: 'DevelopersRecipientProvider']],
                        to: "m.ehtasham.azhar@gmail.com",
                        attachmentsPattern: 'trivyscan.txt'
                    )
                }
            }
        }
        stage('Deploy to Environment test') {
            steps {
                sshagent(['jenkins-agent']) {
                    sh 'echo "Starting SSH connection test"'
                    sh 'ssh -tt -o StrictHostKeyChecking=no ubuntu@15.188.246.138 ls'
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
                        sshagent(['jenkins-agent']) {
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
                body: """
                    Hello,

                    The Jenkins job completed successfully. The Docker image '${env.IMAGE_NAME}:${env.TAG}' has been successfully pushed to ECR and deployed.

                    Best regards,
                    Jenkins
                """,
                recipientProviders: [[$class: 'DevelopersRecipientProvider']],
                to: "m.ehtasham.azhar@gmail.com"
            )
        }
        failure {
            emailext(
                subject: "Jenkins Job - Failure Notification",
                body: """
                    Hello,

                    The Jenkins job failed during the process. Please check the logs for details.

                    Best regards,
                    Jenkins
                """,
                recipientProviders: [[$class: 'DevelopersRecipientProvider']],
                to: "m.ehtasham.azhar@gmail.com"
            )
        }
    }
}
