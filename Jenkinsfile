pipeline {
    agent any

    stages {

        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/Gujjar-Apurv-023/k8s-kind-voting-app.git', branch: 'main'
                echo "Repository cloned successfully"
            }
        }

        stage('Build Vote Image') {
            steps {
                sh '''
                cd vote
                docker build -t apurv023/vote-app:v1 .
                '''
            }
        }

        stage('Build Result Image') {
            steps {
                sh '''
                cd result
                docker build -t apurv023/result-app:v1 .
                '''
            }
        }

        stage('Build Worker Image') {
            steps {
                sh '''
                cd worker
                docker build -t apurv023/worker-app:v1 .
                '''
            }
        }

        stage('Login & Push Images') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'ApurvCredd',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {

                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin

                    docker push apurv023/vote-app:v1
                    docker push apurv023/result-app:v1
                    docker push apurv023/worker-app:v1
                    '''
                }
            }
        }

        stage('Deploy to GKE') {
            steps {
                sh '''
                echo "Deploying application"

                kubectl delete -f k8s-specifications/ || true
                kubectl apply -f k8s-specifications/

                kubectl set image deployment/vote vote=apurv023/vote-app:v1
                kubectl set image deployment/result result=apurv023/result-app:v1
                kubectl set image deployment/worker worker=apurv023/worker-app:v1

                kubectl rollout status deployment/vote
                kubectl rollout status deployment/result
                kubectl rollout status deployment/worker

                kubectl get all
                '''
            }
        }

        stage('Install Monitoring Stack (Prometheus + Grafana)') {
            steps {
                sh '''
                echo "Installing Helm"

                curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
                chmod 700 get_helm.sh
                ./get_helm.sh

                echo "Adding Helm repositories"

                helm repo add prometheus-community https://prometheus-community.github.io/helm-charts || true
                helm repo add stable https://charts.helm.sh/stable || true
                helm repo update

                echo "Deleting old monitoring stack if exists"

                helm uninstall kind-prometheus -n monitoring || true

                echo "Deleting namespace if exists"

                kubectl delete namespace monitoring --ignore-not-found=true

                echo "Creating monitoring namespace"

                kubectl create namespace monitoring

                echo "Installing kube-prometheus-stack"

                helm install kind-prometheus prometheus-community/kube-prometheus-stack \
                --namespace monitoring \
                --create-namespace \
                --set grafana.service.type=LoadBalancer \
                --set prometheus.service.type=LoadBalancer \
                --set alertmanager.service.type=LoadBalancer

                echo "Checking services"

                kubectl get svc -n monitoring
                kubectl get namespace
                '''
            }
        }

    }
}
