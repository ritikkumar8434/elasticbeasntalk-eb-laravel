pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO = "741853493541.dkr.ecr.us-east-1.amazonaws.com/laravel"
        IMAGE_TAG = "latest"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('ECR Login') {
            steps {
                sh '''
                  aws ecr get-login-password --region ${AWS_REGION} \
                  | docker login --username AWS --password-stdin ${ECR_REPO}
                '''
            }
        }

        stage('Create .env') {
            steps {
                sh '''
                  echo "APP_ENV=production" >> .env
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
                sh '''
                  aws ecs update-service \
                    --cluster laravel-cluster \
                    --service laravel-service \
                    --force-new-deployment \
                    --region ${AWS_REGION}
                '''
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
