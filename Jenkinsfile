pipeline {
    agent any

    environment {
        DOCKER_HUB = "nithinkunusoth"
        IMAGE_NAME = "my-k8s-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout from GitHub') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/kunusothnithin/node-k8s-app.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                set -e
                npm install
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                set -e
                docker build -t $DOCKER_HUB/$IMAGE_NAME:$IMAGE_TAG .
                docker images
                '''
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-cred',
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh '''
                    set -e
                    echo "Logging into Docker Hub..."
                    echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                set -e
                echo "Pushing image..."
                docker push $DOCKER_HUB/$IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Start Minikube if not running') {
            steps {
                sh '''
                set -e
                if ! minikube status | grep -q "apiserver: Running"; then
                    echo "Starting Minikube..."
                    minikube start --driver=virtualbox
                fi
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                set -e
                sed -i "s/IMAGE_TAG/$IMAGE_TAG/g" k8s/deployment.yaml

                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                '''
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
    }
}
