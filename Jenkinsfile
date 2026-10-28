
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'jenkins-cicd-task'
        CONTAINER_NAME = 'jenkins-cicd-app'
        HOST_PORT = '3001'
        APP_PORT = '3000'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:latest .'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    docker run --rm ${IMAGE_NAME}:latest npm --version
                    echo "Dependencies were installed during the Docker build."
                '''
            }
        }

        stage('Code Quality') {
            steps {
                sh '''
                    docker run --rm ${IMAGE_NAME}:latest node --check app.js
                    docker run --rm ${IMAGE_NAME}:latest node --check bin/www
                    docker run --rm ${IMAGE_NAME}:latest node --check routes/index.js
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    docker run --rm ${IMAGE_NAME}:latest node --version
                    docker image inspect ${IMAGE_NAME}:latest
                '''
            }
        }

        
        stage('Deploy') {
            steps {
                sh '''
                    set -e

                    if docker container inspect ${CONTAINER_NAME} >/dev/null 2>&1; then
                        docker stop ${CONTAINER_NAME}
                        docker rm ${CONTAINER_NAME}
                    fi

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
