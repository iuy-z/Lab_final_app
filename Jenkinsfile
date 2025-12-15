pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "your_dockerhub_username"
        IMAGE_NAME = "flask-user-app"
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }

    stages {

        stage('Code Fetch') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-credentials',
                    url: 'https://github.com/iuy-z/Lab_final_app.git'
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    echo "Building Docker Image..."
                    sh "docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} ."
                    sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker tag ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} ${DOCKERHUB_USER}/${IMAGE_NAME}:latest"
                    sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Kubernetes Deployment') {
            steps {
                echo "Deploying application using kubectl..."
                sh '''
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                '''
            }
        }

        stage('Monitoring (Prometheus & Grafana)') {
            steps {
                echo "Deploying monitoring stack..."
                sh '''
                kubectl create namespace monitoring || true
                helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
                helm repo update
                helm install prometheus kube-prometheus-stack --namespace monitoring --create-namespace || true
                '''
            }
        }
    }

    post {
        success {
            echo "CI/CD pipeline executed successfully!"
        }
        failure {
            echo "Pipeline failed. Check logs!"
        }
    }
}
