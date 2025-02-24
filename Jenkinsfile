pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = 'gouthamrohan/dockerimages'
        EC2_IP = '13.232.72.77'
        DOCKERFILE_PATH = '/home/ec2-user/node-app'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                   docker.build("${DOCKER_HUB_REPO}:${env.BUILD_ID}")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-creds') {
                        docker.image("${DOCKER_HUB_REPO}:${env.BUILD_ID}").push()
                    }
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                script {
                    sshagent(['ec2-ssh-key']) {
                        sh """
                            ssh -o StrictHostKeyChecking=no ec2-user@${EC2_IP} '
                                docker pull ${DOCKER_HUB_REPO}:${env.BUILD_ID}
                                docker stop node-app || true
                                docker rm node-app || true
                                docker run -d --name node-app -p 80:3000 ${DOCKER_HUB_REPO}:${env.BUILD_ID}
                            '
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed. Check logs.'
        }
    }
}

