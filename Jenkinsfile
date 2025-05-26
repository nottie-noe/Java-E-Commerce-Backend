pipeline {
    agent any

    environment {
        IMAGE_NAME = 'nottiey/ecommerce-backend'
        IMAGE_TAG = 'latest'
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo '🔄 Cloning repository...'
                git url: 'https://github.com/nottie-noe/Java-E-Commerce-Backend.git', branch: 'main'
            }
        }

        stage('Build Project') {
            steps {
                echo '⚙️ Running Maven clean install...'
                sh '''
                    cd backend
                    mvn clean install -DskipTests
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🐳 Building Docker image...'
                sh '''
                    cd backend
                    docker build -t $IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Push Docker Image to DockerHub') {
            steps {
                echo '📦 Pushing Docker image to DockerHub...'
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check console output.'
        }
    }
}
