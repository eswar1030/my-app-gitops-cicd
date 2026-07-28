pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'eswar1030' // Replace with your actual Docker Hub username
        IMAGE_NAME     = 'my-app'    // Replace with your actual image name
    }

    stages {
        stage('Update GitOps Repo') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'github-pat-credential', variable: 'TOKEN')]) {
                        sh """
                            git clone https://\${TOKEN}@github.com/eswar1030/my-app-gitops-cicd.git gitops-temp
                            cd gitops-temp
                            sed -i 's|${env.DOCKERHUB_USER}/${env.IMAGE_NAME}:.*|${env.DOCKERHUB_USER}/${env.IMAGE_NAME}:${BUILD_NUMBER}|g' deployment.yaml
                            git config user.name "Jenkins CI"
                            git config user.email "jenkins@ci.com"
                            git commit -am "Update image tag to ${BUILD_NUMBER}"
                            git push origin main
                        """
                    }
                }
            }
        }
    }
}
