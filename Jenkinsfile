pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "nottiey/ecommerce-backend"
        DOCKER_CREDENTIALS_ID = "docker-hub-credentials"
        GITHUB_CREDENTIALS_ID = "github-creds"
        NEXUS_CREDENTIALS_ID = "nexus"    // Jenkins credentials ID for Nexus username/password
    }

    stages {
        stage('Clone Repository') {
            steps {
                echo "🔄 Cloning repository..."
                git branch: 'main', credentialsId: "${GITHUB_CREDENTIALS_ID}", url: 'https://github.com/nottie-noe/Java-E-Commerce-Backend.git'
            }
        }

        stage('Build & Deploy to Nexus') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${NEXUS_CREDENTIALS_ID}", usernameVariable: 'NEXUS_USERNAME', passwordVariable: 'NEXUS_PASSWORD')]) {
                        // Create temporary settings.xml with Nexus credentials
                        writeFile file: 'settings.xml', text: """
                        <settings>
                          <servers>
                            <server>
                              <id>nexus</id>
                              <username>${NEXUS_USERNAME}</username>
                              <password>${NEXUS_PASSWORD}</password>
                            </server>
                          </servers>
                        </settings>
                        """

                        echo "⚙️ Running Maven deploy with Nexus credentials..."
                        sh '''
                            cd backend

                            # List before build
                            echo "Listing backend directory before build:"
                            ls -l

                            # Run Maven clean deploy using settings.xml (will push artifact to Nexus)
                            mvn clean deploy -s ../settings.xml -DskipTests

                            # Confirm target directory contents
                            echo "Listing target directory after deploy:"
                            ls -l target || echo "ERROR: target directory or jar not found!"
                        '''
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🐳 Building Docker image..."
                sh '''
                    cd backend

                    if [ ! -f Dockerfile ]; then
                        echo "❌ ERROR: Dockerfile not found in backend/"
                        exit 1
                    fi

                    if ls target/*.jar 1> /dev/null 2>&1; then
                        echo "✅ Found JAR file(s) in target/:"
                        ls target/*.jar
                    else
                        echo "❌ ERROR: No JAR file found in target/. Docker build will fail at COPY step!"
                        exit 1
                    fi

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
