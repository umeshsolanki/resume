pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'umesh-resume'
        DOCKER_TAG = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
        CONTAINER_NAME = 'umesh-resume-container'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    echo "Building from branch: ${env.BRANCH_NAME}"
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image: ${DOCKER_IMAGE}:${DOCKER_TAG}"
                    sh """
                        docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                        docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }
        
        stage('Stop Existing Container') {
            steps {
                script {
                    sh """
                        if docker ps -a | grep -q ${CONTAINER_NAME}; then
                            docker stop ${CONTAINER_NAME} || true
                            docker rm ${CONTAINER_NAME} || true
                        fi
                    """
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    echo "Deploying container: ${CONTAINER_NAME}"
                    sh """
                        docker run -d \\
                            --name ${CONTAINER_NAME} \\
                            -p 8080:80 \\
                            --restart unless-stopped \\
                            ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }
        
        stage('Health Check') {
            steps {
                script {
                    echo "Waiting for container to start..."
                    sleep(time: 5, unit: 'SECONDS')
                    sh """
                        if docker ps | grep -q ${CONTAINER_NAME}; then
                            echo "Container ${CONTAINER_NAME} is running successfully"
                            docker ps | grep ${CONTAINER_NAME}
                        else
                            echo "ERROR: Container failed to start"
                            exit 1
                        fi
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo "Deployment successful! Resume is available at http://jenkins.umeshsolanki.in:8080"
        }
        failure {
            echo "Deployment failed. Please check the logs."
        }
        always {
            cleanWs()
        }
    }
}

