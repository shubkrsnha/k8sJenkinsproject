pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "rolex2k/flask-app"
        //KUBECONFIG = credentials('kube-config')   // if using kubeconfig secret
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/shubkrsnha/k8sJenkinsproject.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t $DOCKER_IMAGE:${BUILD_NUMBER} ./app"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry([credentialsId: 'docker-hub', url: '']) {
                        sh "docker push $DOCKER_IMAGE:${BUILD_NUMBER}"
                    }
                }
            }
        }

        stage('Update K8s Deployment') {
            steps {
                script {
                    sh """
                        kubectl set image deployment/flask-app flask-app=$DOCKER_IMAGE:${BUILD_NUMBER} --record
                        kubectl rollout status deployment/flask-app
                    """
                }
            }
        }

        stage('Cleanup') {
            steps {
                sh "docker rmi $DOCKER_IMAGE:${BUILD_NUMBER} || true"
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
