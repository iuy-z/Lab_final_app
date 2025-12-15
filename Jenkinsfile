pipeline {
    agent any

    environment {
        IMAGE_NAME = "flask-user-app"
        KUBECONFIG = "/var/lib/jenkins/.kube/config"
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
                    passwordVariable: 'DOCKER_TOKEN'
                )]) {
                    sh '''
                        echo "$DOCKER_TOKEN" | docker login -u "$DOCKER_USER" --password-stdin
                        docker build -t $DOCKER_USER/$IMAGE_NAME:$BUILD_NUMBER .
                        docker push $DOCKER_USER/$IMAGE_NAME:$BUILD_NUMBER
                        docker tag $DOCKER_USER/$IMAGE_NAME:$BUILD_NUMBER $DOCKER_USER/$IMAGE_NAME:latest
                        docker push $DOCKER_USER/$IMAGE_NAME:latest
                    '''
                }
            }
        }

        stage('Kubernetes Deployment') {
            steps {
                sh '''
                    kubectl get nodes
                    kubectl apply -f k8s/deployment.yaml --validate=false
                    kubectl apply -f k8s/service.yaml --validate=false
                '''
            }
        }
        stage('Monitoring (Prometheus & Grafana)') {
            steps {
                echo "Deploying Prometheus and Grafana..."
                sh """
                    # Set KUBECONFIG for Jenkins
                    export KUBECONFIG=${KUBECONFIG}
            
                    # Apply Prometheus YAML
                    kubectl apply -f monitoring/prometheus.yaml --validate=false
            
                    # Apply Grafana YAML
                    kubectl apply -f monitoring/grafana.yaml --validate=false
            
                    # Optional: check pods
                    kubectl get pods -n default
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
