pipeline {
    agent any

    environment {
        // Application source repository
        APP_REPO = 'https://github.com/eswar1030/my-app.git'

        // GitOps repository
        GITOPS_REPO = 'https://github.com/eswar1030/my-app-gitops-cicd.git'

        // Docker Hub repository
        DOCKER_HUB_REPO = 'eswarpala1988/my-app'

        // Image tag
        BUILD_TAG = "v${BUILD_NUMBER}"
    }

    stages {

        // ============================================================
        // STAGE 1 - Checkout Application Source
        // ============================================================
        stage('Checkout Source Code') {
            steps {
                echo "=========================================="
                echo "Checking out application source code"
                echo "=========================================="

                deleteDir()

                git branch: 'main',
                    credentialsId: 'github-creds',
                    url: "${APP_REPO}"

                sh '''
                    echo "===== Git Information ====="
                    git status
                    git log -1 --oneline
                '''
            }
        }


        // ============================================================
        // STAGE 2 - Maven Build
        // ============================================================
        stage('Maven Build') {
            steps {
                echo "=========================================="
                echo "Building application with Maven"
                echo "=========================================="

                sh '''
                    java -version
                    mvn -version

                    mvn clean package -DskipTests
                '''
            }
        }


        // ============================================================
        // STAGE 3 - Unit Tests
        // ============================================================
        stage('Unit Tests') {
            steps {
                echo "=========================================="
                echo "Running Unit Tests"
                echo "=========================================="

                sh '''
                    mvn test
                '''
            }
        }


        // ============================================================
        // STAGE 4 - SonarQube
        // ============================================================
        stage('SonarQube Analysis') {
            steps {
                echo "=========================================="
                echo "Running SonarQube Analysis"
                echo "=========================================="

                // IMPORTANT:
                // Change 'sonarqube-server' to the exact name
                // configured in Jenkins -> Manage Jenkins
                // -> System -> SonarQube servers

                withSonarQubeEnv('sonarqube-server') {
                    sh '''
                        mvn sonar:sonar
                    '''
                }
            }
        }


        // ============================================================
        // STAGE 5 - Quality Gate
        // ============================================================
        stage('Quality Gate') {
            steps {
                echo "=========================================="
                echo "Waiting for SonarQube Quality Gate"
                echo "=========================================="

                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }


        // ============================================================
        // STAGE 6 - Build Docker Image
        // ============================================================
        stage('Build Docker Image') {
            steps {
                echo "=========================================="
                echo "Building Docker Image"
                echo "=========================================="

                sh '''
                    docker version

                    echo "Building:"
                    echo "${DOCKER_HUB_REPO}:${BUILD_TAG}"

                    docker build \
                        -t ${DOCKER_HUB_REPO}:${BUILD_TAG} \
                        -t ${DOCKER_HUB_REPO}:latest \
                        .
                '''
            }
        }


        // ============================================================
        // STAGE 7 - Push Image to Docker Hub
        // ============================================================
        stage('Push Image to Docker Hub') {
            steps {
                echo "=========================================="
                echo "Pushing Docker Image to Docker Hub"
                echo "=========================================="

                script {
                    docker.withRegistry(
                        'https://index.docker.io/v1/',
                        'docker-credentials'
                    ) {

                        docker.image(
                            "${DOCKER_HUB_REPO}:${BUILD_TAG}"
                        ).push()

                        docker.image(
                            "${DOCKER_HUB_REPO}:latest"
                        ).push()
                    }
                }
            }
        }


        // ============================================================
        // STAGE 8 - Clone GitOps Repository
        // ============================================================
        stage('Clone GitOps Repository') {
            steps {
                echo "=========================================="
                echo "Cloning GitOps Repository"
                echo "=========================================="

                dir('gitops-temp') {

                    git branch: 'main',
                        credentialsId: 'github-creds',
                        url: "${GITOPS_REPO}"

                    sh '''
                        echo "===== GitOps Repository ====="
                        git status
                        echo
                        echo "===== Files ====="
                        ls -la
                    '''
                }
            }
        }


        // ============================================================
        // STAGE 9 - Update Kubernetes Manifest
        // ============================================================
        stage('Update Kubernetes Manifest') {
            steps {
                echo "=========================================="
                echo "Updating deployment.yaml"
                echo "=========================================="

                dir('gitops-temp') {

                    sh '''
                        echo "===== Before Update ====="
                        cat deployment.yaml

                        echo
                        echo "===== Updating Image ====="

                        sed -i -E \
                        "s|image:.*|image: ${DOCKER_HUB_REPO}:${BUILD_TAG}|g" \
                        deployment.yaml

                        echo
                        echo "===== After Update ====="
                        cat deployment.yaml
                    '''
                }
            }
        }


        // ============================================================
        // STAGE 10 - Commit and Push GitOps Changes
        // ============================================================
        stage('Commit and Push GitOps Changes') {
            steps {
                echo "=========================================="
                echo "Committing GitOps Change"
                echo "=========================================="

                dir('gitops-temp') {

                    sh '''
                        git config user.name "Jenkins CI"
                        git config user.email "jenkins@ci.com"

                        echo "===== Git Diff ====="
                        git diff -- deployment.yaml || true

                        if git diff --quiet deployment.yaml; then
                            echo "No changes detected."
                            exit 0
                        fi

                        git add deployment.yaml

                        git commit \
                            -m "Update image tag to ${BUILD_TAG} [skip ci]"

                        git push origin main
                    '''
                }
            }
        }
    }


    // ================================================================
    // POST ACTIONS
    // ================================================================
    post {

        success {
            echo "=========================================="
            echo "PIPELINE SUCCESS"
            echo "=========================================="

            echo "Docker Image:"
            echo "${DOCKER_HUB_REPO}:${BUILD_TAG}"

            echo "GitOps Repository:"
            echo "${GITOPS_REPO}"

            echo "ArgoCD should detect the GitOps change."
        }

        failure {
            echo "=========================================="
            echo "PIPELINE FAILED"
            echo "=========================================="

            echo "Check the failed stage and Jenkins console output."
        }

        always {
            echo "Cleaning workspace..."

            sh '''
                rm -rf gitops-temp || true
            '''

            cleanWs()
        }
    }
}
