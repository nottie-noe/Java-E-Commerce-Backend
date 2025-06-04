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

        stage('Deploy to EKS') {
            steps {
                echo '🚀 Deploying to Kubernetes...'
                withCredentials([
                    file(credentialsId: 'kubeconfig-eks', variable: 'KUBECONFIG_FILE'),
                    usernamePassword(credentialsId: 'aws-credentials', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh '''
                        export KUBECONFIG=$KUBECONFIG_FILE
                        export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                        export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY

                        cd backend/k8s
                        sed -i "s|image: .*|image: $IMAGE_NAME:$IMAGE_TAG|g" ecommerce-rollout.yaml

                        kubectl apply -f ecommerce-rollout.yaml
                        kubectl apply -f service-v2.yaml
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
