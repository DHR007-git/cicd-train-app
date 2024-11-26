pipeline {
    agent any
    environment {
        DOCKER_HUB_CREDENTIALS = 'dockerhub-creds'
        GIT_REPO = 'https://github.com/DHR007-git/cicd-train-app.git'
        IMAGE_NAME = 'train-schedule-app'
        IMAGE_TAG = "${GIT_COMMIT}"
        K8S_CREDENTIALS = 'k8s-creds'
        KUBERNETES_DEPLOYMENT = 'train-schedule-app-deployment'
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: "${GIT_REPO}"
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t ${DOCKER_HUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} .'
                }
            }
        }
        stage('Push Docker Image to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDENTIALS}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    script {
                        sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
                        sh 'docker push ${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}'
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: "${K8S_CREDENTIALS}", variable: 'K8S_CONFIG')]) {
                    script {
                        sh "kubectl --kubeconfig=$K8S_CONFIG set image deployment/${KUBERNETES_DEPLOYMENT} ${IMAGE_NAME}=${DOCKER_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}"
                    }
                }
            }
        }
    }
    post {
        success {
            echo "Build and Deploy Successful"
        }
        failure {
            echo "Build or Deploy Failed"
        }
    }
}
