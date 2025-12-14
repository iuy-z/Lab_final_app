pipeline {
    agent any

    environment {
        IMAGE_NAME = "flask-user-app"
        IMAGE_TAG  = "${BUILD_NUMBER}"
        DOCKER_TLS_VERIFY = "1"
        DOCKER_HOST = "tcp://192.168.49.2:2376"
        DOCKER_CERT_PATH = "/home/ubuntu/.minikube/certs"
        MINIKUBE_ACTIVE_DOCKERD = "minikube"
    }

    stages {

        /* ===============================
           CODE FETCH STAGE
        =============================== */
        stage('Code Fetch') {
            steps {
                echo "Fetching source code from GitHub..."
                git branch: 'main',
                    credentialsId: 'github-credentials',  // optional if private repo
                    url: 'https://github.com/iuy-z/Lab_final_app.git'
            }
        }

        /* ===============================
           DOCKER IMAGE CREATION STAGE
        =============================== */
        stage('Docker Build for Minikube') {
            steps {
                echo "Building Docker image for Minikube..."
                sh '''
                # Docker build using Minikube's Docker daemon
                docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                '''
            }
        }

        /* ===============================
           KUBERNETES DEPLOYMENT STAGE
        =============================== */
        stage('Kubernetes Deployment') {
            steps {
                echo "Deploying application to Minikube..."
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
                echo "Deploying Prometheus & Grafana on Minikube..."
                sh '''
                kubectl apply -f monitoring/prometheus.yaml
                kubectl apply -f monitoring/grafana.yaml
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Minikube CI/CD pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed – check logs"
        }
    }
}
