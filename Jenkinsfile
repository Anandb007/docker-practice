pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO = "github-jenkins-ecr"
        ACCOUNT_ID = sh(
            script: "aws sts get-caller-identity --query Account --output text",
            returnStdout: true
        ).trim()
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_URI = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $ECR_REPO:$IMAGE_TAG .
                '''
            }
        }

        stage('Login to ECR using IAM Role') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION \
                | docker login --username AWS --password-stdin $IMAGE_URI
                '''
            }
        }

        stage('Tag Image') {
            steps {
                sh '''
                docker tag $ECR_REPO:$IMAGE_TAG $IMAGE_URI:$IMAGE_TAG
                docker tag $ECR_REPO:$IMAGE_TAG $IMAGE_URI:latest
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                docker push $IMAGE_URI:$IMAGE_TAG
                docker push $IMAGE_URI:latest
                '''
            }
        }
    }
}
