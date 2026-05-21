pipeline {
    agent any

    environment {
        REGISTRY   = "localhost:5000"
        IMAGE_NAME = "demo-python-app"
    }

    stages {
        stage('Build Docker image') {
            steps {
                sh '''
                    set -e
                    TAG="build-${BUILD_NUMBER}"
                    docker build -t ${IMAGE_NAME}:${TAG} .
                    echo "${TAG}" > image_tag.txt
                '''
            }
        }

        stage('Run tests in container') {
            steps {
                sh '''
                    set -e
                    TAG="$(cat image_tag.txt)"
                    docker run --rm ${IMAGE_NAME}:${TAG} pytest -q
                '''
            }
        }

        stage('Push to local registry') {
            steps {
                sh '''
                    set -e
                    TAG="$(cat image_tag.txt)"
                    FULL_IMAGE="${REGISTRY}/${IMAGE_NAME}:${TAG}"
                    docker tag ${IMAGE_NAME}:${TAG} "${FULL_IMAGE}"
                    docker push "${FULL_IMAGE}"
                    echo "${FULL_IMAGE}" > image_fullname.txt
                '''
            }
        }

        stage('Deploy container') {
            steps {
                sh '''
                    set -e
                    FULL_IMAGE="$(cat image_fullname.txt)"

                    docker rm -f demo-python-app-prod || true
                    docker pull "${FULL_IMAGE}"

                    docker run -d \
                      --name demo-python-app-prod \
                      -p 8000:8000 \
                      "${FULL_IMAGE}"
                '''
            }
        }
    }

    post {
        always {
            sh '''
                echo "=== Running containers ==="
                docker ps
            '''
        }
    }
}