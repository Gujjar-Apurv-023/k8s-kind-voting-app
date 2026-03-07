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
                credentialsId: 'apurv023',
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

    stage('Deploy to Kubernetes') {
        steps {
            sh '''
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
            echo "Installing HELM"

            curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
            chmod 700 get_helm.sh
            ./get_helm.sh

            echo "Adding Helm Repositories"

            helm repo add prometheus-community https://prometheus-community.github.io/helm-charts || true
            helm repo add stable https://charts.helm.sh/stable || true
            helm repo update

            echo "Checking existing Prometheus stack"

            if helm list -n monitoring | grep kind-prometheus; then
                echo "Removing existing Prometheus stack"
                helm uninstall kind-prometheus -n monitoring
            fi

            echo "Deleting old monitoring namespace"

            kubectl delete namespace monitoring --ignore-not-found=true

            echo "Creating monitoring namespace"

            kubectl create namespace monitoring

            echo "Installing kube-prometheus-stack"

            helm install kind-prometheus prometheus-community/kube-prometheus-stack \
            --namespace monitoring \
            --set prometheus.service.type=NodePort \
            --set prometheus.service.nodePort=30000 \
            --set grafana.service.type=NodePort \
            --set grafana.service.nodePort=31000 \
            --set alertmanager.service.type=NodePort \
            --set alertmanager.service.nodePort=32000 \
            --set prometheus-node-exporter.service.type=NodePort \
            --set prometheus-node-exporter.service.nodePort=32001

            echo "Waiting for monitoring pods to become ready"

            kubectl wait --for=condition=Ready pods --all -n monitoring --timeout=300s

            echo "Monitoring services"

            kubectl get svc -n monitoring

            echo "Starting Port Forwarding"

            kubectl port-forward svc/kind-prometheus-kube-prome-prometheus -n monitoring 9090:9090 --address=0.0.0.0 &
            kubectl port-forward svc/kind-prometheus-grafana -n monitoring 3000:80 --address=0.0.0.0 &

            echo "Prometheus and Grafana deployed successfully"
            '''
        }
    }

}


}
