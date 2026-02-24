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
                sh 'npm install'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $DOCKER_HUB/$IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }


        stage('Docker Login + Push (Debug)') {
    steps {
        withCredentials([usernamePassword(
            credentialsId: 'docker-cred',
            usernameVariable: 'DOCKER_USERNAME',
            passwordVariable: 'DOCKER_PASSWORD'
        )]) {
            sh '''
                echo "Logging in..."
                echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin

                echo "Pushing image..."
                docker push nithinkunusoth/my-k8s-app:${BUILD_NUMBER}
            '''
        }
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
                sed -i "s/IMAGE_TAG/$IMAGE_TAG/g" k8s/deployment.yaml

                minikube image load $DOCKER_HUB/$IMAGE_NAME:$IMAGE_TAG

                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                '''
            }
        }
    }
}
