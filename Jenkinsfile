
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

        stage('Build') {
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
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the console output.'
        }
    }
}
