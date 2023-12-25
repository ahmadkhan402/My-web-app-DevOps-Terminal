pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                script {
                    dockerImage = docker.build("ahmadkhan402/terminal-web-app:${env.BUILD_ID}")
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
                                        docker push ahmadkhan402/terminal-web-app:${env.BUILD_ID}
                                        docker stop ahmadkhan402/terminal-web-app || true
                                        docker rm ahmadkhan402/terminal-web-app || true
                                        docker run -d --name ahmadkhan402/terminal-web-app -p 80:80 ahmadkhan402/terminal-web-app:${env.BUILD_ID}
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
