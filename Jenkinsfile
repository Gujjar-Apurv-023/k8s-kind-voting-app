pipeline {
agent any

stages {

    stage('Clone Repository') {
        steps {
            git url: 'https://github.com/Gujjar-Apurv-023/k8s-kind-voting-app.git', branch: 'main'
            echo "this is for clone testing 1"
        }
    }

    stage('Build Vote Image') {
        steps {
            sh '''
            cd vote
            docker build -t apurv023/vote-app:v1 .
            cd ..
            '''
        }
    }

    stage('Build Result Image') {
        steps {
            sh '''
            cd result
            docker build -t apurv023/result-app:v1 .
            cd ..
            '''
        }
    }

    stage('Build Worker Image') {
        steps {
            sh '''
            cd worker
            docker build -t apurv023/worker-app:v1 .
            cd ..
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
            
            '''
        }
    }

}

}
