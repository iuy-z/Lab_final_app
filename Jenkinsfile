pipeline {
    agent any

    environment {
        IMAGE_NAME = "flask-user-app"
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }

    stages {

        /* ===============================
           CODE FETCH STAGE
        =============================== */
        stage('Code Fetch') {
            steps {
                echo "Fetching source code from GitHub..."
                git branch: 'main',
                    credentialsId: 'github-credentials',
                    url: 'https://github.com/yourusername/myapp.git'
            }
        }

        /* ===============================
           DOCKER IMAGE CREATION STAGE
        =============================== */
        stage('Docker Build & Push') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {

                        sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker build -t $DOCKER_USER/${IMAGE_NAME}:${IMAGE_TAG} .
                        docker tag $DOCKER_USER/${IMAGE_NAME}:${IMAGE_TAG} $DOCKER_USER/${IMAGE_NAME}:latest
                        docker push $DOCKER_USER/${IMAGE_NAME}:${IMAGE_TAG}
                        docker push $DOCKER_USER/${IMAGE_NAME}:latest
                        '''
                    }
                }
            }
        }

        /* ===============================
           KUBERNETES DEPLOYMENT STAGE
        =============================== */
        stage('Kubernetes Deployment') {
            steps {
                echo "Deploying application to Kubernetes..."
                sh '''
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                '''
            }
        }

        /* ===============================
           PROMETHEUS / GRAFANA STAGE
        =============================== */
        stage('Monitoring (Prometheus & Grafana)') {
            steps {
                echo "Deploying Prometheus and Grafana..."
                sh '''
                kubectl apply -f monitoring/prometheus.yaml
                kubectl apply -f monitoring/grafana.yaml
                '''
            }
        }
    }

    post {
        success {
            echo "✅ CI/CD Pipeline executed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check Jenkins logs."
        }
    }
}
