pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = "ahmadkhan402/terminal-web-app"
        DOCKER_IMAGE_TAG = "${env.BUILD_ID}"
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub-credentials-id')
    }

    stages {
        stage('Build') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}")
                }
            }
        }

        stage('Push') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', DOCKER_HUB_CREDENTIALS) {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Test') {
            steps {
                sh 'ls -l index.html'
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Tag current image as backup (previous)
                    sh "docker tag ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} ${DOCKER_IMAGE_NAME}:previous || true"
                    // Push the new image and deploy
                    sh "docker push ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                    sh "docker stop ${DOCKER_IMAGE_NAME} || true"
                    sh "docker rm ${DOCKER_IMAGE_NAME} || true"
                    sh "docker run -d --name ${DOCKER_IMAGE_NAME} -p 80:80 ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
                }
            }
        }
    }

    post {
        failure {
            emailext body: "Something is wrong with the build ${env.BUILD_URL}",
                 subject: "Failed Pipeline: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                 to: 'ahmadsafiullah777@gmail.com'
        }
    }
}
