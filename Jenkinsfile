pipeline {
    agent any
    
    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = '884990039558.dkr.ecr.us-east-1.amazonaws.com/my-cicd-app'
        EC2_HOST = 'ubuntu@13.221.41.71'
        IMAGE_TAG = "${BUILD_NUMBER}"
        APP_PORT = '3000'
        AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code from GitHub...'
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                script {
                    sh "docker build -t ${ECR_REPO}:${IMAGE_TAG} ."
                    sh "docker tag ${ECR_REPO}:${IMAGE_TAG} ${ECR_REPO}:latest"
                }
            }
        }
        
        stage('Login to ECR') {
            steps {
                echo 'Logging in to AWS ECR...'
                script {
                    sh '''
                        aws ecr get-login-password --region ${AWS_REGION} | \
                        docker login --username AWS --password-stdin ${ECR_REPO}
                    '''
                }
            }
        }
        
        stage('Push to ECR') {
            steps {
                echo 'Pushing Docker image to ECR...'
                script {
                    sh "docker push ${ECR_REPO}:${IMAGE_TAG}"
                    sh "docker push ${ECR_REPO}:latest"
                }
            }
        }
        
        stage('Deploy to EC2') {
            steps {
                echo 'Deploying to EC2 instance...'
                sshagent(['ec2-ssh-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_HOST} '
                            aws ecr get-login-password --region ${AWS_REGION} | \
                            docker login --username AWS --password-stdin ${ECR_REPO}
                            
                            docker stop my-app || true
                            docker rm my-app || true
                            
                            docker pull ${ECR_REPO}:latest
                            docker run -d --name my-app -p ${APP_PORT}:${APP_PORT} ${ECR_REPO}:latest
                            
                            docker ps
                        '
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}