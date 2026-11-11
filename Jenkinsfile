pipeline {
    agent any

    environment {
        IMAGE_NAME = "kubedemoapp:v1"
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
                bat "kubectl --kubeconfig \"${KUBECONFIG_PATH}\" rollout status deployment/kubedemoapp-deployment"
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

/*How to Fix It


1. Create a Docker Hub Personal Access Token (Recommended)

• Generate a Token: Log in to Docker Hub, go to Account Settings > Personal access tokens, and click Generate new token.
• Set Permissions: Give the token Read & Write permissions so Jenkins can push the image, then copy the generated token immediately.
• Revoke Exposed Password: Since your password appeared in plain text in the logs, change your Docker Hub password right away.

2. Store Credentials Securely in Jenkins

Instead of hardcoding credentials in your Jenkinsfile, store them in Jenkins Credentials:
• Navigate to Credentials: Open your Jenkins dashboard and go to Manage Jenkins > Credentials > System > Global credentials (unrestricted).
• Add Credential: Click Add Credentials, select Username with password, enter your Docker Hub username (nihaaaa34) and your new Personal Access Token as the password, and set the ID to dockerhub-creds.

3. Update Your Jenkinsfile

Replace the plain-text docker login command in your Jenkinsfile with withCredentials and --password-stdin so Jenkins masks the secret in logs:
groovy
stage('Docker Login') {
    steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
            bat 'echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin'
        }
    }
}
Use code with caution.

4. Verify Image Tagging Before Pushing

• Check Image Name: Your build stage currently tags the image locally as kubedemoapp:v1.
• Match Repository Namespace: Before running docker push in the next stage, make sure your Jenkinsfile tags the image with your Docker Hub username (for example, nihaaaa34/kubedemoapp:v1), 
otherwise Docker Hub will reject the push even after a successful login.
If you share the contents of your Jenkinsfile (with any passwords removed), I can review the Push Docker Image and Deploy to Kubernetes stages to help prevent any errors in the remaining steps.

}*/
