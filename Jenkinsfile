pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'eswar1030'
        IMAGE_NAME     = 'my-app'
    }

    stages {
        stage('Update GitOps Repo') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github-pat-credential', usernameVariable: 'GIT_USER', passwordVariable: 'TOKEN')]) {
                        sh """
                            # Clean up old directory if it exists from a previous failed run
                            rm -rf gitops-temp

                            git clone https://\${TOKEN}@github.com/eswar1030/my-app-gitops-cicd.git gitops-temp
                            cd gitops-temp

                            # Update the image tag in deployment.yaml
                            sed -i 's|${env.DOCKERHUB_USER}/${env.IMAGE_NAME}:.*|${env.DOCKERHUB_USER}/${env.IMAGE_NAME}:${BUILD_NUMBER}|g' deployment.yaml

                            # Quote config values to prevent syntax errors
                            git config user.name "Jenkins CI"
                            git config user.email "jenkins@ci.com"

                            # Only commit and push if deployment.yaml actually changed
                            if ! git diff --quiet deployment.yaml; then
                                git commit -am "Update image tag to ${BUILD_NUMBER}"
                                git push origin main
                            else
                                echo "No changes detected in deployment.yaml. Skipping commit."
                            fi
                        """
                    }
                }
            }
        }
    }
}
