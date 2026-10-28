
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'jenkins-cicd-task'
        CONTAINER_NAME = 'jenkins-cicd-app'
        HOST_PORT = '3002'
        APP_PORT = '3000'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Code Quality') {
            steps {
                sh 'node --check app.js'
                sh 'node --check bin/www'
                sh 'node --check routes/index.js'
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
                        if curl --fail --silent http://localhost:${HOST_PORT}/; then
                            echo "Health check passed!"
                            exit 0
                        fi
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
