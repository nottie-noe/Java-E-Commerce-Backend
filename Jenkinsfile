pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "nottiey/ecommerce-backend"
        DOCKER_CREDENTIALS_ID = "docker-hub-credentials"
        GITHUB_CREDENTIALS_ID = "github-creds"
    }

    stages {
        stage('Clone Repository') {
            steps {
                echo "🔄 Cloning repository..."
                git branch: 'main', credentialsId: "${GITHUB_CREDENTIALS_ID}", url: 'https://github.com/nottie-noe/Java-E-Commerce-Backend.git'
            }
        }

        stage('Build with Maven') {
            steps {
                echo "⚙️ Running Maven build..."
                sh '''
                    cd backend

                    # List contents before build to confirm clean state
                    echo "Listing backend directory before build:"
                    ls -l

                    # List target dir before build (should be empty or not exist)
                    echo "Listing target directory before build:"
                    ls -l target || echo "No target directory yet"

                    # Run Maven package
                    mvn clean package -DskipTests

                    # List target dir after build - confirm jar is created
                    echo "Listing target directory after build:"
                    ls -l target || echo "ERROR: target directory or jar not found! Maven build may have failed."
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🐳 Building Docker image..."
                sh '''
                    cd backend

                    # Debug: check Dockerfile presence and contents
                    if [ ! -f Dockerfile ]; then
                        echo "❌ ERROR: Dockerfile not found in backend/"
                        exit 1
                    else
                        echo "Dockerfile found:"
                        head -20 Dockerfile
                    fi

                    # Debug: confirm JAR presence for Docker COPY
                    if ls target/*.jar 1> /dev/null 2>&1; then
                        echo "✅ Found JAR file(s) in target/:"
                        ls target/*.jar
                    else
                        echo "❌ ERROR: No JAR file found in target/. Docker build will fail at COPY step!"
                        exit 1
                    fi

                    # Build docker image with backend as context
                    docker build --no-cache -t $DOCKER_IMAGE:latest .
                '''
            }
        }

        stage('Push Docker Image to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "$DOCKER_CREDENTIALS_ID", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    echo "🚀 Logging into Docker Hub and pushing image..."
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
                echo "📦 Deploying app to Kubernetes..."
                sh '''
                    if ! command -v kubectl &> /dev/null; then
                        echo "❌ kubectl not found. Skipping deployment."
                        exit 1
                    fi

                    kubectl apply -f backend/k8s/deployment.yaml
                    kubectl apply -f backend/k8s/service.yaml

                    echo "✅ Kubernetes resources:"
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
            echo "❌ Pipeline failed. Check the console output for errors above."
        }
    }
}
