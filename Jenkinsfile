pipeline {
    agent { label 'docker' }

    environment {
        ECR_REPO = '464787010492.dkr.ecr.eu-west-3.amazonaws.com/main/guy-repo'
        IMAGE_NAME = 'app-image'
        TAG = "${env.BRANCH_NAME}-${env.BUILD_ID}"
        SSH_KEY = credentials('root-ssh-key') // Corrected SSH key reference
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${env.BRANCH_NAME}", url: 'https://github.com/guillou73/assignment10.git'
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
                withCredentials([usernamePassword(credentialsId: 'aws-ecr', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh """
                        aws ecr get-login-password --region eu-west-3 | docker login --username AWS --password-stdin ${env.ECR_REPO}
                        docker push ${env.ECR_REPO}:${env.TAG}
                    """
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
                        to: "guymonthe2001@yahoo.fr"
                    )
                }
            }
        }

        stage('Static Code Analysis - SonarQube') {
            steps {
                script {
                    withSonarQubeEnv('SonarQubeServer') {
                        sh 'mvn sonar:sonar' // Corrected Maven SonarQube command
                    }
                }
            }
        }
    }
}
