
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
                      -f Docker/Dockerfile \
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
        sh '''
            export KUBECONFIG=/var/lib/jenkins/.kube/config

            echo "Deploying application to K3s..."

            kubectl apply -f kubernetets/deployment.yaml
            kubectl apply -f kubernetets/service.yaml

            kubectl -n myshop set image \
              deployment/myshop-app \
              myshop=arupdeb381/myshop-app:1.0.0
        '''
    }
}

stage('Verify Deployment') {
    steps {
        sh '''
            export KUBECONFIG=/var/lib/jenkins/.kube/config

            kubectl -n myshop rollout status \
              deployment/myshop-app \
              --timeout=180s

            kubectl -n myshop get deployments
            kubectl -n myshop get pods -o wide
            kubectl -n myshop get services
        '''
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
