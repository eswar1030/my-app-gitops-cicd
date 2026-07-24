stage('Update GitOps Repo') {
    steps {
        script {
            withCredentials([string(credentialsId: 'github-token', variable: 'TOKEN')]) {
                sh """
                    git clone https://${TOKEN}@github.com/eswar1030/my-app-gitops-cicd.git gitops-temp
                    cd gitops-temp
                    sed -i 's|${DOCKERHUB_USER}/${IMAGE_NAME}:.*|${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}|g' deployment.yaml
                    git config user.name "Jenkins CI"
                    git config user.email "jenkins@ci.com"
                    git commit -am "Update image tag to ${BUILD_NUMBER}"
                    git push origin main
                """
            }
        }
    }
}
