
pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        AWS_ACCOUNT_ID = '485944703924'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        ECR_REPOSITORY = 'ecr-demo'
        IMAGE_NAME = 'ecr-demo'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code from GitHub'
                checkout scm
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker image'
                sh '''
                    docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Test Application') {
            steps {
                echo 'Testing application container'
                sh '''
                    docker run -d \
                    --name flask-test-${BUILD_NUMBER} \
                    -p 5001:5000 \
                    ${IMAGE_NAME}:${BUILD_NUMBER}

                    sleep 5

                    curl -f http://localhost:5001/health
                '''
            }

        stage('ECR Login') {
            steps {
                echo 'Authenticating with AWS ECR'
                sh '''
                    aws ecr get-login-password \
                    --region ${AWS_REGION} |
                    docker login \
                    --username AWS \
                    --password-stdin ${ECR_REGISTRY}
                '''
            }
        }

        stage('Docker Tag') {
            steps {
                echo 'Tagging Docker image'
                sh '''
                    docker tag ${IMAGE_NAME}:${BUILD_NUMBER} \
                    ${ECR_REGISTRY}/${ECR_REPOSITORY}:${BUILD_NUMBER}

                    docker tag ${IMAGE_NAME}:${BUILD_NUMBER} \
                    ${ECR_REGISTRY}/${ECR_REPOSITORY}:latest
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                echo 'Pushing Docker image to AWS ECR'
                sh '''
                    docker push ${ECR_REGISTRY}/${ECR_REPOSITORY}:${BUILD_NUMBER}

                    docker push ${ECR_REGISTRY}/${ECR_REPOSITORY}:latest
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'CI/CD Pipeline failed. Check the console output.'
        }

        always {
            echo 'Pipeline execution completed.'
        }
    }
}

