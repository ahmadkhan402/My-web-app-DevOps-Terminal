@Library('docker') _

pipeline {
    agent any

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image
                    docker.build("ahmadkhan402/terminal-web-app:${BUILD_NUMBER}")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    // Push Docker image to Docker Hub
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                        docker.image("ahmadkhan402/terminal-web-app:${BUILD_NUMBER}").push()
                    }
                }
            }
        }
    }
}

