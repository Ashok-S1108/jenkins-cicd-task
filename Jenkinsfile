
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'jenkins-cicd-task'
        CONTAINER_NAME = 'jenkins-cicd-app'
        HOST_PORT = '3001'
        APP_PORT = '3000'
        NODE_IMAGE = 'node:22-alpine'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    docker run --rm \
                        -v "$WORKSPACE:/app" \
                        -w /app \
                        ${NODE_IMAGE} \
                        npm ci
                '''
            }
        }

        stage('Code Quality') {
            steps {
                sh '''
                    docker run --rm \
                        -v "$WORKSPACE:/app" \
                        -w /app \
                        ${NODE_IMAGE} \
                        sh -c "node --check app.js &&
                               node --check bin/www &&
                               node --check routes/index.js"
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:latest .'
            }
        }

        stage('Test') {
            steps {
                sh '''
                    docker run --rm ${IMAGE_NAME}:latest node --version
                    docker run --rm ${IMAGE_NAME}:latest npm --version
                    docker image inspect ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker rm -f ${CONTAINER_NAME} || true

                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        --restart unless-stopped \
                        -p ${HOST_PORT}:${APP_PORT} \
                        ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    for i in $(seq 1 15); do
                        if curl --fail --silent \
                            http://localhost:${HOST_PORT}/; then
                            echo "Health check passed!"
                            exit 0
                        fi

                        echo "Waiting for application..."
                        sleep 2
                    done

                    echo "Health check failed."
                    docker logs ${CONTAINER_NAME}
                    exit 1
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check the console output.'
        }

        always {
            sh 'docker ps -a'
        }
    }
}
