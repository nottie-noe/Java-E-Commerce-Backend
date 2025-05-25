pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "nottiey/ecommerce-backend"
        DOCKER_CREDENTIALS_ID = "docker-hub-credentials" // Confirm this matches the correct credentials ID in Jenkins
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
                sh '''
                    cd backend
                    if [ ! -f pom.xml ]; then
                        echo "❌ No pom.xml found in backend directory! Exiting..."
                        exit 1
                    fi
                    mvn clean package -DskipTests
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    cd backend
                    if [ ! -f Dockerfile ]; then
                        echo "❌ No Dockerfile found! Exiting..."
                        exit 1
                    fi
                    docker build --no-cache -t $DOCKER_IMAGE .
                    docker tag $DOCKER_IMAGE "$DOCKER_USER/$DOCKER_IMAGE:latest"
                '''
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "$DOCKER_CREDENTIALS_ID", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push "$DOCKER_USER/$DOCKER_IMAGE:latest"
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            when {
                expression { return fileExists('backend/k8s/deployment.yaml') }
            }
            steps {
                sh '''
                    if ! command -v kubectl &> /dev/null; then
                        echo "❌ kubectl is not installed or configured!"
                        exit 1
                    fi
                    kubectl apply -f backend/k8s/deployment.yaml
                    kubectl apply -f backend/k8s/service.yaml

                    echo "✅ Verifying Kubernetes deployment..."
                    kubectl get pods -n default
                    kubectl get services -n default
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
