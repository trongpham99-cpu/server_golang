pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'trongpham99/server_golang'
        DOCKER_TAG = 'latest'
        PROD_SERVER = 'ec2-54-255-237-49.ap-southeast-1.compute.amazonaws.com'
        PROD_USER = 'ubuntu'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'master', url: 'https://github.com/trongpham99-cpu/server_golang.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }

        stage('Run Tests') {
            steps {
                echo 'Running tests...'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials') {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                    }
                }
            }
        }

        stage('Deploy Golang to DEV') {
            steps {
                script {
                    echo 'Clearing server_golang-related images and containers...'
                    sh '''
                        docker container stop server-golang || echo "No container named server-golang to stop"
                        docker container rm server-golang || echo "No container named server-golang to remove"
                        docker image rm ${DOCKER_IMAGE}:${DOCKER_TAG} || echo "No image ${DOCKER_IMAGE}:${DOCKER_TAG} to remove"
                    '''
                    
                    echo 'Deploying to DEV environment...'
                    sh '''
                        docker image pull ${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker network create dev || echo "Network already exists"
                        docker container run -d --rm --name server-golang -p 4000:4000 --network dev ${DOCKER_IMAGE}:${DOCKER_TAG}
                    '''
                }
            }
        }

        stage('Deploy to Production on AWS') {
            steps {
                script {
                    echo 'Deploying to Production...'
                    sshagent(['aws-ssh-key']) { // Use the Jenkins credential ID
                        sh '''
                            ssh -o StrictHostKeyChecking=no ${PROD_USER}@${PROD_SERVER} << EOF
                                docker container stop server-golang || echo "No container to stop"
                                docker container rm server-golang || echo "No container to remove"
                                docker image rm ${DOCKER_IMAGE}:${DOCKER_TAG} || echo "No image to remove"
                                docker image pull ${DOCKER_IMAGE}:${DOCKER_TAG}
                                docker container run -d --rm --name server-golang -p 4000:4000 ${DOCKER_IMAGE}:${DOCKER_TAG}
                            EOF
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
