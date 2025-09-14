pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "rolex2k/flask-app"
        DOCKER_CREDENTIALS = "dockerhub-creds"
        GIT_REPO = "https://github.com/shubkrsnha/k8sJenkinsproject.git"
        GIT_BRANCH = "main"
    }

    stages {

        stage('Checkout') {
            steps {
                script {
                    deleteDir()
                    echo "🔄 Checking out branch: ${GIT_BRANCH}"
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: "*/${GIT_BRANCH}"]],
                        userRemoteConfigs: [[url: "${GIT_REPO}"]]
                    ])
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "🐳 Building Docker image: ${DOCKER_IMAGE}:${BUILD_NUMBER}"
                    sh "docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} -f app/Dockerfile app/"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    echo "📤 Pushing Docker image to Docker Hub"
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS) {
                        sh "docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}"
                    }
                }
            }
        }

        stage('Update K8s Deployment') {
            steps {
                script {
                    echo "🚀 Deploying to Kubernetes"
                    sh """
                        kubectl set image deployment/flask-app \
                        flask-app=${DOCKER_IMAGE}:${BUILD_NUMBER}
                        kubectl rollout status deployment/flask-app
                    """
                }
            }
        }

        stage('Cleanup') {
            steps {
                script {
                    echo "🧹 Cleaning up unused Docker images"
                    sh "docker rmi ${DOCKER_IMAGE}:${BUILD_NUMBER} || true"
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment Successful!"
        }
        failure {
            echo "❌ Deployment Failed!"
        }
    }
}
