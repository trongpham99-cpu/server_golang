pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'trongpham99/server_golang'
        DOCKER_TAG = 'latest'
        PROD_SERVER = 'ec2-54-255-237-49.ap-southeast-1.compute.amazonaws.com'
        SSH_KEY = 'key-ductrong-pham.pem'
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
                    echo 'Clearing all images and containers...'
                    sh '''
                        docker container stop $(docker container ls -q) || echo "No containers to stop"
                        docker container prune -f
                        docker image prune -a -f
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
                    sh '''
                        ssh -i "${SSH_KEY}" ${PROD_USER}@${PROD_SERVER} << EOF
                            docker container stop $(docker container ls -q) || echo "No containers to stop"
                            docker container prune -f
                            docker image prune -a -f
                            docker image pull ${DOCKER_IMAGE}:${DOCKER_TAG}
                            docker container run -d --rm --name server-golang -p 4000:4000 ${DOCKER_IMAGE}:${DOCKER_TAG}
                        EOF
                    '''
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
