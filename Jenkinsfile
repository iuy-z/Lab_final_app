pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "irum90"
        DOCKERHUB_PASS = credentials('dockerhub-password') // Jenkins secret
        IMAGE_NAME    = "flask-user-app"
        IMAGE_TAG     = "${BUILD_NUMBER}"
        KUBECONFIG    = "/var/lib/jenkins/.kube/config" // path for Jenkins user
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
                    echo "Logging into DockerHub..."
                    sh "echo ${DOCKERHUB_PASS} | docker login -u ${DOCKERHUB_USER} --password-stdin"
                    
                    echo "Building Docker Image..."
                    sh "docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} ."
                    
                    echo "Pushing Docker Image..."
                    sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
                    
                    echo "Tagging and pushing latest..."
                    sh "docker tag ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} ${DOCKERHUB_USER}/${IMAGE_NAME}:latest"
                    sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Kubernetes Deployment') {
            steps {
                echo "Deploying application using kubectl..."
                sh """
                export KUBECONFIG=${KUBECONFIG}
                kubectl get nodes
                kubectl apply -f k8s/deployment.yaml --validate=false
                kubectl apply -f k8s/service.yaml --validate=false
                """
            }
        }

        stage('Monitoring (Prometheus & Grafana)') {
            steps {
                echo "Deploying monitoring stack..."
                sh """
                export KUBECONFIG=${KUBECONFIG}
                kubectl create namespace monitoring || true
                helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
                helm repo update
                helm install prometheus kube-prometheus-stack --namespace monitoring --create-namespace || true
                """
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
