pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "nottiey/ecommerce-backend"
        DOCKER_CREDENTIALS_ID = "dockerhub-creds"
        GITHUB_CREDENTIALS_ID = "github-creds"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', credentialsId: "${GITHUB_CREDENTIALS_ID}", url: 'https://github.com/nottie-noe/Java-E-Commerce-Backend.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build --no-cache -t $DOCKER_IMAGE .
                '''
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "$DOCKER_CREDENTIALS_ID", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker tag $DOCKER_IMAGE "$DOCKER_USER/$DOCKER_IMAGE:latest"
                        docker push "$DOCKER_USER/$DOCKER_IMAGE:latest"
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            when {
                expression { return fileExists('k8s/deployment.yaml') }
            }
            steps {
                sh '''
                    if ! command -v kubectl &> /dev/null; then
                        echo "❌ kubectl is not installed or configured!"
                        exit 1
                    fi
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully."
        }
        failure {
            echo "❌ Pipeline failed. Check console output."
        }
    }
}
