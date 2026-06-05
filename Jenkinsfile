pipeline {
    agent any

    tools {
        nodejs 'NodeJS'
    }

    environment {
        APP_INTERNAL_PORT = '3000'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo "Branch: ${env.BRANCH_NAME}"
            }
        }

        stage('Set environment') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        env.ENV_NAME = 'main'
                        env.IMAGE_NAME = 'nodemain'
                        env.IMAGE_TAG = 'v1.0'
                        env.CONTAINER_NAME = 'node-main'
                        env.HOST_PORT = '3000'
                    } else if (env.BRANCH_NAME == 'dev') {
                        env.ENV_NAME = 'dev'
                        env.IMAGE_NAME = 'nodedev'
                        env.IMAGE_TAG = 'v1.0'
                        env.CONTAINER_NAME = 'node-dev'
                        env.HOST_PORT = '3001'
                    } else {
                        error "Unsupported branch: ${env.BRANCH_NAME}"
                    }

                    env.FULL_IMAGE_NAME = "${env.IMAGE_NAME}:${env.IMAGE_TAG}"

                    echo "Environment: ${env.ENV_NAME}"
                    echo "Image: ${env.FULL_IMAGE_NAME}"
                    echo "Container: ${env.CONTAINER_NAME}"
                    echo "Host port: ${env.HOST_PORT}"
                }
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build docker image') {
            steps {
                sh 'docker build -t ${FULL_IMAGE_NAME} .'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    set +e

                    if docker ps -a --format '{{.Names}}' | grep -w "${CONTAINER_NAME}"; then
                        echo "Stopping existing container: ${CONTAINER_NAME}"
                        docker stop "${CONTAINER_NAME}"
                        docker rm "${CONTAINER_NAME}"
                    else
                        echo "No existing container for ${CONTAINER_NAME}"
                    fi

                    set -e

                    docker run -d \
                      --name "${CONTAINER_NAME}" \
                      --expose "${APP_INTERNAL_PORT}" \
                      -p "${HOST_PORT}:${APP_INTERNAL_PORT}" \
                      "${FULL_IMAGE_NAME}"

                    docker ps
                '''
            }
        }
    }
}