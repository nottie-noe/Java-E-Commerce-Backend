pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "nottiey/ecommerce-backend"
        DOCKER_CREDENTIALS_ID = "docker-hub-credentials"
        GITHUB_CREDENTIALS_ID = "github-creds"
    }

    stages {
        stage('Debug Workspace') {
            steps {
                sh 'ls -R'
            }
        }

        stage('Build with Maven') {
            steps {
                sh '''
                    cd backend && \
                    if [ ! -f pom.xml ]; then
                        echo "❌ No pom.xml found! Exiting..."
                        exit 1
                    fi
                    mvn clean package -DskipTests
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    cd backend && \
                    if [ ! -f Dockerfile ]; then
                        echo "❌ No Dockerfile found! Exiting..."
                        exit 1
                    fi
                    docker build --no-cache -t $DOCKER_IMAGE:latest .
                '''
            }
        }

        stage('Push Docker Image to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "$DOCKER_CREDENTIALS_ID", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $DOCKER_IMAGE:latest
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
                        echo "❌ kubectl not found. Skipping deployment."
                        exit 1
                    fi

                    kubectl apply -f backend/k8s/deployment.yaml
                    kubectl apply -f backend/k8s/service.yaml

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
