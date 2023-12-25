pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                script {
                    // Declare dockerImage outside the script block
                    def dockerImage = docker.build("ahmadkhan402/terminal-web-app:${env.BUILD_ID}")
                }
            }
        }

        stage('Push') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
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
                    sshPublisher(
                        publishers: [
                            sshPublisherDesc(
                                configName: "MyUbuntuServer", 
                                transfers: [sshTransfer(
                                    execCommand: """
                                        docker pull ahmadkhan402/terminal-web-app:${env.BUILD_ID}
                                        docker stop terminal-web-app || true
                                        docker rm terminal-web-app || true
                                        docker run -d --name terminal-web-app -p 80:80 ahmadkhan402/terminal-web-app:${env.BUILD_ID}
                                    """
                                )]
                            )
                        ]
                    )
                } 
            }
        }
    }

    post {
        failure {
            mail(
                to: 'ahmadsafiullah777@gmail.com',
                subject: "Failed Pipeline: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                body: "Something is wrong with the build ${env.BUILD_URL}"
            )
        }
    }
}

pipeline {
    agent any

    environment {
        DOCKER_REGISTRY_CREDS = 'DockerAccess'
        EMAIL_NOTIFICATION = 'ahmadsafiullah777@gmail.com'
        DOCKER_BFLASK_IMAGE = 'ahmadkhan402/terminal-web-app' // 
    }

    stages {
        stage('Build') {
            steps {
                sh 'docker build -t terminal-web-app .'
                sh "docker tag terminal-web-app ${DOCKER_BFLASK_IMAGE}"
            }
        }

        stage('Test') {
            steps {
                sh 'docker run terminal-web-app python -m pytest app/tests/'
            }
        }

        stage('Deploy') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_REGISTRY_CREDS}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin docker.io"
                        sh "docker push ${DOCKER_BFLASK_IMAGE}"

                        sh "docker pull ${DOCKER_BFLASK_IMAGE}:latest"
                        sh "docker stop terminal-web-app || true"
                        sh "docker rm terminal-web-app || true"

                        try {
                            sh "docker run -d -p 80:5000 --name terminal-web-app ${DOCKER_BFLASK_IMAGE}:latest"
                        } catch (Exception e) {
                            currentBuild.result = 'FAILURE'
                            error("Deployment failed. Rolling back to the previous version.")
                        }
                    }
                }
            }
        }
    }
}
