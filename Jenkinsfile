pipeline {
    agent any

    environment {
        IMAGE_NAME = "kubedemoappp:v1"
        DOCKER_REPO = "nihaaaa34/sample:kubeimage12"
        KUBECONFIG_PATH = "C:\\Windows\\System32\\config\\systemprofile\\.kube\\config"
    }

    stages {
        stage('Build Docker Image') {
            steps {
                echo "Building Docker Image..."
                bat "docker build -t ${IMAGE_NAME} ."
            }
        }

        stage('Docker Login') {
            steps {
                echo "Logging in to Docker Hub..."
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    // Note: No space between %DOCKER_PASS% and | to prevent Windows from adding a trailing space to the password
                    bat 'echo %DOCKER_PASS%| docker login -u %DOCKER_USER% --password-stdin'
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                echo "Tagging and pushing Docker Image to Docker Hub..."
                bat "docker tag ${IMAGE_NAME} ${DOCKER_REPO}"
                bat "docker push ${DOCKER_REPO}"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "Deploying to Kubernetes..."
                bat "kubectl --kubeconfig \"${KUBECONFIG_PATH}\" config current-context"
                bat "kubectl --kubeconfig \"${KUBECONFIG_PATH}\" get nodes"
                bat "kubectl --kubeconfig \"${KUBECONFIG_PATH}\" apply -f deployment.yaml"
                bat "kubectl --kubeconfig \"${KUBECONFIG_PATH}\" apply -f service.yaml"
                bat "kubectl --kubeconfig \"${KUBECONFIG_PATH}\" rollout status deployment/kubedemoappp-deployment"
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Please check the logs.'
        }
        always {
            // Optional: Clean up Docker Hub session after build finishes
            bat 'docker logout'
        }
    }
}
