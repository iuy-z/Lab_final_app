pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "irum90"
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
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo "Logging into DockerHub..."
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                    echo "Building Docker image..."
                    docker build -t $DOCKER_USER/flask-user-app:$BUILD_NUMBER .

                    echo "Pushing Docker image..."
                    docker push $DOCKER_USER/flask-user-app:$BUILD_NUMBER

                    echo "Tagging latest..."
                    docker tag $DOCKER_USER/flask-user-app:$BUILD_NUMBER $DOCKER_USER/flask-user-app:latest
                    docker push $DOCKER_USER/flask-user-app:latest
                    '''
                }
            }
        }

        stage('Kubernetes Deployment') {
            steps {
                echo "Deploying application using kubectl..."
                sh '''
                kubectl apply -f k8s/deployment.yaml \
                  --insecure-skip-tls-verify=true \
                  --validate=false
        
                kubectl apply -f k8s/service.yaml \
                  --insecure-skip-tls-verify=true \
                  --validate=false
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
                helm install prometheus kube-prometheus-stack \
                    --namespace monitoring \
                    --create-namespace || true
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
