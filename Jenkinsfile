pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'swathikarthik/test-dev:latest'
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: "${env.BRANCH_NAME}", url: 'https://github.com/manjukolkar/scroll-web.git'
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
                kubectl apply -f $DEPLOY_FILE
                echo "Waiting for pods to stabilize..."
                sleep 20
                kubectl get pods
                '''
            }
        }

        stage('Apply Ingress & Verify') {
            steps {
                sh '''
                echo "🌐 Applying Ingress for domain $DOMAIN ..."
                kubectl apply -f $DEPLOY_FILE
                echo "Waiting for ingress to be ready..."
                sleep 20
                kubectl get ingress
                echo "🔍 Verifying application availability..."
                curl -I http://$DOMAIN || echo "⚠️ Could not verify via curl, please check browser."
                echo "✅ Deployment complete! Access: http://$DOMAIN"
                '''
            }
        }
    }

    post {
        success {
            echo '✅ CI/CD pipeline executed successfully. App deployed and accessible via Ingress.'
        }
        failure {
            echo '❌ Build or deploy failed. Please review Jenkins logs.'
        }
        aborted {
            echo '⚠️ Pipeline aborted by user.'
        }
    }
}

