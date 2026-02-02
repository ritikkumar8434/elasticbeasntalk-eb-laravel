pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO = "058264136175.dkr.ecr.us-east-1.amazonaws.com/laravel"
        IMAGE_TAG = "latest"
        AWS_CREDENTIALS = "aws-creds"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('ECR Login') {
            steps {
                withAWS(credentials: "${AWS_CREDENTIALS}", region: "${AWS_REGION}") {
                    sh '''
                      aws ecr get-login-password --region ${AWS_REGION} \
                      | docker login --username AWS --password-stdin ${ECR_REPO}
                    '''
                }
            }
        }

        stage('Create .env') {
            steps {
                sh '''
                  echo "APP_ENV=production" > .env
                  echo "APP_DEBUG=false" >> .env
                  echo "DB_CONNECTION=mysql" >> .env
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                  docker build -t laravel:${IMAGE_TAG} .
                  docker tag laravel:${IMAGE_TAG} ${ECR_REPO}:${IMAGE_TAG}
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                  docker push ${ECR_REPO}:${IMAGE_TAG}
                '''
            }
        }

        stage('Deploy to ECS') {
            steps {
                withAWS(credentials: "${AWS_CREDENTIALS}", region: "${AWS_REGION}") {
                    sh '''
                      aws ecs update-service \
                        --cluster laravel-cluster \
                        --service laravel-service \
                        --force-new-deployment
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment completed successfully!"
        }
        failure {
            echo "❌ Deployment failed!"
        }
    }
}
