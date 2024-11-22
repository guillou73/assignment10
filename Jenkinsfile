pipeline {
    agent { label 'ec2-slave' }

    environment {
        ECR_REPO = 'your-ecr-repo-url'
        IMAGE_NAME = 'app-image'
        TAG = "${env.BRANCH_NAME}-${env.BUILD_ID}"
        SSH_KEY = credentials('ec2-ssh-key')
    }
