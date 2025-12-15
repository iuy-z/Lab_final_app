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
                    export KUBECONFIG=${KUBECONFIG}

                    # Create monitoring namespace if not exists
                    kubectl get ns monitoring || kubectl create ns monitoring

                    # Apply Prometheus pod + service
                    kubectl apply -f monitoring/prometheus.yaml -n monitoring --validate=false
                    kubectl apply -f monitoring/prometheus-svc.yaml -n monitoring --validate=false

                    # Apply Grafana pod + service
                    kubectl apply -f monitoring/grafana.yaml -n monitoring --validate=false
                    kubectl apply -f monitoring/grafana-svc.yaml -n monitoring --validate=false

                    # Optional: check pods and services in monitoring namespace
                    kubectl get pods -n monitoring
                    kubectl get svc -n monitoring
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
