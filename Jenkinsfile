pipeline {
    environment {
        IMAGE_NAME = "staticwebsite"
        APP_EXPOSED_PORT = "80"
        IMAGE_TAG = "latest"
        STAGING = "chocoapp-staging"
        PRODUCTION = "chocoapp-prod"
        DOCKERHUB_ID = "nono642"
        DOCKERHUB_PASSWORD = credentials('dockerhub_password')
        APP_NAME = "noah"
        STG_API_ENDPOINT = "127.0.0.1:1993/"
        STG_APP_ENDPOINT = "127.0.0.1:8080/"
        PROD_API_ENDPOINT = "127.0.0.1:1993/"
        PROD_APP_ENDPOINT = "127.0.0.1:80/"
        INTERNAL_PORT = "80"
        EXTERNAL_PORT = "${APP_EXPOSED_PORT}"
        CONTAINER_IMAGE = "${DOCKERHUB_ID}/${IMAGE_NAME}:${IMAGE_TAG}"
    }
    agent any
    stages {
        stage('Build image') {
            steps {
                script {
                   bat 'docker build -t ${DOCKERHUB_ID}/${IMAGE_NAME}:${IMAGE_TAG} .'
                }
            }
        }
        stage('Run container based on builded image') {
            steps {
                script {
                    bat """
                        echo "Cleaning existing container if exist"
                        docker ps -a | grep -i ${IMAGE_NAME} && docker rm -f ${IMAGE_NAME}
                        docker run --name ${IMAGE_NAME} -d -p ${APP_EXPOSED_PORT}:${INTERNAL_PORT} ${DOCKERHUB_ID}/${IMAGE_NAME}:${IMAGE_TAG}
                        sleep 5
                    """
                }
            }
        }
        stage('Test image') {
            steps {
                script {
                    bat "curl -v 172.17.0.1:${APP_EXPOSED_PORT} | grep -i 'Dimension'"
                }
            }
        }
        stage('Clean container') {
            steps {
                script {
                    bat """
                        docker stop ${IMAGE_NAME}
                        docker rm ${IMAGE_NAME}
                    """
                }
            }
        }
        stage('Login and Push Image on docker hub') {
            steps {
                script {
                    bat """
                        echo ${DOCKERHUB_PASSWORD} | docker login -u ${DOCKERHUB_ID} --password-stdin
                        docker push ${DOCKERHUB_ID}/${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }
               stage('STAGING - Deploy app') {
            agent any
            steps {
                script {
                    bat """
                        echo  {\\"your_name\\":\\"${APP_NAME}\\",\\"container_image\\":\\"${CONTAINER_IMAGE}\\", \\"external_port\\":\\"${EXTERNAL_PORT}80\\", \\"internal_port\\":\\"${INTERNAL_PORT}\\"}  > data.json 
                        curl -v -X POST http://${STG_API_ENDPOINT}/staging -H 'Content-Type: application/json'  --data-binary @data.json  2>&1 | grep 200
                    """
                }
            }
        }

        stage('PROD - Deploy app') {
            when {
                expression { GIT_BRANCH == 'origin/main' }
            }
            agent any

            steps {
                script {
                    bat """
                        echo  {\\"your_name\\":\\"${APP_NAME}\\",\\"container_image\\":\\"${CONTAINER_IMAGE}\\", \\"external_port\\":\\"${EXTERNAL_PORT}\\", \\"internal_port\\":\\"${INTERNAL_PORT}\\"}  > data.json 
                        curl -v -X POST http://${PROD_API_ENDPOINT}/prod -H 'Content-Type: application/json'  --data-binary @data.json  2>&1 | grep 200
                    """
                }
            }
        }
    }

    post {
        success {
            script {
                slackSend (color: '#00FF00', message: "ULRICH - SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}) - PROD URL => http://${PROD_APP_ENDPOINT} , STAGING URL => http://${STG_APP_ENDPOINT}")
            }
        }
        failure {
            script {
                slackSend (color: '#FF0000', message: "ULRICH - FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
            }
        }
    }
}
