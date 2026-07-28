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
                    withCredentials([string(credentialsId: 'github-pat-credential', variable: 'TOKEN')]) {
                        sh '''
                            # Clean up old directory if it exists
                            rm -rf gitops-temp

                            # Clone repository using credential token
                            git clone https://${TOKEN}@github.com/eswar1030/my-app-gitops-cicd.git gitops-temp
                        '''

                        dir('gitops-temp') {
                            sh """
                                # Configure Git identity locally
                                git config user.name "Jenkins CI"
                                git config user.email "jenkins@ci.com"

                                # Update image tag in deployment manifest
                                sed -i 's|${env.DOCKERHUB_USER}/${env.IMAGE_NAME}:.*|${env.DOCKERHUB_USER}/${env.IMAGE_NAME}:${BUILD_NUMBER}|g' deployment.yaml

                                # Check for modifications and push if changes exist
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
    post {
        always {
            sh 'rm -rf gitops-temp'
        }
    }
}
