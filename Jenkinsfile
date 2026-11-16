
pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'docker-cred'
        DOCKER_IMAGE          = 'manasavenu2004/maker'
        SONAR_SERVER_NAME     = 'sonarqube-env'
        SONAR_SCANNER_TOOL    = 'sonar-scanner'
        GIT_CREDENTIALS_ID    = 'github-cred'
        MANIFEST_REPO         = 'https://github.com/manasavenu-2004/maker_pipeline.git'
        MANIFEST_BRANCH       = 'main'
    }

    stages {

        stage('Checkout Source Code') {
            steps {
                echo "Source code checkout completed"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool "${SONAR_SCANNER_TOOL}"

                    withSonarQubeEnv("${SONAR_SERVER_NAME}") {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=maker-key \
                            -Dsonar.projectName=maker-project \
                            -Dsonar.sources=.
                        """
                    }
                }
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: "${DOCKER_CREDENTIALS_ID}",
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_PASS'
                        )
                    ]) {
                        sh """
                            echo "\$DOCKER_PASS" | docker login -u "\$DOCKER_USER" --password-stdin

                            docker build \
                            -t ${DOCKER_IMAGE}:${BUILD_NUMBER} \
                            -t ${DOCKER_IMAGE}:latest \
                            -f Dockerfile .

                            docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                            docker push ${DOCKER_IMAGE}:latest
                        """
                    }
                }
            }
        }

        stage('Update Manifest Repo') {
            steps {
                dir('manifest-workspace') {

                    // Clone the manifest repository
                    git branch: "${MANIFEST_BRANCH}",
                        credentialsId: "${GIT_CREDENTIALS_ID}",
                        url: "${MANIFEST_REPO}"

                    sh """
                        echo "===== CURRENT DIRECTORY ====="
                        pwd

                        echo "===== FILES ====="
                        ls -la

                        echo "===== MANIFEST FILE ====="
                        ls -la manifests/

                        # Update Docker image tag
                        sed -i '/name: maker/{n;s|image:.*|image: ${DOCKER_IMAGE}:${BUILD_NUMBER}|}' manifests/deployment.yaml

                        # Configure Git identity
                        git config user.name "manasavenu-2004"
                        git config user.email "manasavenu88914@gmail.com"

                        # Commit and push updated manifest
                        git add manifests/deployment.yaml
                        git commit -m "chore(manifest): update maker image tag to ${BUILD_NUMBER} [skip ci]" || echo "No changes to commit"

                        git push origin ${MANIFEST_BRANCH}
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully! Manifest repo updated with new image tag.'
        }

        failure {
            echo 'Pipeline failed. Please review the stage logs above.'
        }
    }
}
