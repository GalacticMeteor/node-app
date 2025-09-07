pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = "my-app"
        DOCKER_TAG = "${BUILD_NUMBER}"
        CONTAINER_NAME = "my-app-${BUILD_NUMBER}"
        GITHUB_REPO = "https://github.com/GalacticMeteor/node-app.git"
        GITHUB_BRANCH = "main"
        EXPOSED_PORT = "8080"
        CONTAINER_PORT = "8080"
    }
    
    stages {
        stage('Checkout from GitHub') {
            steps {
                echo "Checking out code from GitHub..."
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "*/${GITHUB_BRANCH}"]],
                    userRemoteConfigs: [[url: "${GITHUB_REPO}"]]
                ])
            }
        }
        
        stage('Stop Existing Container') {
            steps {
                echo "Stopping existing containers..."
                sh '''
                    docker ps -q --filter ancestor=${DOCKER_IMAGE} | xargs -r docker stop || true
                    docker ps -aq --filter ancestor=${DOCKER_IMAGE} | xargs -r docker rm || true
                '''
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh """
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} -t ${DOCKER_IMAGE}:latest .
                """
            }
        }
        
        stage('Run Docker Container') {
            steps {
                echo "Running Docker container..."
                sh """
                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p ${EXPOSED_PORT}:${CONTAINER_PORT} \
                        --restart unless-stopped \
                        ${DOCKER_IMAGE}:${DOCKER_TAG}
                """
            }
        }
        
        stage('Verify Deployment') {
            steps {
                echo "Verifying container is running..."
                sh """
                    docker ps --filter name=${CONTAINER_NAME}
                    docker logs --tail 10 ${CONTAINER_NAME}
                """
            }
        }
    }
}