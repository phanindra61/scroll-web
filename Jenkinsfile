pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'phanindra61/test-dev:latest'
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'master', url: 'https://github.com/swathis10/scroll-web.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                echo "🔧 Building Docker image..."
                docker build -t $DOCKER_IMAGE .
                '''
            }
        }

        stage('Login to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                echo "📦 Pushing image to Docker Hub..."
                docker push $DOCKER_IMAGE
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                echo "🚀 Deploying to Kubernetes..."
                microk8s.kubectl apply -f deploy.yaml
                '''
            }
        }
    }
}

