pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "rolex2k/flask-app"
        DOCKER_CREDENTIALS = "https://hub.docker.com/repositories/rolex2k"
        GIT_REPO = "https://github.com/shubkrsnha/k8sJenkinsproject.git"
        GIT_BRANCH = "main"   // <-- Change to "master" if your repo branch is master
    }

    stages {

        stage('Checkout') {
            steps {
                script {
                    // Clean workspace to avoid old metadata issues
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
                    sh "docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} ./app"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    echo "📤 Pushing Docker image to Docker Hub"
                    withDockerRegistry([credentialsId: "${DOCKER_CREDENTIALS}", url: "https://index.docker.io/v1/"]) {
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
                        flask-app=${DOCKER_IMAGE}:${BUILD_NUMBER} --record
                        kubectl rollout status deployment/flask-app
                    """
                }
            }
        }

        stage('Cleanup') {
            steps {
                script {
                    echo "🧹 Cleaning up unused Docker image"
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
