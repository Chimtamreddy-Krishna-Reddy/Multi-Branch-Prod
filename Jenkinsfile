pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    environment {
        IMAGE_NAME = "kastrov/multibranch-flask-app"
        GIT_USER   = "kastrokiran"
        GIT_EMAIL  = "learnwithkastro@gmail.com"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Push Image') {
            when {
                branch 'main'
            }

            steps {
                script {

                    env.IMAGE_TAG = "build-${BUILD_NUMBER}"

                    withCredentials([
                        usernamePassword(
                            credentialsId: 'dockerhub-creds',
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_PASS'
                        )
                    ]) {

                        sh '''
                            set -e

                            echo "$DOCKER_PASS" | docker login \
                                -u "$DOCKER_USER" \
                                --password-stdin

                            docker build \
                                -t ${IMAGE_NAME}:${IMAGE_TAG} .

                            docker push \
                                ${IMAGE_NAME}:${IMAGE_TAG}

                            docker logout
                        '''
                    }
                }
            }
        }

        stage('Update K8s Manifest') {
            when {
                branch 'main'
            }

            steps {
                script {

                    withCredentials([
                        usernamePassword(
                            credentialsId: 'github-creds',
                            usernameVariable: 'GIT_USERNAME',
                            passwordVariable: 'GIT_TOKEN'
                        )
                    ]) {

                        sh '''
                            set -e

                            git config user.name "$GIT_USER"
                            git config user.email "$GIT_EMAIL"

                            sed -i \
                                "s|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|" \
                                k8s/deployment.yml

                            git add k8s/deployment.yml

                            if git diff --cached --quiet; then
                                echo "No changes detected"
                                exit 0
                            fi

                            git commit \
                                -m "Update image to ${IMAGE_TAG} [skip ci]"

                            git push \
                                https://${GIT_USERNAME}:${GIT_TOKEN}@github.com/Chimtamreddy-Krishna-Reddy/Multi-Branch-Prod.git \
                                HEAD:main
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}