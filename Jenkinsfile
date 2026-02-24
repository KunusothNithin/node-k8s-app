pipeline {
    agent any

    stages {

        stage('Checkout from GitHub') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/kunusothnithin/node-k8s-app.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t my-k8s-app:${BUILD_NUMBER} .
                docker tag my-k8s-app:${BUILD_NUMBER} nithinkunusoth/my-k8s-app:${BUILD_NUMBER}
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker push nithinkunusoth/my-k8s-app:${BUILD_NUMBER}'
            }
        }

    stage('Start Minikube if not running') {
    steps {
        sh '''
        if ! minikube status | grep -q "apiserver: Running"; then
            echo "Minikube is not running. Starting now..."
            minikube start --driver=virtualbox
        fi
        '''
    }
}

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                # Replace image tag inside deployment.yaml
                sed -i "s/IMAGE_TAG/${BUILD_NUMBER}/g" k8s/deployment.yaml

                # Load image into Minikube
                minikube image load nithinkunusoth/my-k8s-app:${BUILD_NUMBER}

                # Apply manifests
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                '''
            }
        }
    }
}