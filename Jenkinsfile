
pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'arupdeb381'
        IMAGE_NAME = 'myshop-app'
        IMAGE_TAG = '1.0.0'
        NAMESPACE = 'myshop'
        DEPLOYMENT = 'myshop-app'
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Verify Environment') {
            steps {
                sh '''
                    docker --version
                    kubectl version --client
                    ls -la
                '''
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
                        usernameVariable: 'DH_USER',
                        passwordVariable: 'DH_TOKEN'
                    )
                ]) {
                    sh '''
                        set +x
                        export DOCKER_CONFIG="$(mktemp -d)"
                        trap 'rm -rf "$DOCKER_CONFIG"' EXIT

                        echo "$DH_TOKEN" | docker login \
                          -u "$DH_USER" \
                          --password-stdin

                        docker push \
                          ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Deploy to K3s') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'k3s-kubeconfig',
                        variable: 'KUBE_CONFIG_FILE'
                    )
                ]) {
                    sh '''
                        export KUBECONFIG="$KUBE_CONFIG_FILE"

                        kubectl apply \
                          -f kubernetes/deployment.yaml

                        kubectl apply \
                          -f kubernetes/service.yaml

                        kubectl -n ${NAMESPACE} set image \
                          deployment/${DEPLOYMENT} \
                          myshop=${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'k3s-kubeconfig',
                        variable: 'KUBE_CONFIG_FILE'
                    )
                ]) {
                    sh '''
                        export KUBECONFIG="$KUBE_CONFIG_FILE"

                        kubectl -n ${NAMESPACE} rollout status \
                          deployment/${DEPLOYMENT} \
                          --timeout=180s

                        kubectl -n ${NAMESPACE} get deployments
                        kubectl -n ${NAMESPACE} get pods -o wide
                        kubectl -n ${NAMESPACE} get services
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'CI/CD completed successfully!'
        }

        failure {
            echo 'CI/CD failed. Check console output.'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}
