
pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'arupdeb381'
        IMAGE_NAME = 'myshop-app'
        IMAGE_TAG = '1.0.0'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Verify Docker') {
            steps {
                sh 'docker --version'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                      -f docker/Dockerfile \
                      -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} \
                      .
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker-hub-id',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {
                    sh '''
                        set +x
                        export DOCKER_CONFIG="$(mktemp -d)"
                        trap 'docker logout >/dev/null 2>&1 || true; rm -rf "$DOCKER_CONFIG"' EXIT

                        echo "$DOCKER_TOKEN" | docker login \
                          -u "$DOCKER_USER" \
                          --password-stdin

                        docker push \
                          ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Docker image pushed successfully!'
        }

        failure {
            echo 'Pipeline failed! Check console logs.'
        }
    }
}
